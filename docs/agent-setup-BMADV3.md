## Agent Workspace Setup

### Agent Type

Expert Agent

### Workspace Configuration

Complete sidecar structure created for persistent memory, knowledge base, and personal workflows.

### Setup Elements

**Memory and Session Management:**
- `memories.md` - Persistent memory bank for tracking plugins tested, user preferences, session history, and project patterns

**Knowledge Base Structure:**
- `knowledge/project-patterns.md` - Learned naming conventions, project structures, and framework preferences
- `knowledge/test-templates.md` - Customizable test templates that evolve with usage
- `knowledge/best-practices.md` - Consolidated best practices for Dynamics 365 testing

**Private Instructions:**
- `instructions.md` - Core directives, operating principles, domain boundaries, and session protocols

**Personal Workflow Capabilities:**
- `workflows/` folder ready for 5 specialized workflows:
  - generate-tests.md
  - analyze-plugin.md
  - review-tests.md
  - coverage-report.md
  - teach-practices.md

**Learning and Adaptation Framework:**
- Memory integration for remembering past testing sessions
- Knowledge base growth with project-specific patterns
- Template evolution based on feedback
- Session-to-session continuity

### Location

**Main Agent Location:**
`{project-root}/.bmad/custom/src/agents/dynamics-qa-expert/`

**Sidecar Location:**
`{project-root}/.bmad/custom/src/agents/dynamics-qa-expert/dynamics-qa-expert-sidecar/`

**Complete Structure:**
```
dynamics-qa-expert/
├── dynamics-qa-expert.agent.yaml (to be created)
└── dynamics-qa-expert-sidecar/
    ├── memories.md ✅
    ├── instructions.md ✅
    ├── knowledge/
    │   ├── project-patterns.md ✅
    │   ├── test-templates.md ✅
    │   └── best-practices.md ✅
    └── workflows/
        └── (workflows to be created in next steps)
```

### Workspace Features

**Memory Persistence:** Marcos will remember conversations, plugins tested, and user preferences across sessions

**Knowledge Growth:** The knowledge base will expand as Marcos learns project-specific patterns and conventions

**Adaptive Learning:** Templates and patterns evolve based on successful test generations and user feedback

**Privacy and Security:** Domain restrictions ensure sidecar files remain private while test generation happens in appropriate project folders

---

**Status:** Workspace successfully configured and ready for agent finalization! 🎉
