# Workflow Phases & Coding Guidelines

Complete reference for all workflow phases, coding rules, anti-patterns, and error troubleshooting.

---

# Phase 1: Discovery

**Goal:** Understand available APIs and project context before writing any code.

## Step 1.1: Read Project Context and Current File

**ALWAYS start by reading `project.json`** to get the project name (for namespace) and installed dependencies:
```
ReadFileTool:
  filePath: <PROJECT_DIR>/project.json
```

**For EDIT requests — also read the target file:**
```
ReadFileTool:
  filePath: <workflow to edit>
```

**For CREATE requests:**
- Check project structure to determine appropriate file location
- Identify naming conventions from existing files

## Step 1.2: Discover Available Descriptors (Object Repository + UILibrary packages)

**Call this when the request involves UI automation, or when you're unsure whether descriptors are needed.** Skip for pure data processing, helper classes, or non-UI workflows.

```
GetProjectContextTool:
  queryType: "objects"
```

This returns all available descriptors from both the project's own Object Repository and any installed UILibrary NuGet packages (e.g. `*.UILibrary`, `*.Descriptors`), replacing the need to read `ObjectRepository.cs` directly.

**When to call:**
- The user asks for UI automation (click, type, open app, fill form, etc.)
- The user's request is ambiguous and might involve UI interaction
- The `project.json` dependencies include `UiPath.UIAutomation.Activities` or UILibrary packages

**When to skip:**
- Coded Source File (helper/model/utility) with no service dependencies
- Pure data processing (Excel, queues, assets, file I/O) with no UI interaction
- Test cases that only use `testing.*` assertions without UI

**Decision tree (when called):**
1. **Matching descriptor EXISTS** → Use it via `Descriptors.AppName.ScreenName.ElementName`
2. **Similar descriptor EXISTS** → Ask user which to use or if a new one is needed
3. **NO matching descriptor** → Ask user to create one in Object Repository

For the full Finding Descriptors hierarchy (3-step process), see [ui-automation/ui-automation.md](ui-automation/ui-automation.md#finding-descriptors).

## Step 1.3: API Discovery

**MANDATORY: Before generating any C# code, learn from existing project patterns first.**

### Step A: Search for Existing C# Files

```
FileSearchTool:
  regexQuery: ".*\\.cs$"
  rootDirectory: <absolute path to project root>
  maxResults: 100
```

- Count .cs files returned, excluding `./local/` and `./codedworkflows/` directories
- These folders contain generated/temporary files that should not count as examples

### Step B: Analyze Existing Files

- **1-4 files found** → READ all of them
- **5+ files found** → READ at least 5 diverse .cs files

Extract:
- Using statements
- Namespace patterns
- Class structure
- Service usage
- Argument patterns
- Logging patterns
- Error handling

## Discovery Checkpoint

Before proceeding, verify:
- [ ] Read `project.json` for namespace and dependencies
- [ ] If UI automation involved: called `GetProjectContextTool` with `queryType: "objects"`
- [ ] Used FileSearchTool for .cs files
- [ ] Counted results (excluding `./local/` and `./codedworkflows/`)
- [ ] If 5+ found, READ at least 5 using ReadFileTool
- [ ] Extracted common patterns

## Step 1.4: Use CodeGenerationPrerequisitesTool (Only If Necessary)

Call **ONLY IF** fewer than 5 .cs files found OR existing files don't show API usage **for the domain the user requested** (e.g., user asked for UI automation but all 5 files only use Excel).

```
Found >= 5 .cs files?
  ├─ YES: Do existing files use APIs relevant to the USER'S REQUEST?
  │   ├─ YES: Use patterns from existing files → SKIP CodeGenerationPrerequisitesTool
  │   └─ NO: Call CodeGenerationPrerequisitesTool with the user's request
  └─ NO: Call CodeGenerationPrerequisitesTool
```

**"Relevant" means the same domain as the user's request.** Five Excel workflows are NOT relevant if the user asked for a mail automation — call the tool in that case.

**If the tool returns UI Automation code with Descriptors**, you MUST replace generic descriptor paths with actual ones from the `GetProjectContextTool` results (Step 1.2).

## Understanding Available Services

Each service is injected by UiPath Studio **only when its corresponding NuGet package** is listed in `project.json` `dependencies`. Missing package = compile error (`CS0103`).

For the complete service-to-package mapping and versions → see [SERVICE_INDEX.md](SERVICE_INDEX.md).

---

# Phase 2: Generate or Edit

**Goal:** Create or modify C# code using patterns discovered in Phase 1.

## For CREATE Requests

Determine file type (see [codedworkflow-reference.md § Three Types](codedworkflow-reference.md#three-types-of-cs-files)), then generate using discovered APIs.

```
WriteFileTool:
  filePath: <appropriate path based on project structure>
  content: <C# code following proper structure>
```

### Test Case Specifics

- Use `testing.*` assertions: `VerifyExpression`, `VerifyAreEqual`, `VerifyContains`, `VerifyRange`
- For data-driven tests, add default parameter values to `Execute`
- For Before/After hooks in test suites → see [codedworkflow-reference.md § Before/After Hooks](codedworkflow-reference.md#beforeafter-hooks-shared-setupteardown)

### Code Templates

For ready-to-use code templates and full examples → [code-examples.md](code-examples.md)

## For EDIT Requests

Using the file content read in Step 1.1, apply changes:

```
EditFileTool:
  filePath: <workflow>
  edits: [
    { oldContent: <exact text from file>, newContent: <modified text> }
  ]
```

**Critical:** `oldContent` must match exactly and be unique in the file.

## Adding Dependencies

If the generated code uses a service not yet in `project.json`, add it before validation:

```
EditFileTool:
  filePath: <PROJECT_DIR>/project.json
  edits: [{ oldContent: <dependencies block>, newContent: <with new package added> }]
```

Use bracket notation for versions: `"PackageName": "[version]"`. See [SERVICE_INDEX.md](SERVICE_INDEX.md) for the canonical list of packages and versions.

## Generation Checklist

Before proceeding to validation:
- [ ] File path follows project conventions
- [ ] Namespace matches sanitized project name
- [ ] Using statements only include packages in project.json (add missing ones first)
- [ ] For workflows/test cases: inherits `CodedWorkflow`, has correct attribute
- [ ] For source files: NO base class, NO attribute

---

# Phase 3: Validate & Fix Loop

**Goal:** Iterate until `GetErrorsTool` returns 0 errors.

## Validation Loop

```
REPEAT:
  1. GetErrorsTool (onlyCurrentFile: true)
  2. IF 0 errors → EXIT to Phase 4
  3. GetQuickFixesTool for suggestions
  4. GetTypeDefinitionsTool for type issues
  5. ReadFileTool to understand current content
  6. Apply fix with EditFileTool
  7. GOTO 1

DO NOT stop until GetErrorsTool returns 0 errors.
DO NOT skip validation steps.
DO NOT assume edits worked without checking.
```

Expect 3-7 iteration cycles for complex workflows.

## Error Fix Priority

Fix errors in this order:
1. **Syntax errors** — Missing semicolons, braces, parentheses
2. **Type errors** — `CS0246`, `CS0029`, `CS1061`
3. **Logic errors** — `CS0161`, `CS0128`

For the full error lookup table → see [Common Issues and Fixes](#common-issues-and-fixes) below.

## Unrecoverable Errors (Bail Out)

Stop retrying and ask the user when encountering:

| Error Type | Action |
|------------|--------|
| Missing NuGet package (user must add) | "Please add `<Package>` to Project Dependencies in Studio" |
| ObjectRepository.cs missing | "This project isn't set up for coded workflows. Please enable coded workflows in Studio." |
| Descriptors from unknown UILibrary | "I found references to `<Package>` but it's not installed. Please add it to project dependencies." |

## Max Retry Policy

- **Validation loop:** Max 5 fix attempts per error
- **After 5 failed attempts:** Present remaining errors to user with explanation
- **Do NOT:** Make unrelated changes during retry — only fix the specific error

---

# Phase 4: Run & Test (Optional)

**Goal:** Execute the workflow to verify runtime behavior.

## Running a Workflow

After reaching 0 compilation errors, optionally run:

```
RunWorkflowTool:
  filePath: <full path to the .cs file>
```

## When to Run

**Run when:**
- User requests execution
- Logic should be verified against actual data
- User wants to see output or results

**Don't run when:**
- Requires unavailable external dependencies (database, API, credentials)
- Performs destructive operations (delete files, send emails, modify data)
- User only asked for code generation
- Requires UI interaction that can't be automated

## Handling Runtime Errors

If a runtime error occurs:

1. **Analyze** the error message and stack trace
2. **Identify** the problematic code location
3. **Fix** with EditFileTool
4. **Re-validate** with GetErrorsTool until 0 errors
5. **Re-run** with RunWorkflowTool
6. **Max retries:** 2 execution attempts before asking user for guidance

## Common Runtime Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `FileNotFoundException` | Input file path incorrect | Verify file exists, check path |
| `NullReferenceException` | Uninitialized variable or null return | Add null checks |
| `ConnectionHttpException` | Integration Service token expired | Ask user to re-authorize connection |
| `SelectorNotFoundException` | UI element not found | Ask user to verify/update selector in Object Repository |
| `TimeoutException` | Operation took too long | Increase timeout or check target availability |

---

# Phase 5: Response

**Goal:** Provide a concise summary to the user after completing the task.

After completing the task, include:

1. **File path** of created/edited file
2. **Brief description** of what it does
3. **Key APIs used** and their purposes
4. **Execution results** (if run)
5. **Limitations or blockers** (if any)

---

# Reference: Using Statements Rules

**CRITICAL: Only include `using` statements for namespaces actually used in the file.** Adding usings for packages not in `project.json` will cause compile errors.

**Minimal using statements** (always safe in any workflow/test case file):
```csharp
using System;
using System.Collections.Generic;
using UiPath.CodedWorkflows;
```

**Add based on actual usage** — only include these when the file uses the corresponding types/services AND the package is in `project.json`:
```csharp
// If using system.* service (UiPath.System.Activities package):
using UiPath.Core;
using UiPath.Core.Activities.Storage;       // only if using storage APIs
using UiPath.Orchestrator.Client.Models;    // only if using Orchestrator models

// If using testing.* service (UiPath.Testing.Activities package):
using UiPath.Testing;
using UiPath.Testing.Enums;                 // only if using testing enums
using UiPath.Testing.Activities.TestData;   // only if using test data queues

// If using uiAutomation.* service (UiPath.UIAutomation.Activities package):
using UiPath.UIAutomationNext.API.Contracts;
using UiPath.UIAutomationNext.API.Models;
using UiPath.UIAutomationNext.Enums;

// If using Object Repository descriptors (Descriptors.App.Screen.Element):
using <ProjectNamespace>.ObjectRepository;  // e.g. using RoboticEnterpriseFramework.ObjectRepository;
// OR if descriptors come from a UILibrary NuGet package (not the project's own OR):
// using <PackageNamespace>.ObjectRepository;  // e.g. using MultipleApps.Descriptors.ObjectRepository;
// CRITICAL: Without this, you get CS0103: The name 'Descriptors' does not exist in the current context
// NOTE: When descriptors come from a UILibrary package, use the PACKAGE namespace, not the project namespace

// If using excel.* service (UiPath.Excel.Activities package):
using UiPath.Excel;
using UiPath.Excel.Activities;
using UiPath.Excel.Activities.API;
using UiPath.Excel.Activities.API.Models;

// If using word.* service (UiPath.Word.Activities package):
using UiPath.Word;
using UiPath.Word.Activities;
using UiPath.Word.Activities.API;
using UiPath.Word.Activities.API.Models;

// If using powerpoint.* service (UiPath.Presentations.Activities package):
using UiPath.Presentations;
using UiPath.Presentations.Activities;
using UiPath.Presentations.Activities.API;
using UiPath.Presentations.Activities.API.Models;

// If using mail.* service (UiPath.Mail.Activities package):
using UiPath.Mail.Activities.Api;

// If using office365.* service (UiPath.MicrosoftOffice365.Activities package):
using UiPath.MicrosoftOffice365.Activities.Api;

// If using google.* service (UiPath.GSuite.Activities package):
using UiPath.GSuite.Activities.Api;

// Standard .NET (add as needed):
using System.Data;           // DataTable
using System.Linq;           // LINQ
using System.IO;             // file operations
using System.Text.RegularExpressions;  // regex
```

**When adding a file that uses a service:**
1. Check `project.json` to confirm the required package is listed in `dependencies` — add it if missing
2. Add only the `using` statements needed for the types actually referenced in the file

---

# Reference: Best Practices

### Code Quality
- **Start simple, iterate** — Create minimal working version first, then refine
- **Only include using statements for packages in project.json** — Adding unused usings causes compile errors
- **Match input parameter names exactly** — Execute method signature parameter names are case-sensitive

### File Operations
- **ALWAYS read a file before editing it** — Understand current state before making changes
- **Prefer editing over creating new files** — Build on existing work, avoid file bloat

---

# Reference: Anti-Patterns (What NOT to Do)

### Project & Code Structure

- Never generate C# code without first searching for existing .cs files (API Discovery)
- Never edit files without reading them first
- Never skip the `[Workflow]` or `[TestCase]` attribute on the Execute method
- Never forget to inherit from `CodedWorkflow` (except Coded Source Files)
- Never add `using` statements for packages not in `project.json` — causes CS errors
- Never guess service method names — verify with existing code, `GetTypeDefinitionsTool`, or `CodeGenerationPrerequisitesTool`

### UI Automation

- Never hardcode UI selectors — use Object Repository descriptors
- Never write UI code referencing descriptors without first calling `GetProjectContextTool` with `queryType: "objects"` to discover all available descriptors
- Never skip checking for missing descriptors — ask the user to add them in Studio's Object Repository
- Never use UITask (ScreenPlay) as the primary approach — resolve descriptors via Finding Descriptors hierarchy first
- Never use an element descriptor on the wrong screen handle — each `UiTargetApp` is bound to its screen. Wrong handle gives `"Target name 'X' is not part of the current screen."`
- Never use `SelectItem` on web dropdowns without a `TypeInto` fallback — web `<select>` elements often fail with `"Cannot select item"`
- Never forget `using <ProjectNamespace>.ObjectRepository;` (or `using <PackageName>.ObjectRepository;` for UILibrary packages) when referencing `Descriptors.*`
- Never bypass Object Repository by constructing `TargetAppModel` with raw URL/BrowserType

### Validation & Execution

- Never assume create/edit succeeded without checking with `GetErrorsTool`
- Never continue retrying indefinitely — stop after 5 validation fix attempts or 2 runtime execution retries
- Never make unrelated changes during retry — only fix the specific error

---

# Reference: Common Issues and Fixes

### Compilation Errors

| Issue | Cause | Fix |
|-------|-------|-----|
| `CS0246: Type not found` | Missing using statement | Add appropriate `using` directive |
| `CS1061: Does not contain definition` | Wrong method name or missing service | Check existing .cs files for correct API patterns, or use GetTypeDefinitionsTool |
| `CS0029: Cannot convert type` | Type mismatch | Use GetTypeDefinitionsTool to check expected types |
| `CS0103: Name does not exist` | Undefined variable/property or missing package | Declare variable, check spelling, or add package to `project.json` |
| `CS0161: Not all paths return value` | Missing return statement | Add return for all code paths |
| `CS0128: Local variable already defined` | Duplicate variable name | Rename or remove duplicate |
| `[Workflow]` not recognized | Missing UiPath.CodedWorkflows reference | Ensure project has correct packages |
| `[TestCase]` not recognized | Missing UiPath.Testing.Activities package | Add `UiPath.Testing.Activities` to `project.json` dependencies |
| `testing` service not available | Missing UiPath.Testing.Activities package | Add `UiPath.Testing.Activities` to `project.json` dependencies |
| Service property not available | Not inheriting from CodedWorkflow, or missing package | Add `: CodedWorkflow` to class declaration and/or add package to `project.json` |
| Service used in Coded Source File | Source files don't have service access | Move logic to a Coded Workflow or pass services as parameters |
| `IBeforeAfterRun` not recognized | Missing using statement | Add `using UiPath.CodedWorkflows;` |

### UI Automation Errors

| Issue | Cause | Fix |
|-------|-------|-----|
| Object Repository element not found | Wrong descriptor path | Check exact path in Object Repository via `GetProjectContextTool` |
| `Descriptors` does not contain element | Selector not in Object Repository | Ask user to add the element to Object Repository in Studio |
| UI element not found at runtime | Selector doesn't match actual UI | Ask user to update/re-capture the selector in Studio |
| **"Target name 'X' is not part of the current screen"** | Element descriptor used on wrong screen handle | Use the `UiTargetApp` handle from `Open`/`Attach` for the screen that owns the element |
| **"Cannot select item. It was not found among existing items"** | `SelectItem` fails on web dropdowns | Use `TypeInto` instead of `SelectItem` for web `<select>` elements |

### Studio Errors

| Issue | Cause | Fix |
|-------|-------|-----|
| **Workflow cannot be found** | Entrypoint not in project.json | Verify project.json entrypoint has the file listed |
| **Service property not available** | Missing package dependency | Add required package to project.json dependencies |
