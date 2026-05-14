# CodedWorkflow Base Class Reference

All workflow and test case files inherit from `CodedWorkflow`, which provides built-in methods and service access. The `CodedWorkflow` class is a **partial class** — you can extend it in a Coded Source File (see "Before/After Hooks" below).

> **Note:** All examples use `MyProject` as the placeholder namespace. Replace with the actual project namespace (sanitized project name from `project.json`).

## Three Types of .cs Files

| Type | Base Class | Attribute | Purpose |
|------|-----------|-----------|---------|
| **Coded Workflow** | `CodedWorkflow` | `[Workflow]` | Executable automation logic |
| **Coded Test Case** | `CodedWorkflow` | `[TestCase]` | Automated test with assertions |
| **Coded Source File** | None (plain C#) | None | Reusable models, helpers, utilities, hooks |

## Namespace Rules

The namespace for all `.cs` files MUST match the **sanitized project name** from `project.json` `"name"` field.

**Sanitization rules:**
1. Replace spaces and hyphens with `_`
2. Remove any characters invalid in C# identifiers (keep letters, digits, `_`)
3. If the name starts with a digit, prepend `_`
4. For files in subfolders, append the folder name: `ProjectName.FolderName`

| Project Name (in `project.json`) | Namespace |
|---|---|
| `MyProject` | `MyProject` |
| `My Invoice-App 2` | `My_Invoice_App_2` |
| `RoboticEnterpriseFramework` | `RoboticEnterpriseFramework` |
| `3D-Automation` | `_3D_Automation` |
| `My Project` (file in `Helpers/` subfolder) | `My_Project.Helpers` |

## File Format: Coded Workflow

```csharp
using System;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class MyWorkflow : CodedWorkflow
    {
        [Workflow]
        public void Execute()
        {
            // Workflow implementation
        }
    }
}
```

## File Format: Coded Test Case

```csharp
using System;
using System.Collections.Generic;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class TestMyFeature : CodedWorkflow
    {
        [TestCase]
        public void Execute()
        {
            // Arrange (Given)
            var expected = "success";

            // Act (When)
            var actual = workflows.ProcessSomething();

            // Assert (Then)
            testing.VerifyAreEqual(expected, actual, "Should return success");
        }
    }
}
```

### Testing Assertions

Available via the `testing` service (requires `UiPath.Testing.Activities` in `project.json`). See [testing/testing.md](testing/testing.md) for full API.

| Method | Purpose |
|--------|---------|
| `VerifyExpression(bool condition, string message)` | Assert boolean condition is true |
| `VerifyAreEqual<T>(T expected, T actual, string message)` | Assert two values are equal |
| `VerifyAreNotEqual<T>(T notExpected, T actual, string message)` | Assert two values are NOT equal |
| `VerifyContains(string fullString, string substring, string message)` | Assert string contains substring |
| `VerifyIsTrue(bool condition, string message)` | Alias for VerifyExpression |
| `VerifyRange(double value, double min, double max, string message)` | Assert value is within range |

### Data-Driven Test Cases

Add default parameter values to `Execute` for parameterized tests:

```csharp
[TestCase]
public void Execute(string inputValue = "INV-001", bool expectedValid = true)
{
    var (isValid, message) = workflows.ValidateInput(inputValue);
    testing.VerifyAreEqual(expectedValid, isValid, $"Validation for '{inputValue}': {message}");
}
```

## File Format: Coded Source File

Plain C# classes — NO `CodedWorkflow` inheritance, NO `[Workflow]`/`[TestCase]` attribute, NO `.cs.json` companion file. May group related types in one file.

```csharp
using System;
using System.Collections.Generic;

namespace MyProject
{
    public class MyHelper
    {
        public static bool IsValid(string input) => !string.IsNullOrEmpty(input);
    }

    public class MyModel
    {
        public string Name { get; set; }
        public decimal Amount { get; set; }
    }

    public enum MyStatus { Pending, Approved, Rejected }
}
```

**IMPORTANT:** Coded Source Files do NOT have access to `CodedWorkflow` services (`excel`, `mail`, `system`, `uiAutomation`, etc.). If you need service access, use a Coded Workflow instead.

## Built-in Methods (available in any workflow/test case via `this`)

| Method | Description |
|--------|-------------|
| `Log(string message, LogLevel level = LogLevel.Info, IDictionary<string, object> additionalLogFields = null)` | Output log messages with optional level and custom fields |
| `Delay(TimeSpan time)` / `Delay(int delayMs)` | Pause execution synchronously |
| `DelayAsync(TimeSpan time)` / `DelayAsync(int delayMs)` | Pause execution asynchronously |
| `BuildClient(string scope = "Orchestrator", bool force = true)` | Build an authenticated `HttpClient` for Orchestrator or custom scopes |
| `GetRunningJobInformation()` | Get info about the current running job (status, progress, parameters, timestamps) |
| `RunWorkflow(string workflowFilePath, IDictionary<string, object> inputArguments = null, TimeSpan? timeout = null, bool isolated = false, InvokeTargetSession targetSession = InvokeTargetSession.Current)` | **Fallback method:** Invoke workflow by string path. Use `workflows.MyWorkflow()` instead when possible |
| `RunWorkflowAsync(...)` | Async version of `RunWorkflow` (same limitations apply) |

### Logging Best Practices

```csharp
Log("Starting workflow execution...");
Log($"Processing {items.Count} items");
Log("Workflow completed successfully", LogLevel.Info);
Log("Warning: Retry limit approaching", LogLevel.Warn);
Log("Error occurred: " + ex.Message, LogLevel.Error);
```

## Invoking Other Workflows

**Recommended:** Use the strongly-typed `workflows` property:

```csharp
var result = workflows.ProcessInvoice(invoiceId: "INV-001", amount: 1500.00m);
Log($"Processing completed: {result.success}");
```

**Benefits of `workflows.MyWorkflow()`:**
- **Type-safe:** Compile-time checking of workflow names and parameters
- **IntelliSense:** Auto-completion for workflow names and parameters
- **Refactor-friendly:** Renaming workflows/parameters updates all references

**Default parameters:** Workflows with default parameter values can be invoked with or without those arguments:
```csharp
// If ProcessData has: Execute(string source, int maxRows = 100, bool verbose = false)
workflows.ProcessData(source: "invoices.csv");                          // maxRows=100, verbose=false
workflows.ProcessData(source: "invoices.csv", maxRows: 500);           // verbose=false
workflows.ProcessData(source: "invoices.csv", maxRows: 500, verbose: true);  // all explicit
```

**Fallback (string-based):** For dynamic scenarios where workflow name isn't known at compile time:

```csharp
string workflowPath = GetWorkflowPathFromConfig();
var result = RunWorkflow(workflowPath, new Dictionary<string, object>
{
    { "invoiceId", "INV-001" },
    { "amount", 1500.00m }
});
```

## Arguments (In/Out/InOut Patterns)

**Input Arguments:** Method parameters

```csharp
[Workflow]
public void Execute(string inputPath, int maxRetries)
{
    // inputPath and maxRetries are input arguments
}
```

**Output Arguments:** Return values (use tuples for multiple outputs)

```csharp
[Workflow]
public (bool success, string message) Execute(string input)
{
    return (true, "Completed successfully");
}
```

**InOut Arguments:** Syntax differs for single vs multiple InOut arguments

```csharp
// Single in argument named Output becomes an InOut argument
[Workflow]
public int Execute(int Output)
{
    Output++;
    return Output;
}

// Multiple InOut arguments — return type is a tuple with matching names and types
[Workflow]
public (int count, bool isDone) Execute(int count, bool isDone)
{
    count++;
    isDone = false;
    return (count, isDone);
}
```

## Available APIs (Service Properties)

Access these through inherited service properties. Each requires its corresponding NuGet package in `project.json`. See [SERVICE_INDEX.md](SERVICE_INDEX.md) for the full service-to-package mapping.

| API | Property | Common Operations | Reference |
|-----|----------|-------------------|-----------|
| **System** | `system` | `GetAsset`, `AddQueueItem`, `GetCredential`, `WriteTextFile`, `ReadTextFile` | [system/system.md](system/system.md) |
| **Excel** | `excel` | `UseExcelFile`, `UseWorkBook` | [excel/excel.md](excel/excel.md) |
| **Word** | `word` | `UseWordFile` | [word/word.md](word/word.md) |
| **PowerPoint** | `powerpoint` | `UsePresentationFile` | [powerpoint/powerpoint.md](powerpoint/powerpoint.md) |
| **Mail** | `mail` | `SendMail`, `ReadMail`, `GetMailFolders`, `SaveAttachments` | [mail/mail.md](mail/mail.md) |
| **Office 365** | `office365` | `Mail`, `Calendar`, `Excel`, `OneDrive`, `SharePoint` | [office365/office365.md](office365/office365.md) |
| **Google Workspace** | `google` | `Gmail`, `Calendar`, `Drive`, `Sheets`, `Docs` | [gsuite/gsuite.md](gsuite/gsuite.md) |
| **Testing** | `testing` | `VerifyExpression`, `VerifyAreEqual`, `VerifyContains`, `VerifyRange` | [testing/testing.md](testing/testing.md) |
| **UI Automation** | `uiAutomation` | `Click`, `TypeInto`, `GetText`, `ElementExists`, `Open` | [ui-automation/ui-automation.md](ui-automation/ui-automation.md) |

## Object Repository Usage

Access UI elements defined in the Object Repository via the `Descriptors` static class:

```csharp
// Pattern: Descriptors.AppName.ScreenName.ElementName
var emailField = Descriptors.UiPath_Banking_App.Form.Email;
var submitButton = Descriptors.MyApp.LoginScreen.SubmitButton;

// Use with UI Automation
uiAutomation.TypeInto(emailField, "user@example.com");
uiAutomation.Click(submitButton);
```

## Integration Service Connections

When packages that use Integration Service connections are installed (e.g. `UiPath.MicrosoftOffice365.Activities`, `UiPath.GSuite.Activities`), Studio auto-generates two files in `.codedworkflows/`:

- **`ConnectionsManager.cs`** — Exposes a typed property for each connection category (e.g. `O365Mail`, `Excel`, `OneDrive`, `Gmail`, etc.)
- **`ConnectionsFactory.cs`** — Contains factory classes with typed properties for each configured connection instance

These are injected via the `connections` property on `CodedWorkflow`.

### Usage Pattern

```csharp
// Step 1: Get the connection from the auto-generated factory
var mailConnection = connections.O365Mail.My_Workspace_user_company_com;

// Step 2: Get a sub-service from the connection-based service
var mailService = office365.Mail(mailConnection);

// Step 3: Call methods on the sub-service
mailService.SendEmail("recipient@example.com", "Subject", "Body");
```

### Connection Types by Package

| Package | Connection Class | Factory Name | Used By |
|---------|-----------------|--------------|---------|
| `UiPath.MicrosoftOffice365.Activities` | `MailConnection` | `O365Mail` | `office365.Mail()`, `office365.Calendar()` |
| `UiPath.MicrosoftOffice365.Activities` | `ExcelConnection` | `Excel` | `office365.Excel()` |
| `UiPath.MicrosoftOffice365.Activities` | `OneDriveConnection` | `OneDrive` | `office365.OneDrive()`, `office365.Sharepoint()` |
| `UiPath.GSuite.Activities` | `GmailConnection` | `Gmail` | `google.Gmail()`, `google.Calendar()` |
| `UiPath.GSuite.Activities` | `DriveConnection` | `GoogleDrive` | `google.Drive()` |
| `UiPath.GSuite.Activities` | `SheetsConnection` | `GoogleSheets` | `google.Sheets()` |
| `UiPath.GSuite.Activities` | `DocsConnection` | `GoogleDocs` | `google.Docs()` |

### Important Notes

- Connection names in the factory are sanitized versions of the Integration Service display name (spaces/special chars replaced with `_`)
- The connection ID (GUID) is embedded in the factory — it references the specific Integration Service connection
- If a connection is **not authorized** or the token is expired, you get `ConnectionHttpException: Connection [...] failed to authorize` at runtime — re-authorize in Automation Cloud → Integration Service
- The `connections` property is always available on `CodedWorkflow` regardless of installed packages, but the factory properties (`.O365Mail`, `.OneDrive`, etc.) only exist when the corresponding package is installed and connections are configured

## Before/After Hooks (Shared Setup/Teardown)

To add shared setup/teardown logic that runs before and after ALL test cases, create a `CodedWorkflowBase` class (a Coded Source File) implementing `IBeforeAfterRun`:

```csharp
// CodedWorkflowBase.cs — Coded Source File (NOT a workflow, NO .cs.json)
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class CodedWorkflowBase : CodedWorkflow, IBeforeAfterRun
    {
        public void Before(BeforeRunContext context)
        {
            Log($"[BEFORE] Starting {context.RelativeFilePath}");
            // Setup: open app, log in, navigate to starting state
        }

        public void After(AfterRunContext context)
        {
            Log($"[AFTER] Finished {context.RelativeFilePath}");
            // Teardown: close app, clean up test data, log out
        }
    }
}
```

Then inherit from `CodedWorkflowBase` instead of `CodedWorkflow`:

```csharp
public class TestInvoiceCreation : CodedWorkflowBase
{
    [TestCase]
    public void Execute()
    {
        // Before() runs automatically before this method
        // ... test logic ...
        // After() runs automatically after this method (even on failure)
    }
}
```

**Key points:**
- `CodedWorkflowBase.cs` is a Coded Source File — no `.cs.json`, not an entry point
- `Before()` runs before EVERY workflow/test case that inherits from `CodedWorkflowBase`
- `After()` runs after EVERY workflow/test case, even if it throws an exception
- Perfect for test suites needing shared app setup/teardown, login/logout, test data management

## The `services` Property

The `services` property provides access to:
- `services.Container` — dependency injection container for resolving custom services
- `OrchestratorClientService` (via `BuildClient`) — Orchestrator API interaction
- `WorkflowInvocationService` (via `RunWorkflow`) — fallback for dynamic workflow invocation
- `OutputLoggerService` (via `Log`) — logging
