# UI Automation Activities API Reference

Reference for the `uiAutomation` service from `UiPath.UIAutomation.Activities` package.

**Required package:** `"UiPath.UIAutomation.Activities": "[25.10.21]"`

**Auto-imported namespaces:** `System`, `UiPath.UIAutomationNext.API.Contracts`, `UiPath.UIAutomationNext.Enums`, `UiPath.UIAutomationNext.API.Models`

**Service accessor:** `uiAutomation` (type `IUiAutomationAppService`)

---

## Overview

The UI Automation API provides coded workflow access to UI interactions with desktop applications, web browsers, and SAP. The main entry point is `IUiAutomationAppService`, accessed via the `uiAutomation` service accessor.

### Workflow Pattern

1. **Open** or **Attach** to an application screen — returns a `UiTargetApp` handle.
2. Use the `UiTargetApp` handle to perform element interactions (Click, TypeInto, GetText, etc.).
3. The `UiTargetApp` is `IDisposable` — use `using` blocks or dispose manually.

### Screen Handle Affinity (Critical)

**Each `UiTargetApp` handle is bound to a specific screen.** Element descriptors can ONLY be used with the handle for the screen they belong to. Using a descriptor from Screen A on a handle attached to Screen B will fail with `"Target name 'X' is not part of the current screen."`.

```csharp
// CORRECT — use Home elements on the homeScreen handle
var homeScreen = uiAutomation.Open(Descriptors.MyApp.Home);
homeScreen.Click(Descriptors.MyApp.Home.Products);   // OK — Products is a Home element
homeScreen.Click(Descriptors.MyApp.Home.Loans);       // OK — Loans is a Home element

// Then attach to the next screen for its elements
var formScreen = uiAutomation.Attach(Descriptors.MyApp.Form);
formScreen.TypeInto(Descriptors.MyApp.Form.Email, "test@example.com");  // OK

// WRONG — using a Home element on the Form screen handle
formScreen.Click(Descriptors.MyApp.Home.Loans);  // FAILS: "Target name 'Loans' is not part of the current screen"
```

**When navigating multi-screen flows:** perform all interactions for one screen before attaching to the next. If the page changes after a click, attach to the new screen and use that handle for subsequent interactions.

### Target Resolution

Each method on `UiTargetApp` accepts targets in multiple forms:
- **`string target`** — a target name defined in the Object Repository screen.
- **`IElementDescriptor elementDescriptor`** — a strongly-typed Object Repository descriptor (e.g., `Descriptors.MyApp.LoginScreen.Username`).
- **`TargetAnchorableModel target`** — accessed via the `UiTargetApp` indexer: `app["targetName"]` or `app[Descriptors.MyApp.Screen.Element]`.
- **`RuntimeTarget target`** — a runtime target returned by `GetChildren` or `GetRuntimeTarget`.

### Finding Descriptors

**MANDATORY for any workflow that uses `uiAutomation.*` calls.** Follow this decision tree in **strict order** — stop at the first step that yields the descriptor you need.

> **CRITICAL:** Steps 1 → 2 → 3 MUST be followed sequentially. You MUST NOT skip to Step 3 (UITask) without completing Steps 1-2. Even if a form has many fields, indicate ALL of them in Step 2 rather than falling back to UITask. UITask is ONLY for genuinely brittle selectors where indication fails to produce reliable results.

---

#### Step 1 — Discover all available descriptors via GetProjectContextTool

**Call `GetProjectContextTool` with `queryType: "objects"`** to get the complete descriptor hierarchy. This returns descriptors from **both** the project's own Object Repository **and** any installed UILibrary NuGet packages (e.g. `*.UILibrary`, `*.Descriptors`) in a single call.

```
GetProjectContextTool:
  queryType: "objects"
```

> **Note:** This should already have been called during Phase 1 Discovery (Step 1.2). If you have the results from that call, reuse them — no need to call again.

- If the descriptor you need is present → **use it directly** and stop here.
- If the required screen/element is missing → proceed to Step 2.

**Important: Add the correct ObjectRepository using statement** — any workflow file that references `Descriptors.*` needs:
```csharp
// For descriptors from the project's own Object Repository:
using <ProjectNamespace>.ObjectRepository;  // e.g. using RoboticEnterpriseFramework.ObjectRepository;

// For descriptors from a UILibrary NuGet package:
using <PackageNamespace>.ObjectRepository;  // e.g. using MultipleApps.Descriptors.ObjectRepository;
```
Without this, you get `CS0103: The name 'Descriptors' does not exist in the current context`.

**UILibrary package descriptors — additional notes:**
- The `using` statement uses the **package namespace**, NOT the project namespace
- App names with spaces are sanitized to use underscores: `"UiPath Banking App"` → `Descriptors.UiPath_Banking_App`
- The hierarchy is: `Descriptors.<App>.<Screen>.<Element>`
- If the package is not yet in `project.json`, add it before using its descriptors

---

#### Step 2 — Ask the user to add the element via Studio

If the required descriptor is not found in the `GetProjectContextTool` results (neither from the project's Object Repository nor from any UILibrary package), **ask the user** to add the element using Studio's Object Repository panel or the Indicate in App feature.

**What to ask the user:**

| Scenario | What to say |
|----------|-------------|
| Missing screen | "I need a screen descriptor for [screen description]. Please add it to the Object Repository in Studio using 'Indicate in App' and let me know when it's done." |
| Missing element | "I need an element descriptor for [element description] on the [screen name] screen. Please add it to the Object Repository in Studio and let me know the descriptor path." |
| Missing app | "I need to automate [application name] but it's not in the Object Repository. Please add the application and its screens/elements in Studio's Object Repository panel." |

After the user adds elements, call `GetProjectContextTool` with `queryType: "objects"` again to refresh the available descriptors.

- If the descriptors are now available → **use them** and stop here.
- If selectors are brittle or the UI is dynamic and hard to target precisely → proceed to Step 3.

> **IMPORTANT:** Do NOT jump to Step 3 because adding elements "seems tedious." Even for complex forms with 10+ fields, ask the user to add ALL missing screens and elements in Step 2. UITask is non-deterministic, slower, and should only be used when selectors are genuinely unreliable (e.g., dynamic IDs that change every page load). Also, NEVER bypass the Object Repository by constructing `TargetAppModel` with raw URL/BrowserType — always use descriptors.

---

#### Step 3 — Use ScreenPlay (AI fallback, last resort only)

ScreenPlay (`UITask`) is an AI-powered agent that interprets natural-language instructions and performs UI interactions without requiring precise selectors. Use it **only** when step 2 selectors are not reliable enough — it is slower and non-deterministic compared to Object Repository descriptors.

**UITask API** — called on a `UiTargetApp` handle:

```csharp
// Simple — always pass agentType explicitly; the default DOMBased is deprecated
UITaskResult UITask(string task, NUITaskAgentType agentType = NUITaskAgentType.DOMBased, NChildInteractionMode interactionMode = NChildInteractionMode.SameAsCard)

// With full options
UITaskResult UITask(UITaskOptions options)

// Async variants
Task<UITaskResult> UITaskAsync(string task, ...)
Task<UITaskResult> UITaskAsync(UITaskOptions options)
```

**UITaskOptions** — created via `Options.UITask(task).With(...)`:

| Property | Type | Default | Description |
|---|---|---|---|
| `Task` | `string` | required | Natural-language instruction for the AI agent |
| `AgentType` | `NUITaskAgentType` | `DOMBased` | AI model to use |
| `InteractionMode` | `NChildInteractionMode` | `SameAsCard` | How the agent interacts with UI elements |
| `MaxIterations` | `int` | project default | Max iterations before giving up (0 = no limit) |
| `ClipboardMode` | `NTypeByClipboardMode` | project default | Whether the agent can use clipboard for typing |
| `IsDOMEnabled` | `bool` | project default | Whether the agent can use native DOM of the target app |
| `IsVariableSecurityEnabled` | `bool` | `true` | Prevents prompt injection via variables |
| `TraceAttachMode` | `NTraceAttachMode` | project default | When to attach trace files to the job |

**UITaskResult:**

| Property | Type | Description |
|---|---|---|
| `Status` | `string` | Execution status (`"Succeeded"` or failure) |
| `Result` | `string` | String result returned by the agent |
| `ErrorMessage` | `string?` | Error message (null if successful) |
| `HttpErrorStatusCode` | `string?` | HTTP status code for server-side errors |
| `TraceFiles` | `NUITaskTraceFiles?` | Trace files (based on ScreenPlay settings) |

**NUITaskAgentType:**

| Value | Model | DOM Support |
|---|---|---|
| `GeminiFlash25` | `gemini-2.5-flash` | Yes |
| `DOMBasedGPT41` | `gpt-4.1` | Yes |
| `DOMBasedGPT41Mini` | `gpt-4.1-mini` | Yes |
| `GPT5` | `gpt-5` | Yes |
| `GPT5Mini` | `gpt-5-mini` | Yes |
| `OpenAIOperator` | `openai-computer-use` | No |
| `AnthropicClaudeCU` | `claude-computer-use` | No |
| `DOMBased` | `gemini-2.0-flash` | Yes *(deprecated)* |

**NTraceAttachMode:** `OnFailure` · `Always` · `Never`

**Example:**

```csharp
var app = uiAutomation.Open(Descriptors.MyApp.Main);

// Simple call
var result = app.UITask("Click the Submit button", NUITaskAgentType.GeminiFlash25);

// With options
var result = app.UITask(Options.UITask("Fill in the form with name John Doe")
    .With(agentType: NUITaskAgentType.GPT5, maxIterations: 10, isDOMEnabled: true));

if (result.Status == "Succeeded")
    Log($"Agent completed: {result.Result}");
else
    Log($"Agent failed: {result.ErrorMessage}");
```

See [examples.md](examples.md) for full coded workflow examples using UITask.

---

## Common Pitfalls

### Web Dropdowns and `SelectItem`

`SelectItem` may fail on web-based `<select>` dropdowns with `"Cannot select item. It was not found among existing items."` — even after clicking the dropdown first to expand it. This is common with custom/styled web dropdowns that don't expose their items via standard accessibility APIs.

**Workaround:** Use `TypeInto` instead of `SelectItem` for web dropdowns. `TypeInto` on a `<select>` element types the value directly, which triggers the browser's native matching:

```csharp
// Instead of:
// formScreen.SelectItem(Descriptors.MyApp.Form.Term, "12");  // May fail on web dropdowns

// Use:
formScreen.TypeInto(Descriptors.MyApp.Form.Term, "12");  // Works reliably
```

### Screen Handle Mismatch

Using an element descriptor on the wrong screen handle causes `"Target name 'X' is not part of the current screen."`. Always ensure you use the correct handle for each screen's elements. See **Screen Handle Affinity** above.

---

## API Reference

See [windows-api.md](windows-api.md) for the full API reference including:
- Opening and attaching to applications
- Element interactions (Click, TypeInto, GetText, Hover, Check, SelectItem, etc.)
- Keyboard shortcuts
- Mouse scrolling and drag-and-drop
- Screenshots and highlighting
- Browser operations (GoToUrl, GetUrl, SetRuntimeBrowser)
- Clipboard operations
- State checking (WaitState, IsEnabled)
- Data extraction (ExtractTableData)
- Form filling and popup handling
- JavaScript injection
- SAP-specific operations

### Options Pattern

Most operations accept an optional `*Options` parameter for advanced configuration. Use the `Options` static class for fluent builder creation:

```csharp
// Simple — uses defaults
app.Click("myButton");

// With options
app.Click("myButton", Options.Click(NClickType.Double, NMouseButton.Left).With(keyModifiers: NKeyModifiers.Ctrl));
```

All option types inherit from `TargetOptions` which provides common properties:
- `Timeout` (double, default 30s) — maximum time to find the element.
- `DelayAfter` (double, default 0.3s) — delay after the action.
- `DelayBefore` (double, default 0.2s) — delay before the action.

Use `.WithTimeouts(timeout, delayAfter, delayBefore)` on any option to override these.

Use `.WithContinueOnError(true)` on supported options to continue on error.

See [examples.md](examples.md) for full coded workflow examples.

See [portable-api.md](portable-api.md) when the project is specified to be cross-platform.

---

## Key Enum Reference Summary

| Enum | Values | Description |
|---|---|---|
| `NClickType` | `Single`, `Double`, `Down`, `Up` | Type of mouse click |
| `NMouseButton` | `Left`, `Right`, `Middle` | Mouse button to use |
| `NInteractionMode` | `HardwareEvents`, `Simulate`, `DebuggerApi`, `WindowMessages`, `Background` | How to interact with UI elements |
| `NChildInteractionMode` | `SameAsCard`, `HardwareEvents`, `Simulate`, `DebuggerApi`, `WindowMessages` | Interaction mode for child elements |
| `NAppOpenMode` | `Never`, `IfNotOpen`, `Always` | When to open/start the application |
| `NAppAttachMode` | `ByProcessName`, `ByInstance`, `SingleWindow` | How to find running app instance |
| `NAppCloseMode` | `Never`, `IfOpenedByAppBrowser`, `Always` | When to close app on dispose |
| `NWindowResize` | `None`, `Maximize`, `Restore`, `Minimize` | Window state on open |
| `NCheckType` | `Check`, `Uncheck`, `Toggle` | Checkbox action type |
| `NCheckStateMode` | `WaitAppear`, `WaitDisappear` | Wait condition for element state |
| `NKeyModifiers` | `None`, `Alt`, `Ctrl`, `Shift`, `Win` | Keyboard modifiers (flags, combinable) |
| `NClickMode` | `None`, `Single`, `Double` | Click before typing |
| `NEmptyFieldMode` | `None`, `SingleLine`, `MultiLine` | How to clear field before typing |
| `NTypeByClipboardMode` | `Never`, `Always`, `WhenPossible` | Whether to use clipboard for typing |
| `NScrollDirection` | `Up`, `Down`, `Left`, `Right` | Scroll direction |
| `CursorMotionType` | `Instant`, `Smooth` | Mouse movement style |
| `NBrowserType` | `None`, `IE`, `Firefox`, `Chrome`, `Edge`, `Custom`, `Safari` | Browser type for web automation |
| `NWebDriverMode` | `Disabled`, `WithGUI`, `Headless`, `DevTools`, `DevToolsHeadless` | WebDriver execution mode |
| `BrowserUserDataFolderMode` | `Automatic`, `DefaultFolder`, `CustomFolder` | Browser profile folder mode |
| `NVerifyMode` | `Appears`, `Disappears` | Element state to verify |
| `NUITaskAgentType` | `GeminiFlash25`, `GPT5`, `GPT5Mini`, `DOMBasedGPT41`, etc. | AI agent for UITask/ScreenPlay |
| `NDateSelectionType` | `Date`, `Range`, `Week` | Date picker selection mode |
| `LimitType` | `Rows`, `None`, `Page` | Data extraction limit type |

## Key Type Reference

| Type | Description |
|---|---|
| `IUiAutomationAppService` | Main service interface — Open/Attach apps, standalone operations |
| `UiTargetApp` | Application handle returned by Open/Attach — all element operations |
| `RuntimeTargetApp` | Async runtime application handle (internal to UiTargetApp) |
| `TargetAnchorableModel` | Target element model — accessed via `app["name"]` or `app[descriptor]` |
| `RuntimeTarget` | Runtime element reference returned by `GetChildren`/`GetRuntimeTarget` |
| `TargetAppModel` | Application model with Selector, FilePath, Url, BrowserType, Title, etc. |
| `UiElement` | Core UI element reference |
| `IScreenDescriptor` | Object Repository screen descriptor |
| `IElementDescriptor` | Object Repository element descriptor |
| `Options` | Static class with fluent builder methods for all option types |
| `UITaskResult` | Result from UITask — Status, Result, ErrorMessage |
| `SAPReadStatusbarResult` | SAP statusbar result — MessageType, MessageText, MessageId, MessageNumber |
