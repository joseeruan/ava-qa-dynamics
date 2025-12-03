# QA Generate Tests - Granular Test Generation Instructions

<critical>The workflow execution engine is governed by: {project-root}/.bmad/core/tasks/workflow.xml</critical>
<critical>Communicate in {communication_language}</critical>

<workflow>

<step n="1" goal="Define generation scope">
  <ask>What is the path to your Dynamics 365 source code? (Default: {default_source_path})</ask>
  <action>Store path as {{source_path}}</action>
  
  <ask>What type of tests do you want to generate?
    1. Unit tests only
    2. Integration tests only
    3. Both unit and integration tests
  </ask>
  <action>Store as {{test_type}}</action>
  
  <ask>Generation scope:
    1. All components (full project)
    2. Specific component types (plugins, workflows, APIs)
    3. Specific files or classes
  </ask>
  <action>Store as {{generation_scope}}</action>
  
  <action if="scope=specific types">Ask which types: Plugins, Workflows, Custom APIs, PCF</action>
  <action if="scope=specific files">Ask for file paths or class names (comma-separated)</action>
</step>

<step n="2" goal="Verify test project exists">
  <action>Check if test project exists at: {test_output_location}</action>
  
  <check if="test project exists">
    <action>Confirm: "Test project found at {test_output_location}"</action>
    <ask>Use this location? [y/n]</ask>
    <action if="no">Ask for alternative test project path</action>
  </check>
  
  <check if="test project does NOT exist">
    <ask>No test project found. Would you like to:
      1. Create test project now (recommended)
      2. Specify different location
      3. Cancel and run [QS] Quick Setup first
    </ask>
    
    <action if="option 1">
      <action>Create test project structure:
        - Create directory: {test_output_location}
        - Create .csproj with {test_framework} references
        - Create base classes (TestBase, FakeContextFactory)
        - Create helper directories
      </action>
    </action>
    
    <action if="option 2">Ask for test project path and validate</action>
    <action if="option 3">Exit workflow with message to run Quick Setup</action>
  </check>
  
  <template-output>test_project_ready</template-output>
</step>

<step n="3" goal="Scan and analyze target components">
  <action>Scan {{source_path}} for components matching {{generation_scope}}:</action>
  
  <action>For each target component:
    - Extract metadata (name, namespace, entity, message, stage)
    - Parse code structure and logic
    - Identify decision branches
    - Identify dependencies and mocks needed
    - Analyze complexity (cyclomatic complexity)
    - Identify image requirements (PreImage/PostImage)
    - Identify InputParameters usage
  </action>
  
  <action>Build generation plan:
    - List of components to generate tests for
    - For each: estimated test count, complexity level
    - Total estimated tests
    - Estimated generation time
  </action>
  
  <action>Display generation plan to user</action>
  <ask>Proceed with generation? [y/n/adjust]</ask>
  
  <action if="adjust">Allow user to exclude specific components or adjust scope</action>
  <action if="no">Exit workflow</action>
  
  <template-output>generation_plan</template-output>
</step>

<step n="4" goal="Generate unit tests" if="test_type includes unit">
  <action>For each component in generation plan:</action>
  
  <substep n="4a" title="Load appropriate template">
    <action>Load template from {templates_path} based on component type:
      - Plugin → plugin-test-template.cs
      - Workflow Activity → workflow-test-template.cs
      - Custom API → api-test-template.cs
      - PCF Control → pcf-test-template.cs
    </action>
  </substep>
  
  <substep n="4b" title="Generate test class">
    <action>Create test class file: {{ComponentName}}Tests.cs</action>
    <action>Set namespace to match project structure</action>
    <action>Add class-level XML documentation based on {comment_level}</action>
    <action>Inherit from TestBase</action>
    <action>Add test framework attributes ([TestClass] or [TestFixture])</action>
  </substep>
  
  <substep n="4c" title="Generate setup method">
    <action>Create Setup/Initialize method:
      - Initialize FakeXrmEasy context
      - Register plugin with correct configuration
      - Create common test data
      - Setup mock services
    </action>
    <action>Add comments explaining setup based on {comment_level}</action>
  </substep>
  
  <substep n="4d" title="Generate test methods for each execution path">
    <action>For each identified code path:</action>
    
    <action>Create test method:
      - Name: Execute_When{{Condition}}_Should{{ExpectedBehavior}}
      - Arrange: Setup test data, configure mocks, prepare entity
      - Act: Execute plugin/component
      - Assert: Verify expected outcomes
    </action>
    
    <action>Add detailed comments based on {comment_level}:
      - Detailed: Explain every line (why we arrange this way, what we're testing, why assertion matters)
      - Standard: Explain test purpose and key assertions
      - Minimal: Test name only (self-documenting)
    </action>
    
    <action>Example tests to generate:
      - Happy path (normal execution)
      - Edge cases (boundary values, empty data)
      - Error scenarios (null values, missing fields, invalid data)
      - Permission scenarios (different user roles)
      - Image scenarios (with/without PreImage/PostImage)
    </action>
  </substep>
  
  <substep n="4e" title="Generate helper methods if needed">
    <action>If component has complex setup, create helper methods:
      - CreateTestEntity{{EntityName}}()
      - SetupMock{{ServiceName}}()
      - AssertExpected{{Behavior}}()
    </action>
  </substep>
  
  <substep n="4f" title="Save test file">
    <action>Save to: {test_output_location}/UnitTests/{{ComponentName}}Tests.cs</action>
    <action>Format code properly (indentation, spacing)</action>
    <action>Add file header with generation timestamp and DQA attribution</action>
  </substep>
  
  <action>Report progress: "Generated unit tests for {{component_name}} ({{test_count}} tests)"</action>
  
  <template-output>unit_tests_progress</template-output>
</step>

<step n="5" goal="Generate integration tests" if="test_type includes integration">
  <action>Identify integration test scenarios:</action>
  
  <action>Scan for multi-component interactions:
    - Plugin chains (Plugin A → triggers → Plugin B)
    - Create/Update cascades
    - Workflow triggers from plugins
    - Cross-entity dependencies
  </action>
  
  <action>For each integration scenario:</action>
  
  <substep n="5a" title="Generate integration test class">
    <action>Create: {{ScenarioName}}IntegrationTests.cs</action>
    <action>Setup complete FakeXrmEasy environment:
      - Register all involved plugins with correct stages
      - Setup entity metadata
      - Configure relationships
      - Prepare initial data
    </action>
  </substep>
  
  <substep n="5b" title="Generate integration test method">
    <action>Create test method:
      - Name: IntegrationTest_{{ScenarioDescription}}
      - Arrange: Setup full context with all artifacts
      - Act: Trigger initial action (e.g., Create entity)
      - Assert: Verify complete pipeline execution and end state
    </action>
    
    <action>Add comments explaining:
      - What pipeline sequence is being tested
      - Which artifacts execute in which order
      - What the final expected state is
      - Why this integration matters
    </action>
  </substep>
  
  <substep n="5c" title="Save integration test file">
    <action>Save to: {test_output_location}/IntegrationTests/{{ScenarioName}}IntegrationTests.cs</action>
  </substep>
  
  <action>Report progress: "Generated integration tests for {{scenario_name}}"</action>
  
  <template-output>integration_tests_progress</template-output>
</step>

<step n="6" goal="Generate test data helpers" optional="true">
  <ask>Generate test data helper classes? (Recommended for consistency) [y/n]</ask>
  
  <action if="yes">
    <action>Create TestDataFactory.cs:
      - Methods to create common test entities
      - Methods to setup common relationships
      - Methods to create mock users and teams
      - Constants for commonly used test values
    </action>
    
    <action>Add extensive XML documentation</action>
    <action>Save to: {test_output_location}/Helpers/TestDataFactory.cs</action>
  </action>
  
  <template-output>test_data_helpers</template-output>
</step>

<step n="7" goal="Validate and compile">
  <action>Validate generated test files:
    - Check syntax (no compilation errors)
    - Verify all references resolved
    - Ensure proper namespaces
    - Check test framework attributes correct
  </action>
  
  <ask>Attempt to build test project now? [y/n]</ask>
  
  <action if="yes">
    <action>Run: dotnet build {test_output_location}</action>
    <action>Display build output</action>
    
    <check if="build successful">
      <action>✅ Build successful! Tests ready to run.</action>
    </check>
    
    <check if="build failed">
      <action>⚠️ Build failed. Review errors and fix before running tests.</action>
      <action>Display compilation errors with file/line numbers</action>
    </check>
  </action>
  
  <template-output>build_validation</template-output>
</step>

<step n="8" goal="Generate summary report">
  <action>Compile generation summary:</action>
  
  <section name="Generation Summary">
    - Source path: {{source_path}}
    - Test project: {test_output_location}
    - Test framework: {test_framework}
    - Generation scope: {{generation_scope}}
    - Test types: {{test_type}}
  </section>
  
  <section name="Tests Generated">
    - Unit test classes: {{unit_test_classes_count}}
    - Unit test methods: {{unit_test_methods_count}}
    - Integration test classes: {{integration_test_classes_count}}
    - Integration test methods: {{integration_test_methods_count}}
    - Total tests: {{total_test_count}}
  </section>
  
  <section name="Files Created">
    {{list_of_generated_files}}
  </section>
  
  <section name="Next Steps">
    1. Review generated tests in: {test_output_location}
    2. Customize tests as needed for your specific scenarios
    3. Run tests: `dotnet test {test_output_location}`
    4. Add tests to source control
    5. Integrate into CI/CD pipeline
  </section>
  
  <section name="Running Tests">
    ```bash
    cd {test_output_location}
    dotnet test
    
    # Run specific test class
    dotnet test --filter "FullyQualifiedName~{{TestClassName}}"
    
    # Run with detailed output
    dotnet test --logger "console;verbosity=detailed"
    ```
  </section>
  
  <template-output>generation_summary</template-output>
</step>

<step n="9" goal="Completion">
  <action>✅ Test generation complete! 🎉</action>
  
  <action>Provide quick actions:
    - [V] View generation summary
    - [R] Run tests now
    - [G] Generate more tests (different scope)
    - [E] Exit
  </action>
  
  <action if="V">Display full summary report</action>
  <action if="R">Execute: dotnet test {test_output_location}</action>
  <action if="G">Restart workflow from step 1</action>
  <action if="E">Thank user and exit</action>
</step>

</workflow>
