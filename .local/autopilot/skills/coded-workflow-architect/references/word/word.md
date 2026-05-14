# Word Activities API Reference

Reference for the `word` service from `UiPath.Word.Activities` package.

**Required package:** `"UiPath.Word.Activities": "[2.3.1]"`

**Auto-imported namespaces:** `System`, `System.Collections.Generic`, `System.Data`, `UiPath.Word`, `UiPath.Word.Activities`, `UiPath.Word.Activities.API`, `UiPath.Word.Activities.API.Models`

**Service accessor:** `word` (type `IWordService`)

---

## Two API Layers

The Word API has two layers:

1. **Windows (Interop) API** — uses `word.UseWordDocument(...)` returning `IWordDocument`. Full-featured API that opens Word via COM interop. Supports hyperlinks, pictures, DataTables, bookmarks, clipboard paste, save-as, PDF export, sensitivity labels, and more. See [windows-api.md](windows-api.md).

2. **Portable API** — uses `word.UseDocument(...)` returning `IWordDocumentHandle`. Lightweight cross-platform API using Xceed. Supports basic append text, read text, and replace text. See [portable-api.md](portable-api.md).

**Always prefer the Windows API (`UseWordDocument`) unless cross-platform compatibility is required.**

For full coded workflow examples, see [examples.md](examples.md).

---

## Key Enum Reference

| Enum | Values | Description |
|---|---|---|
| `Position` | `Start`, `End`, `Before`, `After`, `Replace` | Insert position relative to target |
| `Occurrence` | `All`, `First`, `Last`, `Specific` | Which occurrence to target for replacement |
| `InsertRelativeType` | `Document`, `Bookmark`, `Text` | Reference point for insertion |
| `InsertHyperlinkRelativeToType` | `Document`, `Text` | Reference point for hyperlink insertion |
| `PasteRelativeToType` | `Document`, `Text` | Reference point for paste operation |
| `PasteOptionType` | `EmbedData`, `LinkData`, `Picture` | How to paste clipboard content |
| `WordSaveAsType` | `XmlDocument`, `MacroEnabledDocument`, `OldDocument`, `WebPage`, `FilteredWebPage`, `RichText`, `PlainText` | Save-as file format |
| `WordLabelOperation` | `None`, `Add`, `Clear` | Sensitivity label operation |

## Key Types

| Type | Description |
|---|---|
| `IWordDocument` | Disposable handle for Windows interop documents. Returned by `UseWordDocument`. Use with `using`. |
| `IWordDocumentHandle` | Disposable handle for portable (Xceed) documents. Returned by `UseDocument`. Use with `using`. |
| `WordUseOptions` | Options for opening/creating Word documents (Windows API). See [windows-api.md](windows-api.md). |
| `IWordLabelObject` | Sensitivity label with `LabelId` (`string`) and `Justification` (`string`) properties. |
