# QA Analyze Workflow - Instructions

## Purpose
This workflow performs comprehensive analysis of a Dynamics 365 project, identifying all custom components (plugins, workflows, custom APIs, PCF controls), detecting anti-patterns, and analyzing potential pipeline execution conflicts.

## When to Use
- Initial project assessment
- Before starting test generation
- Code review preparation
- Pipeline conflict investigation
- Architecture documentation

## Execution Flow

### Step 1: Validate Project Path
Ensures the provided project path exists and is accessible.

**What it does:**
- Checks if path exists
- Validates it's a directory
- Ensures read permissions

**Failure scenarios:**
- Path doesn't exist → User prompted to provide correct path
- No read permissions → Error with instructions to fix

---

### Step 2: Scan Project Structure
Recursively scans the project directory for relevant files.

**File patterns searched:**
- `**/*.cs` - C# source files (plugins, workflows, entities)
- `**/*.csproj` - Project files
- `**/*.ts`, `**/*.tsx` - TypeScript files (PCF controls)

**Excluded directories:**
- `node_modules/` - NPM dependencies
- `bin/`, `obj/` - Build artifacts

**Output:** List of all relevant files found

---

### Step 3: Identify Components
Analyzes scanned files to identify Dynamics 365 components.

**Detection patterns:**

**Plugins:**
```csharp
public class MyPlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        // Plugin code
    }
}
```

**Custom Workflow Activities:**
```csharp
public class MyWorkflow : CodeActivity
{
    protected override void Execute(CodeActivityContext context)
    {
        // Workflow code
    }
}
```

**Custom APIs:**
```csharp
public class MyCustomAPI : IPlugin
{
    // Implements custom API logic
}
```

**PCF Controls:**
```typescript
export class MyControl implements ComponentFramework.StandardControl<IInputs, IOutputs>
{
    // PCF control code
}
```

**Output:** Categorized list of components with file paths

---

### Step 4: Analyze Plugins
Deep analysis of each plugin found.

**Checks performed:**

1. **Depth Validation:**
   - ✅ Has `if (context.Depth > 1) return;`
   - ❌ Missing depth check → Infinite loop risk

2. **Null Checks:**
   - ✅ Validates entity fields before access: `entity.Contains("field")`
   - ❌ Direct access without check → NullReferenceException risk

3. **Exception Handling:**
   - ✅ Uses try-catch with descriptive messages
   - ❌ No exception handling → Generic error messages

4. **Tracing Usage:**
   - ✅ Uses `ITracingService.Trace()`
   - ❌ No tracing → Hard to debug in production

5. **PreImage/PostImage Access:**
   - ✅ Validates image exists: `context.PreEntityImages.Contains("PreImage")`
   - ❌ Direct access → Runtime error if not configured

6. **Stage Appropriateness:**
   - ✅ Validation in PreValidation/PreOperation
   - ❌ Validation in PostOperation → Data already saved

**Output:** Per-plugin analysis with severity ratings

---

### Step 5: Detect Anti-Patterns
Pattern matching against known Dynamics anti-patterns.

**Anti-patterns detected:**

**HIGH Severity:**
- Missing depth validation
- Queries without paging
- Hardcoded GUIDs
- Missing null checks

**MEDIUM Severity:**
- Synchronous heavy operations
- Missing try-catch
- No ITracingService usage
- Unsafe PreImage/PostImage access

**LOW Severity:**
- Inefficient LINQ queries
- Not disposing IOrganizationService

**Uses:** `@knowledge/antipatterns.md` as reference

**Output:** List of anti-patterns with locations and fix suggestions

---

### Step 6: Analyze Pipeline Order
Identifies potential pipeline execution conflicts.

**Conflict scenarios detected:**

1. **Same Stage Multiple Plugins:**
   - Multiple plugins registered for same entity/message/stage
   - Risk: Execution order undefined
   - Recommendation: Set execution order explicitly

2. **Image Dependencies:**
   - Plugin accesses PreImage but might run in stage where it's unavailable
   - Plugin depends on PostImage in PreOperation stage
   - Risk: NullReferenceException

3. **Execution Order Issues:**
   - PluginB depends on PluginA's changes but runs before it
   - Workflow updates field that plugin validates (timing issue)

**Uses:** `@knowledge/pipeline-execution-order.md` as reference

**Output:** List of potential conflicts with impact assessment

---

### Step 7: Calculate Complexity
Computes code complexity metrics.

**Metrics calculated:**

1. **Cyclomatic Complexity:**
   - Number of decision paths in code
   - High complexity → Harder to test

2. **Lines of Code (LOC):**
   - Physical lines per file/method
   - Helps prioritize testing efforts

3. **Method Count:**
   - Number of methods per class
   - Large classes → Refactoring candidates

4. **Dependency Graph:**
   - Component dependencies
   - Helps identify integration test needs

**Output:** Complexity scores and hotspot identification

---

### Step 8: Generate Report
Compiles all analysis results into structured report.

**Report format options:**
- **Markdown** (default): Human-readable, great for documentation
- **JSON**: Machine-readable, for tool integration
- **Console**: Direct terminal output, for quick checks

**Report sections:**
1. Executive Summary
2. Components Inventory
3. Anti-Patterns Detected
4. Pipeline Conflicts
5. Complexity Analysis
6. Recommendations
7. Next Steps

**Uses:** `@templates/analysis-report.md` as template

---

### Step 9: Output Results
Presents results to user and saves report.

**Actions:**
1. Saves report to: `{{config.analysis_output_path}}/analysis-{timestamp}.{format}`
2. Displays summary in console/chat
3. Suggests next workflow to run

---

## Configuration Requirements

**Required config values:**
- `project_src_path`: Path to Dynamics source code
- `analysis_depth`: `quick`, `standard`, or `deep`
- `analysis_output_path`: Where to save reports

**Config example:**
```yaml
project_src_path: "C:/MyProject/src"
analysis_depth: standard
analysis_output_path: "C:/MyProject/docs/analysis"
```

---

## Inputs

### project_path (required)
- **Type:** string
- **Description:** Path to Dynamics project to analyze
- **Default:** From config `project_src_path`
- **Example:** `"C:/MyDynamicsProject/src"`

### analysis_depth (optional)
- **Type:** string
- **Options:** `quick`, `standard`, `deep`
- **Default:** From config `analysis_depth`
- **Description:**
  - `quick`: Fast scan, basic anti-patterns only
  - `standard`: Full component analysis + anti-patterns
  - `deep`: Everything + complexity metrics + dependency graph

### output_format (optional)
- **Type:** string
- **Options:** `markdown`, `json`, `console`
- **Default:** `markdown`
- **Description:** Format of the analysis report

---

## Outputs

### analysis_report
- **Type:** string
- **Description:** Complete analysis report content

### components_found
- **Type:** object
- **Structure:**
  ```json
  {
    "plugins": ["PluginA.cs", "PluginB.cs"],
    "workflows": ["WorkflowA.cs"],
    "custom_apis": ["CustomAPI.cs"],
    "pcf_controls": ["MyControl.tsx"]
  }
  ```

### antipatterns_detected
- **Type:** array
- **Structure:**
  ```json
  [
    {
      "severity": "HIGH",
      "pattern": "missing_depth_validation",
      "file": "PluginA.cs",
      "line": 25,
      "fix": "Add depth check: if (context.Depth > 1) return;"
    }
  ]
  ```

### pipeline_conflicts
- **Type:** array
- **Structure:**
  ```json
  [
    {
      "type": "same_stage_multiple_plugins",
      "entity": "account",
      "message": "Update",
      "stage": 20,
      "plugins": ["PluginA", "PluginB"],
      "risk": "Execution order undefined",
      "recommendation": "Set execution order in Plugin Registration Tool"
    }
  ]
  ```

---

## Error Handling

**Hooks executed on error:**
1. `log_error`: Saves error details to log
2. `notify_user`: Displays user-friendly error message
3. `save_partial_results`: Saves what was analyzed before error

**Common errors:**
- **Invalid path:** Clear message with example of correct path
- **No components found:** Suggests checking if path is correct
- **Permission denied:** Instructions to fix file permissions

---

## Success Hooks

**Hooks executed on success:**
1. `log_success`: Records successful analysis
2. `suggest_next_steps`: Recommends next workflow

**Next step suggestions:**
- If anti-patterns found → `qa-review` to get detailed recommendations
- If no tests exist → `qa-generate-tests` to create test suite
- If ready for full setup → `qa-quick-setup` for complete testing infrastructure

---

## Execution Time

**Estimates:**
- **Quick analysis:** 10-30 seconds
- **Standard analysis:** 30 seconds - 1 minute
- **Deep analysis:** 1-2 minutes

**Factors affecting time:**
- Project size (number of files)
- Analysis depth
- Number of components found

---

## Example Usage

### Via AVA Agent
```
User: [AN] Analyze
AVA: Vou analisar seu projeto! Qual o caminho?
User: C:/MyProject/src
AVA: *executes qa-analyze workflow*
AVA: Análise completa! Encontrei:
     - 5 plugins
     - 2 workflows
     - 3 anti-patterns (1 HIGH, 2 MEDIUM)
     - 1 conflito de pipeline potencial
     
     Relatório salvo em: docs/analysis/analysis-2025-12-03.md
     
     Próximo passo? [GT] para gerar testes ou [RV] para revisar anti-patterns?
```

### Via Direct Workflow Call
```bash
bmad-method run-workflow qa-analyze \
  --project-path "C:/MyProject/src" \
  --analysis-depth standard \
  --output-format markdown
```

---

## Integration with Other Workflows

**Feeds into:**
- `qa-generate-tests`: Uses component inventory to know what to test
- `qa-review`: Uses anti-pattern detection for detailed review
- `qa-quick-setup`: Uses full analysis to guide test generation

**Standalone value:**
- Can be run independently for project assessment
- Great for onboarding (understand project structure)
- Useful for architecture documentation

---

## Best Practices

1. **Run analysis first:** Before generating tests, understand project structure
2. **Use standard depth:** Unless you need speed (quick) or extreme detail (deep)
3. **Review conflicts:** Pipeline conflicts are often not obvious - review carefully
4. **Save reports:** Keep analysis history to track improvements over time
5. **Re-run after changes:** Analysis becomes stale as code evolves

---

## Tips for Large Projects

**For projects with 100+ files:**
- Use `quick` analysis first to get overview
- Then run `deep` analysis on specific hotspots
- Consider excluding non-critical paths in config

**For multi-solution projects:**
- Run analysis per solution/project
- Compare reports to find common patterns
- Use JSON output for aggregation
