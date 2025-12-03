# QA Quick Setup - All-in-One Test Suite Generation

<critical>The workflow execution engine is governed by: {project-root}/.bmad/core/tasks/workflow.xml</critical>
<critical>Communicate in {communication_language} throughout the process</critical>

<workflow>

<step n="1" goal="Welcome and context gathering">
  <action>Welcome user to QA Quick Setup</action>
  <action>Explain this workflow will:
    1. Analyze Dynamics project structure
    2. Create test project with proper architecture
    3. Generate unit tests for all components
    4. Generate integration tests for key flows
    5. Provide quality report and next steps
  </action>
  
  <ask>What is the path to your Dynamics 365 source code? (Default: {default_source_path})</ask>
  <action>Store path as {{source_path}}</action>
  <action>If user accepts default, use {default_source_path}</action>
  
  <ask>Confirm test output location: {test_output_location} - Is this correct? [y/n]</ask>
  <action if="no">Ask for alternative path and store as {{test_output_location}}</action>
</step>

<step n="2" goal="Analyze project structure">
  <action>Scan {{source_path}} for Dynamics artifacts:</action>
  <action>• Look for .cs files containing IPlugin implementations</action>
  <action>• Look for classes inheriting CodeActivity (custom workflows)</action>
  <action>• Look for Custom API definitions</action>
  <action>• Look for PCF control projects</action>
  <action>• Identify namespaces and project structure</action>
  
  <action>Create artifact inventory:
    - Count plugins found
    - Count workflow activities found
    - Count custom APIs found
    - Count PCF controls found
    - Map dependencies between artifacts
  </action>
  
  <action>Detect existing test framework if {auto_detect_framework} = "yes":
    - Check for existing .csproj files in test directories
    - Look for XUnit, NUnit, or MSTest package references
    - If found, suggest using existing framework
  </action>
  
  <ask if="framework detected">Detected existing test framework: {{detected_framework}}. Use this instead of configured {test_framework}? [y/n]</ask>
  <action if="yes">Set {{active_framework}} = {{detected_framework}}</action>
  <action if="no">Set {{active_framework}} = {test_framework}</action>
  <action if="no framework detected">Set {{active_framework}} = {test_framework}</action>
  
  <template-output>analysis_summary</template-output>
</step>

<step n="3" goal="Run anti-pattern detection">
  <action>Analyze code for common Dynamics anti-patterns based on {antipattern_strictness}:</action>
  
  <action>For each plugin found:
    - Check for depth validation (context.Depth > 1)
    - Check for ITracingService usage
    - Check for hardcoded GUIDs
    - Check for proper exception handling
    - Check for image validation before access
    - Check for null checks on entity attributes
  </action>
  
  <action>For each query found:
    - Check for pagination (TopCount or PagingInfo)
    - Check for ColumnSet specification (not AllColumns)
  </action>
  
  <action>Generate anti-pattern report with:
    - Critical issues (must fix)
    - Warnings (should fix)
    - Suggestions (nice to have)
    - For each issue: location, description, fix recommendation
  </action>
  
  <template-output>antipattern_report</template-output>
</step>

<step n="4" goal="Create test project structure">
  <action>Create test project directory: {{test_output_location}}</action>
  <action>Create subdirectories:
    - /UnitTests
    - /IntegrationTests
    - /Helpers
    - /Mocks
    - /TestData
  </action>
  
  <action>Create test project file (.csproj) with:
    - Target framework: .NET 6.0 or higher
    - Package references for {{active_framework}}
    - Package reference for FakeXrmEasy (latest version)
    - Package reference for FluentAssertions
    - Project reference to source project
  </action>
  
  <action>Load base class templates from {templates_path}:
    - TestBase.cs - Base class for all tests
    - FakeContextFactory.cs - Factory for FakeXrmEasy context
    - MockServiceProvider.cs - Mock service provider
    - TestEntityFactory.cs - Helper to create test entities
  </action>
  
  <action>Generate base helper classes in /Helpers:
    - Use templates loaded above
    - Customize namespaces to match project
    - Add XML documentation comments based on {comment_level}
  </action>
  
  <template-output>test_project_structure</template-output>
</step>

<step n="5" goal="Generate unit tests for plugins">
  <action>For each plugin discovered in step 2:</action>
  
  <action>Analyze plugin code:
    - Identify registered stage (PreValidation, PreOperation, PostOperation)
    - Identify registered message (Create, Update, Delete, etc.)
    - Identify target entity
    - Parse Execute method logic
    - Identify decision branches and paths
    - Identify image requirements (PreImage/PostImage)
  </action>
  
  <action>Generate test class:
    - Class name: {{PluginName}}Tests
    - Inherit from TestBase
    - Add [TestClass] or [TestFixture] based on {{active_framework}}
  </action>
  
  <action>Generate test methods for each execution path:
    - Setup: Create FakeContext, mock services, prepare target entity
    - Act: Execute plugin
    - Assert: Verify expected behavior
    - Add descriptive test names (e.g., "Execute_WhenAccountCreated_ShouldSetDefaultValues")
  </action>
  
  <action>Add test for error scenarios:
    - Test with missing required fields
    - Test with null values
    - Test with invalid data
  </action>
  
  <action>Add comments based on {comment_level}:
    - Detailed: Explain each line (Arrange, Act, Assert, why we test this)
    - Standard: Explain test purpose and key assertions
    - Minimal: Only test name and purpose
  </action>
  
  <template-output>unit_tests_generated</template-output>
</step>

<step n="6" goal="Generate integration tests for key flows">
  <action>Identify integration test scenarios:
    - Look for plugins that trigger other plugins (depth > 1 scenarios)
    - Look for Create→Plugin→Update chains
    - Look for workflows triggered by plugin updates
  </action>
  
  <action>For each integration scenario:</action>
  
  <action>Generate integration test:
    - Setup complete FakeXrmEasy context
    - Register all involved plugins with correct stages
    - Create initial trigger entity
    - Execute full pipeline simulation
    - Assert end-state of all affected entities
  </action>
  
  <action>Add comments explaining:
    - What pipeline sequence is being tested
    - Why this integration matters
    - What we're verifying
  </action>
  
  <template-output>integration_tests_generated</template-output>
</step>

<step n="7" goal="Generate comprehensive quality report">
  <action>Compile final report with:</action>
  
  <section name="Executive Summary">
    - Total artifacts analyzed: {{plugin_count + workflow_count + api_count}}
    - Test project created: {{test_output_location}}
    - Test framework used: {{active_framework}}
    - Unit tests generated: {{unit_test_count}}
    - Integration tests generated: {{integration_test_count}}
    - Anti-patterns detected: {{antipattern_count}}
  </section>
  
  <section name="Project Analysis">
    - Plugins: {{plugin_count}}
    - Workflow Activities: {{workflow_count}}
    - Custom APIs: {{api_count}}
    - PCF Controls: {{pcf_count}}
    - Dependencies mapped: {{dependency_count}}
  </section>
  
  <section name="Test Coverage">
    - Components with unit tests: {{covered_components}}/{{total_components}}
    - Integration flows tested: {{integration_flow_count}}
    - Estimated code coverage: {{estimated_coverage}}%
  </section>
  
  <section name="Quality Issues">
    - Critical anti-patterns: {{critical_count}}
    - Warnings: {{warning_count}}
    - Suggestions: {{suggestion_count}}
    - Detailed list with locations and fixes
  </section>
  
  <section name="Next Steps">
    1. Review and fix critical anti-patterns
    2. Run generated tests: `dotnet test {{test_output_location}}`
    3. Review test results and adjust as needed
    4. Integrate tests into CI/CD pipeline
    5. Use [CR] Code Review workflow for ongoing quality checks
    6. Use [GT] Generate Tests for new components as you develop
  </section>
  
  <section name="How to Run Tests">
    ```bash
    cd {{test_output_location}}
    dotnet restore
    dotnet build
    dotnet test
    ```
  </section>
  
  <template-output>final_quality_report</template-output>
</step>

<step n="8" goal="Completion and guidance">
  <action>Congratulate user on completing QA Quick Setup! 🎉</action>
  
  <action>Provide actionable next steps:
    1. Navigate to test project: {{test_output_location}}
    2. Open in Visual Studio or VS Code
    3. Review generated tests
    4. Fix critical anti-patterns identified
    5. Run tests to verify everything works
    6. Adjust and customize tests as needed
  </action>
  
  <ask>Would you like to:
    - [V] View the quality report
    - [R] Re-run with different settings
    - [E] Exit and review on your own
  </ask>
  
  <action if="V">Display full quality report</action>
  <action if="R">Restart workflow from step 1</action>
  <action if="E">Thank user and exit</action>
</step>

</workflow>
