# AVA Dynamics QA

> **Pipeline-aware QA automation for Microsoft Dynamics 365**

AVA (**A**utomated **V**alidation **A**ssistant) is your AI partner for Dynamics 365 quality assurance. She understands that QA in Dynamics isn't just about testing code—it's about testing how multiple artifacts interact through the execution pipeline.

---

## 🎯 What AVA Does

- ✅ **Analyzes** Dynamics projects (plugins, workflows, APIs, PCF)
- ✅ **Generates** unit & integration tests with FakeXrmEasy mocks
- ✅ **Detects** Dynamics-specific anti-patterns before they become bugs
- ✅ **Validates** pipeline execution order and potential conflicts
- ✅ **100% Standalone** - zero external dependencies

---

## 🚀 Quick Start

### 1. Install the Module

```bash
# Clone this repository
git clone <your-repo-url>

# Navigate to project
cd your-dynamics-project

# AVA is ready - no installation needed!
```

### 2. Run AVA

```bash
# Option 1: Quick setup (all-in-one)
ava *quick-setup

# Option 2: Step by step
ava *analyze              # Analyze project
ava *generate-tests       # Generate tests  
ava *review               # Code review
```

### 3. Tests Created! 

AVA creates:
- `tests/` - Complete test project
- Unit tests for all plugins, workflows, APIs
- Integration tests for multi-artifact flows
- FakeXrmEasy setup and mocks
- Code quality report

---

## 💡 Key Features

### Pipeline Intelligence
AVA understands Dynamics execution order:
- Pre-validation → Pre-operation → Post-operation
- Sync vs Async plugins
- Plugin → Workflow → Business Rule interactions
- PreImage/PostImage dependencies

**Result**: Catch pipeline conflicts before deployment!

### Smart Test Generation
Not just templates—intelligent analysis:
- Detects critical code paths
- Prioritizes high-risk areas
- Creates realistic test scenarios
- Proper FakeXrmEasy mocks for all Dynamics contexts

### Anti-Pattern Detection
Catches common Dynamics mistakes:
- ⚠️ Missing depth validation (infinite loops)
- ⚠️ Queries without paging (performance)
- ⚠️ Hardcoded GUIDs (maintainability)
- ⚠️ Missing null checks (runtime errors)
- ⚠️ Sync code that should be async

### Zero Setup Required
- No API keys
- No external services
- No accounts or logins
- Works offline
- Git clone and go!

---

## 📋 Workflows

### `*quick-setup`
**All-in-one test suite generation**

```
What it does:
1. Scans your src/ directory
2. Identifies all Dynamics components
3. Generates complete test project
4. Creates unit + integration tests
5. Runs code review
6. Outputs quality report

Perfect for: First-time setup, POCs, demos
Time: ~2-5 minutes for typical project
```

### `*analyze`
**Deep project analysis**

```
What it does:
1. Maps all plugins, workflows, APIs, PCF
2. Identifies dependencies
3. Analyzes execution order
4. Detects potential conflicts
5. Generates dependency graph

Perfect for: Understanding complex projects
Output: analysis-report.md with full details
```

### `*generate-tests`
**Intelligent test generation**

```
What it does:
1. Analyzes code complexity
2. Identifies critical paths
3. Creates unit tests (isolated components)
4. Creates integration tests (multi-artifact flows)
5. Sets up FakeXrmEasy infrastructure

Perfect for: Adding tests to existing code
Frameworks: XUnit, NUnit, MSTest (your choice)
```

### `*review`
**Automated code review**

```
What it does:
1. Scans for Dynamics anti-patterns
2. Checks performance issues
3. Validates best practices
4. Assigns quality score
5. Provides actionable suggestions

Perfect for: Pre-PR validation, quality gates
Output: review-report.md with scores and fixes
```

---

## 🎓 Examples

### Example 1: New Dynamics Project

```bash
# You just created a new plugin
# Need tests fast!

ava *quick-setup

# AVA responds:
# 🔍 Analyzing src/...
# Found: 1 plugin, 0 workflows
# 
# 🧪 Generating tests...
# Created: 5 unit tests
# Created: tests/ContactValidationPluginTests.cs
# 
# ✅ Done! Run: dotnet test
```

### Example 2: Complex Multi-Plugin Project

```bash
# Large project, many components
# Want to understand interactions

ava *analyze

# AVA responds:
# 📊 Project Analysis Complete
# 
# Components Found:
# - 23 Plugins
# - 12 Workflows  
# - 5 Custom APIs
# - 3 PCF Controls
#
# ⚠️ 2 Potential Pipeline Conflicts:
# 1. ContactUpdate: Plugin A & Plugin B (same stage)
# 2. AccountCreate: Workflow depends on missing PreImage
#
# 📄 Full report: analysis-report.md
```

### Example 3: Code Review Before PR

```bash
# About to create PR
# Want to check quality

ava *review

# AVA responds:
# 🔍 Reviewing code...
#
# Quality Score: 7.5/10
#
# Issues Found:
# ⚠️ High Priority (2):
#   - Missing depth check in ValidateEmail.cs:45
#   - Query without paging in GetContacts.cs:89
#
# 💡 Medium Priority (3):
#   - Hardcoded GUID in UpdateAccount.cs:123
#   - Missing try-catch in ProcessOrder.cs:67
#   - Sync plugin could be async: NotifyTeam.cs:34
#
# 📄 Full report: review-report.md
```

---

## 🏗️ Module Structure

```
ava-dynamics-qa/
├── agents/
│   └── ava.md                    # AVA agent definition
├── workflows/
│   ├── qa-quick-setup/           # All-in-one workflow
│   ├── qa-analyze/               # Analysis workflow
│   ├── qa-generate-tests/        # Test generation workflow
│   └── qa-review/                # Code review workflow
├── templates/
│   ├── test-project-template/    # Base test project structure
│   └── test-class-template.cs    # Test class templates
├── data/
│   └── antipatterns.json         # Anti-pattern database
├── _module-installer/
│   └── install-config.yaml       # Installation configuration
└── README.md                     # This file
```

---

## ⚙️ Configuration

During installation, you'll configure:

- **Source Path**: Where your Dynamics code lives (default: `src/`)
- **Test Path**: Where to create tests (default: `tests/`)
- **Test Framework**: XUnit, NUnit, or MSTest
- **FakeXrmEasy Version**: Latest, v9, or v3
- **Analysis Depth**: Quick, Standard, or Deep
- **Integration Tests**: Yes or No
- **Auto Review**: Yes or No

All stored in `.bmad/ava-dynamics-qa/config.yaml`

---

## 🎯 Personas & Use Cases

### Maria (Junior Dev)
**Need**: Learn testing, structured guidance
**Uses**: `*quick-setup` with tutorial comments
**Benefit**: Tests created + learns proper structure

### Carlos (Senior Dev)
**Need**: Speed, no friction, CI/CD ready
**Uses**: `*generate-tests` for fast iteration
**Benefit**: Tests in minutes, not hours

### Ana (Tech Lead)
**Need**: Team consistency, quality metrics
**Uses**: `*review` before PRs, standards enforcement
**Benefit**: Consistent quality across team

### Roberto (Architect)
**Need**: Cross-project portability, zero setup
**Uses**: Clone module to each project
**Benefit**: Same QA standards everywhere

---

## 🔮 Roadmap

### ✅ MVP (Current - v1.0)
- Project analysis
- Unit test generation
- Integration test generation
- Code review
- Anti-pattern detection
- Pipeline intelligence

### 🚧 Phase 2 (Planned)
- Mutation testing
- Visual dashboards
- Metrics & trends
- Tutorial mode for juniors
- Fast mode for seniors
- VS Code integration

### 💭 Phase 3 (Future)
- Machine learning patterns
- Real-time IDE feedback
- Team collaboration features
- Custom rule engine

---

## 🤝 Contributing

Want to improve AVA?

1. Fork the repository
2. Create feature branch
3. Make your changes
4. Submit pull request

---

## 📄 License

MIT License - use freely in your projects

---

## 💬 Support

Need help? Found a bug?

- 📧 Open an issue in the repository
- 💬 Ask AVA: `ava *help`
- 📚 Check the examples above

---

## 🙏 Credits

Built with ❤️ using the BMAD Method

Special thanks to:
- FakeXrmEasy team for excellent mocking framework
- Dynamics community for insights and feedback
- All the QA engineers who inspired this tool

---

**Let's make Dynamics development bulletproof! 🚀**

_Your code. Your tests. Your quality. Automated by AVA._
