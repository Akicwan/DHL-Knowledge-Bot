# Service Index

Single source of truth for all UiPath services, packages, and reference files.

---

## Core Packages (Almost Always Needed)

| Service | Package | Version | Main Ref | API Ref | Examples |
|---------|---------|---------|----------|---------|----------|
| `system` | `UiPath.System.Activities` | `[25.12.2]` | [system/system.md](system/system.md) | [system/windows-api.md](system/windows-api.md) | [system/examples.md](system/examples.md) |
| `testing` | `UiPath.Testing.Activities` | `[25.10.0]` | [testing/testing.md](testing/testing.md) | [testing/testing-verification.md](testing/testing-verification.md) | [testing/examples.md](testing/examples.md) |
| `uiAutomation` | `UiPath.UIAutomation.Activities` | `[25.10.21]` | [ui-automation/ui-automation.md](ui-automation/ui-automation.md) | [ui-automation/windows-api.md](ui-automation/windows-api.md) | [ui-automation/examples.md](ui-automation/examples.md) |
| `workflows` | *(implicit in v25.x)* | — | [codedworkflow-reference.md](codedworkflow-reference.md) | — | [code-examples.md](code-examples.md) |

---

## Office & Productivity Packages

| Service | Package | Version | Main Ref | API Ref | Examples |
|---------|---------|---------|----------|---------|----------|
| `excel` | `UiPath.Excel.Activities` | `[3.3.1]` | [excel/excel.md](excel/excel.md) | [excel/windows-api.md](excel/windows-api.md), [excel/portable-api.md](excel/portable-api.md) | [excel/examples.md](excel/examples.md) |
| `word` | `UiPath.Word.Activities` | `[2.3.1]` | [word/word.md](word/word.md) | [word/windows-api.md](word/windows-api.md), [word/portable-api.md](word/portable-api.md) | [word/examples.md](word/examples.md) |
| `powerpoint` | `UiPath.Presentations.Activities` | `[2.3.1]` | [powerpoint/powerpoint.md](powerpoint/powerpoint.md) | [powerpoint/windows-api.md](powerpoint/windows-api.md), [powerpoint/portable-api.md](powerpoint/portable-api.md) | [powerpoint/examples.md](powerpoint/examples.md) |
| `mail` | `UiPath.Mail.Activities` | `[2.5.10]` | [mail/mail.md](mail/mail.md) | [mail/windows-api.md](mail/windows-api.md), [mail/portable-api.md](mail/portable-api.md) | [mail/examples.md](mail/examples.md) |

---

## Cloud Integration Packages (Require Integration Service)

| Service | Package | Version | Main Ref | API Ref | Examples |
|---------|---------|---------|----------|---------|----------|
| `office365` | `UiPath.MicrosoftOffice365.Activities` | `[3.6.10]` | [office365/office365.md](office365/office365.md) | [office365/api.md](office365/api.md) | [office365/examples.md](office365/examples.md) |
| `google` | `UiPath.GSuite.Activities` | `[3.6.10]` | [gsuite/gsuite.md](gsuite/gsuite.md) | [gsuite/api.md](gsuite/api.md) | [gsuite/examples.md](gsuite/examples.md) |

> **Note:** `office365` and `google` require Integration Service connections configured in UiPath Automation Cloud. See [codedworkflow-reference.md § Integration Service Connections](codedworkflow-reference.md#integration-service-connections).

---

## IT Automation & Infrastructure Packages

| Service | Package | Version | Main Ref | API Ref | Examples |
|---------|---------|---------|----------|---------|----------|
| `azure` | `UiPath.Azure.Activities` | `[2.0.0]` | [it-automations/azure/azure.md](it-automations/azure/azure.md) | [it-automations/azure/api.md](it-automations/azure/api.md) | [it-automations/azure/examples.md](it-automations/azure/examples.md) |
| `gcp` | `UiPath.GoogleCloud.Activities` | `[1.0.0]` | [it-automations/google-cloud/google-cloud.md](it-automations/google-cloud/google-cloud.md) | [it-automations/google-cloud/api.md](it-automations/google-cloud/api.md) | [it-automations/google-cloud/examples.md](it-automations/google-cloud/examples.md) |
| `aws` | `UiPath.AmazonWebServices.Activities` | `[1.4.1]` | [it-automations/amazon-web-services/amazon-web-services.md](it-automations/amazon-web-services/amazon-web-services.md) | [it-automations/amazon-web-services/api.md](it-automations/amazon-web-services/api.md) | [it-automations/amazon-web-services/examples.md](it-automations/amazon-web-services/examples.md) |
| `awrks` | `UiPath.AmazonWorkSpaces.Activities` | `[1.4.1]` | [it-automations/amazon-workspaces/amazon-workspaces.md](it-automations/amazon-workspaces/amazon-workspaces.md) | [it-automations/amazon-workspaces/api.md](it-automations/amazon-workspaces/api.md) | [it-automations/amazon-workspaces/examples.md](it-automations/amazon-workspaces/examples.md) |
| `azureAD` | `UiPath.AzureActiveDirectory.Activities` | `[1.6.3]` | [it-automations/azure-active-directory/azure-active-directory.md](it-automations/azure-active-directory/azure-active-directory.md) | [it-automations/azure-active-directory/api.md](it-automations/azure-active-directory/api.md) | [it-automations/azure-active-directory/examples.md](it-automations/azure-active-directory/examples.md) |
| `azureWVD` | `UiPath.AzureWindowsVirtualDesktop.Activities` | `[1.4.1]` | [it-automations/azure-wvd/azure-wvd.md](it-automations/azure-wvd/azure-wvd.md) | [it-automations/azure-wvd/api.md](it-automations/azure-wvd/api.md) | [it-automations/azure-wvd/examples.md](it-automations/azure-wvd/examples.md) |
| `activeDirectoryDomainServices` | `UiPath.ActiveDirectoryDomainServices.Activities` | `[1.4.1]` | [it-automations/active-directory/active-directory.md](it-automations/active-directory/active-directory.md) | [it-automations/active-directory/api.md](it-automations/active-directory/api.md) | [it-automations/active-directory/examples.md](it-automations/active-directory/examples.md) |
| `citrix` | `UiPath.Citrix.Activities` | `[1.5.0]` | [it-automations/citrix/citrix.md](it-automations/citrix/citrix.md) | [it-automations/citrix/api.md](it-automations/citrix/api.md) | [it-automations/citrix/examples.md](it-automations/citrix/examples.md) |
| `hyperv` | `UiPath.HyperV.Activities` | `[1.4.0]` | [it-automations/hyperv/hyperv.md](it-automations/hyperv/hyperv.md) | [it-automations/hyperv/api.md](it-automations/hyperv/api.md) | [it-automations/hyperv/examples.md](it-automations/hyperv/examples.md) |
| `exchangeserver` | `UiPath.ExchangeServer.Activities` | `[1.0.0]` | [it-automations/exchange-server/exchange-server.md](it-automations/exchange-server/exchange-server.md) | [it-automations/exchange-server/api.md](it-automations/exchange-server/api.md) | [it-automations/exchange-server/examples.md](it-automations/exchange-server/examples.md) |
| `systemCenter` | `UiPath.SystemCenter.Activities` | `[1.0.0]` | [it-automations/system-center/system-center.md](it-automations/system-center/system-center.md) | [it-automations/system-center/api.md](it-automations/system-center/api.md) | [it-automations/system-center/examples.md](it-automations/system-center/examples.md) |
| `netiq` | `UiPath.NetIQeDirectory.Activities` | `[1.4.1]` | [it-automations/netiq-edirectory/netiq-edirectory.md](it-automations/netiq-edirectory/netiq-edirectory.md) | [it-automations/netiq-edirectory/api.md](it-automations/netiq-edirectory/api.md) | [it-automations/netiq-edirectory/examples.md](it-automations/netiq-edirectory/examples.md) |

---

## Implicit Services (No Package Required)

| Service | Description | Reference |
|---------|-------------|-----------|
| `workflows` | Invoke other workflows in the project | [codedworkflow-reference.md § Invoking Other Workflows](codedworkflow-reference.md#invoking-other-workflows) |
| `connections` | Integration Service connection management | [codedworkflow-reference.md § Integration Service Connections](codedworkflow-reference.md#integration-service-connections) |
| `Log()` | Logging (built into CodedWorkflow base) | [codedworkflow-reference.md § Built-in Methods](codedworkflow-reference.md#built-in-methods-available-in-any-workflowtest-case-via-this) |
| `Delay()` | Pause execution | [codedworkflow-reference.md § Built-in Methods](codedworkflow-reference.md#built-in-methods-available-in-any-workflowtest-case-via-this) |
| `BuildClient()` | Authenticated HttpClient for Orchestrator | [codedworkflow-reference.md § Built-in Methods](codedworkflow-reference.md#built-in-methods-available-in-any-workflowtest-case-via-this) |

---

## Adding a Package to project.json

To use a service, add its package to `project.json` → `dependencies`:

```json
{
  "dependencies": {
    "UiPath.Excel.Activities": "[3.3.1]",
    "UiPath.Testing.Activities": "[25.10.0]"
  }
}
```

Use `EditFileTool` to add dependencies. Use bracket notation `[version]` for exact version matching.
