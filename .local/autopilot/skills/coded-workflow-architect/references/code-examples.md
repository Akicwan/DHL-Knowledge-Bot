# Code Examples

Generic templates for Coded Workflows, Coded Test Cases, and Coded Source Files. Replace `MyProject` with the actual project namespace.

> **Scope:** This file contains **structural templates** (how to set up files, arguments, InOut patterns, etc.). For **domain-specific examples** (Excel operations, UI automation flows, mail sending, etc.), read the `examples.md` in the corresponding domain folder (e.g. `excel/examples.md`, `ui-automation/examples.md`).

---

## Coded Workflow Examples

### Basic Workflow

```csharp
using System;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class BasicWorkflow : CodedWorkflow
    {
        [Workflow]
        public void Execute()
        {
            Log("Starting workflow...");

            // Your implementation here

            Log("Workflow completed.");
        }
    }
}
```

### Workflow with Arguments

```csharp
using System;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class ProcessDataWorkflow : CodedWorkflow
    {
        [Workflow]
        public (int processedCount, string status) Execute(string inputFile, bool validateData)
        {
            Log($"Processing file: {inputFile}");

            int count = 0;
            string status = "Success";

            try
            {
                if (validateData)
                {
                    // Validation logic
                }
                count = 10;
            }
            catch (Exception ex)
            {
                status = $"Error: {ex.Message}";
                Log(ex.Message, LogLevel.Error);
            }

            return (count, status);
        }
    }
}
```

### Using Excel API

```csharp
using System;
using System.Data;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class ExcelWorkflow : CodedWorkflow
    {
        [Workflow]
        public void Execute(string excelPath)
        {
            Log("Reading Excel file...");

            using (var workbook = excel.UseExcelFile(excelPath))
            {
                DataTable dataTable = workbook.Sheet["Sheet1"].ReadRange(true, true);
                Log($"Successfully read {dataTable.Rows.Count} rows from the Excel file.");

                foreach (DataRow row in dataTable.Rows)
                {
                    string rowContent = string.Join(" | ", row.ItemArray);
                    Log(rowContent);
                }
            }

            Log("Excel processing completed.");
        }
    }
}
```

### Using Mail API

```csharp
using System;
using System.Collections.Generic;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class MailWorkflow : CodedWorkflow
    {
        [Workflow]
        public void Execute(string recipient, string subject, string body)
        {
            var mailOptions = new SendOutlookMailOptions
            {
                To = new List<string> { recipient },
                Subject = subject,
                Body = body
            };
            mail.Outlook().SendMail(mailOptions);

            Log($"Email sent to {recipient}");
        }
    }
}
```

### Using UI Automation API

```csharp
using System;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class UIAutomationWorkflow : CodedWorkflow
    {
        [Workflow]
        public void Execute(string username, string password)
        {
            Log("Starting UI automation...");

            var app = uiAutomation.Open(Descriptors.MyApp.Application);

            app.TypeInto(Descriptors.MyApp.LoginScreen.Username, username);
            app.TypeInto(Descriptors.MyApp.LoginScreen.Password, password);
            app.Click(Descriptors.MyApp.LoginScreen.LoginButton);

            var welcomeText = app.GetText(Descriptors.MyApp.Dashboard.WelcomeMessage);
            Log($"Login successful: {welcomeText}");
        }
    }
}
```

### Async Workflow

```csharp
using System;
using System.Threading.Tasks;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class AsyncDataWorkflow : CodedWorkflow
    {
        [Workflow]
        public async Task Execute(string apiEndpoint)
        {
            Log("Starting async workflow...");

            var client = BuildClient();
            var response = await client.GetAsync(apiEndpoint);
            var content = await response.Content.ReadAsStringAsync();

            Log($"Received {content.Length} characters from API");
            await DelayAsync(1000);  // async delay

            Log("Async workflow completed.");
        }
    }
}
```

### Workflow with Single InOut Argument

A single input argument named `Output` with the same type as the return value becomes an InOut argument.

```csharp
using System;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class IncrementCounterWorkflow : CodedWorkflow
    {
        [Workflow]
        public int Execute(int Output)
        {
            Log($"Counter before: {Output}");
            Output++;
            Log($"Counter after: {Output}");
            return Output;
        }
    }
}
```

### Workflow with Multiple InOut Arguments

When multiple arguments are both input and output, the return type must be a tuple whose names and types match the input parameters exactly.

```csharp
using System;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class UpdateBatchStatusWorkflow : CodedWorkflow
    {
        [Workflow]
        public (int processedCount, bool isComplete) Execute(int processedCount, bool isComplete)
        {
            Log($"Current count: {processedCount}, complete: {isComplete}");

            processedCount += 10;
            isComplete = processedCount >= 100;

            Log($"Updated count: {processedCount}, complete: {isComplete}");
            return (processedCount, isComplete);
        }
    }
}
```

### Workflow with Default Parameters

Parameters with default values become optional when invoked via `workflows.MyWorkflow()`.

```csharp
using System;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class ConfigurableWorkflow : CodedWorkflow
    {
        [Workflow]
        public void Execute(string browser = "chrome.exe", int retryCount = 3, bool verbose = false)
        {
            Log($"Using browser: {browser}, retries: {retryCount}");

            if (verbose)
                Log("Verbose mode enabled — detailed logging active");

            // Implementation here
        }
    }
}

// Calling from another workflow:
// workflows.ConfigurableWorkflow();                                    // all defaults
// workflows.ConfigurableWorkflow(browser: "msedge.exe");              // override one
// workflows.ConfigurableWorkflow(browser: "msedge.exe", retryCount: 5, verbose: true);  // override all
```

### Invoking Other Workflows

```csharp
using System;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class OrchestratorWorkflow : CodedWorkflow
    {
        [Workflow]
        public void Execute(string[] items)
        {
            Log($"Processing {items.Length} items...");

            foreach (var item in items)
            {
                var (success, result) = workflows.ProcessSingleItem(item);

                if (!success)
                {
                    Log($"Failed to process: {item}", LogLevel.Warn);
                    continue;
                }

                Log($"Processed {item}: {result}");
            }

            workflows.Cleanup();
            Log("All items processed.");
        }
    }
}
```

---

## Coded Test Case Examples

### Basic Assertions

```csharp
using System;
using System.Collections.Generic;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class TestDataProcessing : CodedWorkflow
    {
        [TestCase]
        public void Execute()
        {
            // Arrange
            string inputFile = "TestData.xlsx";
            int expectedRows = 5;

            // Act
            var (processedCount, status) = workflows.ProcessData(inputFile, true);

            // Assert
            testing.VerifyAreEqual(expectedRows, processedCount, "Should process exactly 5 rows");
            testing.VerifyExpression(status == "Success", "Processing should succeed");
            Log("Test passed: Data processing verified.");
        }
    }
}
```

### Data-Driven with Parameters

```csharp
using System;
using UiPath.CodedWorkflows;

namespace MyProject
{
    public class TestInputValidation : CodedWorkflow
    {
        [TestCase]
        public void Execute(string inputValue = "INV-001", bool expectedValid = true)
        {
            Log($"Testing validation for: {inputValue}");

            var (isValid, message) = workflows.ValidateInput(inputValue);

            testing.VerifyAreEqual(expectedValid, isValid, $"Validation result for '{inputValue}': {message}");
        }
    }
}
```

### UI Automation with Before/After Hooks

```csharp
using System;
using UiPath.CodedWorkflows;
using UiPath.UIAutomationNext.API.Contracts;

namespace MyProject
{
    public class TestLoginFlow : CodedWorkflowBase  // Uses Before/After hooks
    {
        [TestCase]
        public void Execute()
        {
            // Before() already opened the app and navigated to login

            // Act
            var loginScreen = uiAutomation.Attach(Descriptors.MyApp.LoginScreen);
            loginScreen.TypeInto(Descriptors.MyApp.LoginScreen.Username, "testuser");
            loginScreen.TypeInto(Descriptors.MyApp.LoginScreen.Password, "testpass");
            loginScreen.Click(Descriptors.MyApp.LoginScreen.LoginButton);

            // Assert
            var dashboard = uiAutomation.Attach(Descriptors.MyApp.Dashboard);
            var welcomeText = dashboard.GetText(Descriptors.MyApp.Dashboard.WelcomeMessage);
            testing.VerifyContains(welcomeText, "testuser", "Welcome message should contain username");

            // After() will close the app and clean up
        }
    }
}
```

---

## Coded Source File Examples

### Helper/Utility Class

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

namespace MyProject
{
    // NO CodedWorkflow inheritance, NO [Workflow]/[TestCase] attribute, NO .cs.json
    public class InvoiceHelper
    {
        public static decimal CalculateTotal(List<decimal> amounts)
        {
            return amounts.Sum();
        }

        public static bool IsValidInvoiceNumber(string invoiceNumber)
        {
            return !string.IsNullOrEmpty(invoiceNumber) && invoiceNumber.StartsWith("INV-");
        }

        public static string FormatCurrency(decimal amount, string currencyCode = "USD")
        {
            return $"{currencyCode} {amount:N2}";
        }
    }
}
```

### Data Models (multiple types in one file)

```csharp
using System;

namespace MyProject
{
    public class InvoiceData
    {
        public string InvoiceNumber { get; set; }
        public decimal Amount { get; set; }
        public DateTime DueDate { get; set; }
        public InvoiceStatus Status { get; set; }
    }

    public enum InvoiceStatus
    {
        Pending,
        Approved,
        Rejected,
        Paid
    }
}
```

### Before/After Hooks Base Class

For the `CodedWorkflowBase` template implementing `IBeforeAfterRun` → see [codedworkflow-reference.md § Before/After Hooks](codedworkflow-reference.md#beforeafter-hooks-shared-setupteardown).
