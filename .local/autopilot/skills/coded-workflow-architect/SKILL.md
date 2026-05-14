---
name: coded-workflow-architect
description: Comprehensive workflow for generating and editing Coded Workflows, Coded Test Cases, and Coded Source Files (C# .cs files) in UiPath Studio Desktop. Use this when users need to create new C# automations, test cases, helper/utility classes, modify existing coded workflows, fix C# errors, or iterate on coded workflow implementations. Supports discovery-first approach with error-driven refinement.
icon: FaCode
color: "#0078D4"
---

# Coded Workflow Architect

Generate and edit Coded Workflows, Coded Test Cases, and Coded Source Files (C# .cs files) using a **discovery-first approach** with **iterative error-driven refinement**.

## Loading Strategy

**Read files in this order. Stop when you have enough context for the task.**

1. **ALWAYS read:** [codedworkflow-reference.md](references/codedworkflow-reference.md) (file formats, namespaces, arguments, built-in methods)
2. **ALWAYS read:** [SERVICE_INDEX.md](references/SERVICE_INDEX.md) (service → package mapping)
3. **ALWAYS read:** [coding-guidelines.md](references/coding-guidelines.md) (workflow phases, using statements, anti-patterns, error fixes)
4. **Read domain docs ONLY when the user's request involves that domain:**
   - UI automation → [ui-automation/ui-automation.md](references/ui-automation/ui-automation.md), then windows-api.md / examples.md as needed
   - Excel/Word/Mail/etc. → the corresponding folder under references/
5. **Read for code templates:** [code-examples.md](references/code-examples.md) (generic templates — for domain-specific examples, use the domain's examples.md instead)

**DO NOT** load all reference files. Load only what the current task requires.

---

## Core Principles

1. **API Discovery Before Generation** — Never generate C# code without first reading `project.json` and searching for existing .cs files
2. **Start Simple, Iterate** — Create minimal working version first, then refine through validation cycles
3. **Validate After Every Change** — Always check with GetErrorsTool after any create/edit
4. **Fix Errors Methodically** — Syntax → Type → Logic order, max 5 attempts before asking user

---

## Tool Quick Reference

| Tool | Purpose |
|------|---------|
| **FileSearchTool** | Find .cs files by regex (MANDATORY first step) |
| **ReadFileTool** | Read file contents with line numbers |
| **WriteFileTool** | Create new file |
| **EditFileTool** | Edit existing file via string replacement |
| **GetErrorsTool** | Check for compilation errors |
| **GetQuickFixesTool** | Get quick fix suggestions |
| **GetTypeDefinitionsTool** | Get type info at specific location |
| **GetProjectContextTool** | Get project info including Object Repository and UILibrary descriptors |
| **RunWorkflowTool** | Run/debug a workflow file |
| **CodeGenerationPrerequisitesTool** | Get APIs (ONLY if <5 .cs files found) |

---

## Workflow Phases

All phases are detailed in [coding-guidelines.md](references/coding-guidelines.md). Summary:

| Phase | Goal |
|-------|------|
| **1. Discovery** | Read project.json, discover APIs and descriptors |
| **2. Generate/Edit** | Create or modify C# code, add dependencies |
| **3. Validate & Fix** | Iterate until 0 errors |
| **4. Run & Test** | Execute workflow (optional, only if user requests) |
| **5. Response** | Summarize what was done |

---

## Request Classification

**Step 1: Identify file type** — Coded Workflow, Coded Test Case, or Coded Source File. See [codedworkflow-reference.md § Three Types](references/codedworkflow-reference.md#three-types-of-cs-files) for the full comparison (base class, attributes, service access).

**Step 2: Identify action** — CREATE (generate/create/make/build/new) or EDIT (update/change/fix/modify/add to).

If unclear → **ask the user** rather than guessing.

