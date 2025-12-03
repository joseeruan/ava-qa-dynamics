# QA Analyze - Deep Project Analysis Instructions

<critical>The workflow execution engine is governed by: {project-root}/.bmad/core/tasks/workflow.xml</critical>
<critical>This is a DOCUMENT workflow - outputs markdown analysis report to {default_output_file}</critical>
<critical>Communicate in {communication_language}</critical>

<workflow>

<step n="1" goal="Initialize analysis scope">
  <ask>What is the path to your Dynamics 365 source code? (Default: {default_source_path})</ask>
  <action>Store path as {{source_path}}</action>
  
  <ask>Analysis depth: {pipeline_analysis_depth} - Change? [y/n]</ask>
  <action if="yes">Ask for: quick, standard, or deep</action>
  <action>Store as {{analysis_depth}}</action>
  
  <action>Explain what will be analyzed:
    - Quick: Basic artifact inventory and relationships
    - Standard: Full dependency mapping, execution order, anti-patterns
    - Deep: Everything + complexity analysis, risk scoring, edge case detection
  </action>
</step>

<step n="2" goal="Scan and inventory artifacts">
  <action>Recursively scan {{source_path}} for all Dynamics customizations:</action>
  
  <action>Find Plugins:
    - Search for classes implementing IPlugin interface
    - Extract: Name, namespace, target entity, message, stage, execution mode
    - Parse registered steps if plugin registration code exists
  </action>
  
  <action>Find Custom Workflow Activities:
    - Search for classes inheriting CodeActivity
    - Extract: Name, input/output parameters, business logic summary
  </action>
  
  <action>Find Custom APIs:
    - Search for Custom API definitions (JSON, code, or comments)
    - Extract: Name, bound entity, parameters, permissions
  </action>
  
  <action>Find PCF Controls:
    - Look for PCF project structures
    - Extract: Control name, properties, events
  </action>
  
  <action>Find JavaScript Web Resources:
    - Look for .js files with Dataverse context usage
    - Extract: Form events, ribbon commands, custom logic
  </action>
  
  <template-output>artifact_inventory</template-output>
</step>

<step n="3" goal="Map dependencies and relationships">
  <action>For each artifact, identify dependencies:</action>
  
  <action>Direct dependencies:
    - Entity references (lookup fields)
    - Shared utility classes
    - External service calls
    - Configuration entities
  </action>
  
  <action>Execution dependencies:
    - Plugin A updates field that triggers Plugin B
    - Plugin creates record that starts Power Automate
    - Form script validates data before plugin runs
  </action>
  
  <action>Build dependency graph:
    - Nodes: Artifacts
    - Edges: Dependencies (with type: data, execution, utility)
    - Identify circular dependencies
    - Identify single points of failure
  </action>
  
  <template-output>dependency_map</template-output>
</step>

<step n="4" goal="Analyze pipeline execution order">
  <action>For each entity that has customizations:</action>
  
  <action>Build execution timeline for each message (Create, Update, Delete):
    1. PreValidation sync plugins
    2. PreOperation sync plugins
    3. Core platform operation
    4. PostOperation sync plugins
    5. PostOperation async plugins
    6. Power Automate/workflows
  </action>
  
  <action>Identify execution conflicts:
    - Multiple plugins on same stage/message for same entity
    - Order dependencies not explicitly set
    - Sync plugin that should be async (long-running)
    - Async plugin that should be sync (immediate validation)
  </action>
  
  <action>Identify infinite loop risks:
    - Plugin A updates entity X, triggering Plugin B
    - Plugin B updates entity X, triggering Plugin A again
    - No depth check or depth > reasonable threshold
  </action>
  
  <action>Map image requirements:
    - Which plugins need PreImage
    - Which plugins need PostImage
    - Missing image registrations
  </action>
  
  <template-output>pipeline_analysis</template-output>
</step>

<step n="5" goal="Detect anti-patterns and quality issues">
  <action>Run comprehensive anti-pattern detection based on {antipattern_strictness}:</action>
  
  <action>Scan each plugin for issues:</action>
  
  <check category="Critical">
    - Missing depth validation (infinite loop risk)
    - Unhandled exceptions (no try-catch)
    - Synchronous plugin with HTTP calls (timeout risk)
    - Missing null checks on entity attributes
    - Accessing images without validation
  </check>
  
  <check category="High">
    - Queries without pagination
    - Queries with AllColumns (performance)
    - Hardcoded GUIDs
    - Missing ITracingService (no diagnostics)
    - Complex business logic in PreValidation (should be PreOperation)
  </check>
  
  <check category="Medium">
    - Inconsistent error messages
    - Missing XML documentation
    - Long methods (>100 lines)
    - High cyclomatic complexity (>10)
    - Repeated code blocks
  </check>
  
  <check category="Low" if="strictness=strict">
    - Variable naming conventions
    - Code formatting inconsistencies
    - Missing unit tests
    - No logging strategy
  </check>
  
  <action>For each issue found:
    - Record location (file, line number, method)
    - Describe the problem
    - Explain why it's an issue
    - Provide fix recommendation with code example
    - Assign severity score (1-10)
  </action>
  
  <template-output>quality_issues</template-output>
</step>

<step n="6" goal="Calculate quality metrics" if="analysis_depth=deep">
  <action>Calculate comprehensive quality metrics:</action>
  
  <metric name="Test Coverage Estimate">
    - Count artifacts with corresponding test files
    - Estimate: (artifacts_with_tests / total_artifacts) * 100
  </metric>
  
  <metric name="Anti-Pattern Density">
    - Count total issues per category
    - Calculate: issues_per_1000_lines_of_code
  </metric>
  
  <metric name="Complexity Score">
    - Average cyclomatic complexity across all methods
    - Identify top 10 most complex methods
  </metric>
  
  <metric name="Pipeline Risk Score">
    - Factor in: circular dependencies, missing depth checks, execution conflicts
    - Scale: 0-100 (100 = highest risk)
  </metric>
  
  <metric name="Maintainability Index">
    - Based on: complexity, documentation, code duplication
    - Scale: 0-100 (100 = most maintainable)
  </metric>
  
  <template-output>quality_metrics</template-output>
</step>

<step n="7" goal="Generate recommendations">
  <action>Based on analysis, generate prioritized recommendations:</action>
  
  <priority level="P0 - Critical">
    - Fix infinite loop risks immediately
    - Add missing depth validation
    - Handle unhandled exceptions
    - Fix missing null checks
  </priority>
  
  <priority level="P1 - High">
    - Add pagination to queries
    - Move sync plugins to async where appropriate
    - Remove hardcoded GUIDs
    - Add ITracingService for monitoring
  </priority>
  
  <priority level="P2 - Medium">
    - Improve code organization
    - Add missing documentation
    - Refactor complex methods
    - Add unit tests for critical paths
  </priority>
  
  <priority level="P3 - Low">
    - Standardize naming conventions
    - Improve code formatting
    - Add more comprehensive logging
  </priority>
  
  <action>For each recommendation:
    - Estimated effort (hours)
    - Expected impact (high/medium/low)
    - Dependencies or prerequisites
  </action>
  
  <template-output>recommendations</template-output>
</step>

<step n="8" goal="Generate visualization suggestions" optional="true">
  <ask>Would you like suggestions for visualizing this analysis? [y/n]</ask>
  
  <action if="yes">
    Provide suggestions for:
    - Dependency graph visualization (Graphviz, Mermaid)
    - Pipeline execution diagrams
    - Quality metrics dashboard
    - Trend tracking over time
  </action>
  
  <template-output>visualization_guide</template-output>
</step>

<step n="9" goal="Finalize analysis report">
  <action>Review complete analysis document</action>
  <action>Ensure all sections are comprehensive</action>
  <action>Add table of contents</action>
  <action>Add executive summary</action>
  <action>Save final report to: {default_output_file}</action>
  
  <action>Inform user:
    - Analysis complete! 🎉
    - Report saved to: {default_output_file}
    - Review recommendations and prioritize fixes
    - Use [CR] Code Review for focused anti-pattern analysis
    - Use [GT] Generate Tests to improve test coverage
  </action>
</step>

</workflow>
