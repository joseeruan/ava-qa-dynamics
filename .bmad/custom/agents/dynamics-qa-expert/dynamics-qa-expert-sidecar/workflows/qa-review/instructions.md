# QA Code Review - Automated Quality Analysis Instructions

<critical>The workflow execution engine is governed by: {project-root}/.bmad/core/tasks/workflow.xml</critical>
<critical>This is a DOCUMENT workflow - outputs markdown review report to {default_output_file}</critical>
<critical>Communicate in {communication_language}</critical>

<workflow>

<step n="1" goal="Define review scope">
  <ask>What would you like to review?
    1. Full project (all code)
    2. Recent changes (git diff)
    3. Specific files or components
    4. Pull request / branch comparison
  </ask>
  <action>Store as {{review_scope}}</action>
  
  <action if="scope=full project">
    <ask>Source path? (Default: {default_source_path})</ask>
    <action>Store as {{review_path}}</action>
  </action>
  
  <action if="scope=recent changes">
    <ask>Compare against which branch? (Default: main)</ask>
    <action>Store as {{base_branch}}</action>
    <action>Use git diff to identify changed files</action>
    <action>Store changed files as {{review_files}}</action>
  </action>
  
  <action if="scope=specific files">
    <ask>Enter file paths (comma-separated) or class names:</ask>
    <action>Store as {{review_files}}</action>
  </action>
  
  <action if="scope=pull request">
    <ask>PR number or branch name:</ask>
    <action>Fetch PR diff and extract changed files</action>
    <action>Store as {{review_files}} and {{pr_context}}</action>
  </action>
  
  <ask>Review strictness: {antipattern_strictness} - Change? [y/n]</ask>
  <action if="yes">Ask for: relaxed, balanced, or strict</action>
  <action>Store as {{review_strictness}}</action>
</step>

<step n="2" goal="Load anti-pattern database">
  <action>Load anti-pattern definitions from: {data_path}/dynamics-antipatterns.json</action>
  <action>Load best practices from: {data_path}/best-practices.json</action>
  
  <action>Filter patterns based on {{review_strictness}}:
    - Relaxed: Only critical severity
    - Balanced: Critical + high severity
    - Strict: All severity levels
  </action>
  
  <action>Prepare detection rules for:
    - Missing depth validation
    - Unhandled exceptions
    - Unpaginated queries
    - Hardcoded GUIDs
    - Missing null checks
    - Synchronous HTTP calls
    - Missing tracing
    - Image access without validation
    - And 20+ more patterns...
  </action>
</step>

<step n="3" goal="Analyze code for anti-patterns">
  <action>For each file in {{review_files}}:</action>
  
  <action>Parse code structure:
    - Identify classes and methods
    - Extract logic flow
    - Map dependencies
    - Calculate cyclomatic complexity
  </action>
  
  <action>Run anti-pattern detection:</action>
  
  <check category="Critical" severity="10">
    <pattern id="missing-depth-check">
      <detection>Plugin Execute method without context.Depth validation</detection>
      <location>{{file}}:{{line}}</location>
      <description>Missing depth check creates infinite loop risk</description>
      <fix>Add at start of Execute: if (context.Depth > 1) return;</fix>
      <example>
        ```csharp
        // Before (DANGEROUS)
        public void Execute(IServiceProvider serviceProvider) {
            var context = (IPluginExecutionContext)serviceProvider.GetService(...);
            // No depth check!
        }
        
        // After (SAFE)
        public void Execute(IServiceProvider serviceProvider) {
            var context = (IPluginExecutionContext)serviceProvider.GetService(...);
            if (context.Depth > 1) return; // Prevent infinite loops
        }
        ```
      </example>
    </pattern>
    
    <pattern id="unhandled-exception">
      <detection>Plugin/workflow Execute without try-catch</detection>
      <location>{{file}}:{{line}}</location>
      <description>Unhandled exceptions crash platform operations</description>
      <fix>Wrap Execute body in try-catch, throw InvalidPluginExecutionException</fix>
    </pattern>
    
    <pattern id="sync-http-call">
      <detection>HttpClient or WebRequest in sync plugin</detection>
      <location>{{file}}:{{line}}</location>
      <description>Synchronous plugins timeout with external calls</description>
      <fix>Move to async plugin or use async/await properly</fix>
    </pattern>
  </check>
  
  <check category="High" severity="7-9">
    <pattern id="unpaginated-query">
      <detection>QueryExpression without TopCount or PagingInfo</detection>
      <location>{{file}}:{{line}}</location>
      <description>Retrieving all records causes performance issues</description>
      <fix>Add query.TopCount = 5000 or implement pagination</fix>
    </pattern>
    
    <pattern id="all-columns">
      <detection>ColumnSet(true) or AllColumns usage</detection>
      <location>{{file}}:{{line}}</location>
      <description>Retrieving all columns impacts performance</description>
      <fix>Specify only needed columns: new ColumnSet("field1", "field2")</fix>
    </pattern>
    
    <pattern id="hardcoded-guid">
      <detection>new Guid("...") in business logic</detection>
      <location>{{file}}:{{line}}</location>
      <description>Hardcoded GUIDs break across environments</description>
      <fix>Lookup by name, use config, or environment variables</fix>
    </pattern>
    
    <pattern id="missing-tracing">
      <detection>Plugin without ITracingService usage</detection>
      <location>{{file}}:{{line}}</location>
      <description>No diagnostics available when debugging</description>
      <fix>Get ITracingService and add trace.Trace() calls</fix>
    </pattern>
    
    <pattern id="missing-null-check">
      <detection>entity["field"] without Contains check</detection>
      <location>{{file}}:{{line}}</location>
      <description>NullReferenceException if field not present</description>
      <fix>Check: if (entity.Contains("field") && entity["field"] != null)</fix>
    </pattern>
    
    <pattern id="missing-image-check">
      <detection>PreEntityImages[name] without Contains validation</detection>
      <location>{{file}}:{{line}}</location>
      <description>KeyNotFoundException if image not registered</description>
      <fix>Check: if (context.PreEntityImages.Contains("imageName"))</fix>
    </pattern>
  </check>
  
  <check category="Medium" severity="4-6">
    <pattern id="complex-method">
      <detection>Method with cyclomatic complexity > 10</detection>
      <location>{{file}}:{{line}}</location>
      <description>High complexity makes code hard to test and maintain</description>
      <fix>Refactor into smaller methods with single responsibility</fix>
    </pattern>
    
    <pattern id="long-method">
      <detection>Method with > 100 lines</detection>
      <location>{{file}}:{{line}}</location>
      <description>Long methods are hard to understand and test</description>
      <fix>Extract logical blocks into separate methods</fix>
    </pattern>
    
    <pattern id="duplicate-code">
      <detection>Code blocks repeated > 2 times</detection>
      <location>Multiple locations</location>
      <description>Duplication makes maintenance difficult</description>
      <fix>Extract to shared method or utility class</fix>
    </pattern>
    
    <pattern id="missing-documentation">
      <detection>Public methods without XML documentation</detection>
      <location>{{file}}:{{line}}</location>
      <description>Lack of documentation hinders maintainability</description>
      <fix>Add /// XML comments explaining purpose and parameters</fix>
    </pattern>
  </check>
  
  <check category="Low" severity="1-3" if="strictness=strict">
    <pattern id="naming-convention">
      <detection>Variables not following camelCase/PascalCase</detection>
      <location>{{file}}:{{line}}</location>
      <description>Inconsistent naming reduces readability</description>
      <fix>Follow C# naming conventions</fix>
    </pattern>
    
    <pattern id="unused-variable">
      <detection>Declared variables never used</detection>
      <location>{{file}}:{{line}}</location>
      <description>Dead code clutters codebase</description>
      <fix>Remove unused declarations</fix>
    </pattern>
    
    <pattern id="commented-code">
      <detection>Large blocks of commented-out code</detection>
      <location>{{file}}:{{line}}</location>
      <description>Commented code should be removed (use source control)</description>
      <fix>Delete commented blocks, trust git history</fix>
    </pattern>
  </check>
  
  <action>Record each detected issue with:
    - Pattern ID
    - File and line number
    - Code snippet (3 lines context)
    - Severity score (1-10)
    - Description of problem
    - Fix recommendation
    - Code example (before/after)
  </action>
  
  <template-output>antipattern_detection_results</template-output>
</step>

<step n="4" goal="Calculate quality score">
  <action>Calculate overall quality score (0-100):</action>
  
  <calculation>
    Base score: 100
    
    For each issue detected:
      - Critical (severity 10): -10 points
      - High (severity 7-9): -5 points
      - Medium (severity 4-6): -2 points
      - Low (severity 1-3): -0.5 points
    
    Minimum score: 0
    Maximum score: 100
  </calculation>
  
  <action>Calculate category breakdown:</action>
  <metric>Code Safety Score: Based on critical issues (depth checks, exception handling, null checks)</metric>
  <metric>Performance Score: Based on query optimization, pagination, column sets</metric>
  <metric>Maintainability Score: Based on complexity, documentation, code organization</metric>
  <metric>Best Practices Score: Based on tracing, naming, patterns adherence</metric>
  
  <action>Determine quality grade:
    - A (90-100): Excellent
    - B (80-89): Good
    - C (70-79): Acceptable
    - D (60-69): Needs Improvement
    - F (0-59): Critical Issues
  </action>
  
  <template-output>quality_score</template-output>
</step>

<step n="5" goal="Analyze test coverage" optional="true">
  <ask>Check test coverage for reviewed components? [y/n]</ask>
  
  <action if="yes">
    <action>For each reviewed class:
      - Look for corresponding test file ({{ClassName}}Tests.cs)
      - If found: Analyze test completeness
      - If not found: Flag as "No tests"
    </action>
    
    <action>Calculate coverage metrics:
      - Classes with tests: {{tested_classes}}/{{total_classes}}
      - Methods with tests: {{tested_methods}}/{{total_methods}}
      - Estimated line coverage: {{estimated_coverage}}%
    </action>
  </action>
  
  <template-output>test_coverage_analysis</template-output>
</step>

<step n="6" goal="Generate prioritized recommendations">
  <action>Sort all detected issues by severity</action>
  <action>Group by category (Safety, Performance, Maintainability, Best Practices)</action>
  
  <action>Generate action plan:</action>
  
  <priority level="URGENT" color="red">
    - All critical issues (severity 10)
    - Must fix before deployment
    - Estimated effort per fix
  </priority>
  
  <priority level="HIGH" color="orange">
    - High severity issues (7-9)
    - Should fix this sprint
    - Estimated effort per fix
  </priority>
  
  <priority level="MEDIUM" color="yellow">
    - Medium severity issues (4-6)
    - Plan for next sprint
    - Can be batched together
  </priority>
  
  <priority level="LOW" color="green">
    - Low severity issues (1-3)
    - Add to backlog
    - Address during refactoring
  </priority>
  
  <action>Calculate total estimated effort (hours) for each priority level</action>
  
  <template-output>prioritized_recommendations</template-output>
</step>

<step n="7" goal="Generate comparison metrics" if="review_scope=recent changes OR pull request">
  <action>Compare current state to previous:</action>
  
  <metric>Issues introduced in this change: {{new_issues_count}}</metric>
  <metric>Issues fixed in this change: {{fixed_issues_count}}</metric>
  <metric>Net quality change: {{quality_delta}}</metric>
  
  <action>Determine if quality improved, degraded, or stayed same</action>
  
  <action if="PR context">
    <action>Generate PR review comment summary:
      - Overall assessment (Approve / Request Changes / Comment)
      - Critical blockers (if any)
      - Suggested improvements
      - Positive feedback on good practices
    </action>
  </action>
  
  <template-output>comparison_metrics</template-output>
</step>

<step n="8" goal="Finalize code review report">
  <action>Compile complete review report</action>
  <action>Add executive summary</action>
  <action>Include all sections: score, issues, recommendations, metrics</action>
  <action>Add visualizations (charts, graphs) if applicable</action>
  <action>Save to: {default_output_file}</action>
  
  <action>Inform user:
    - Code review complete! 🧪
    - Quality Score: {{overall_score}}/100 (Grade {{grade}})
    - Critical Issues: {{critical_count}}
    - Report saved to: {default_output_file}
  </action>
</step>

<step n="9" goal="Provide actionable next steps">
  <action>Based on review results, suggest:</action>
  
  <action if="critical issues found">
    ⚠️ URGENT: Address {{critical_count}} critical issues immediately
    Use report to locate and fix each issue
    Re-run review after fixes
  </action>
  
  <action if="no tests found">
    💡 Generate tests using [GT] Generate Tests workflow
    Recommended for {{untested_classes_count}} untested classes
  </action>
  
  <action if="score < 70">
    📈 Consider refactoring session
    Focus on top {{top_issues_count}} high-impact issues
    Track improvement with monthly reviews
  </action>
  
  <action if="score >= 90">
    ✨ Excellent code quality!
    Maintain standards with automated reviews in CI/CD
    Share best practices with team
  </action>
  
  <ask>Would you like to:
    - [V] View full report
    - [F] Fix issues now (open files in editor)
    - [R] Re-run review (after fixes)
    - [E] Exit
  </ask>
  
  <action if="V">Display complete report</action>
  <action if="F">Open flagged files with issue annotations</action>
  <action if="R">Restart workflow from step 1</action>
  <action if="E">Thank user and exit</action>
</step>

</workflow>
