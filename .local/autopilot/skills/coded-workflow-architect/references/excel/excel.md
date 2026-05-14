# Excel Activities API Reference

Reference for the `excel` service from `UiPath.Excel.Activities` package.

**Required package:** `"UiPath.Excel.Activities": "[3.3.1]"`

**Auto-imported namespaces:** `System`, `System.Collections.Generic`, `System.Data`, `UiPath.Excel`, `UiPath.Excel.Activities`, `UiPath.Excel.Activities.API`, `UiPath.Excel.Activities.API.Models`

**Service accessor:** `excel` (type `IExcelService`)

---

## Two API Layers

The Excel API has two layers:

1. **Windows (Interop) API** — uses `excel.UseExcelFile(...)` returning `IWorkbookQuickHandle`. This is the modern, full-featured API that opens Excel via COM interop. Supports sheets, ranges, cells, tables, charts, pivot tables, filtering, sorting, macros, etc. See [windows-api.md](windows-api.md).

2. **Portable API** — uses `excel.UseWorkBook(...)` returning `IWorkHandle`. This is a lightweight API that works cross-platform using a non-interop engine. Supports basic read/write/append operations and CSV operations. See [portable-api.md](portable-api.md).

**Always prefer the Windows API (`UseExcelFile`) unless cross-platform compatibility is required.**

For full coded workflow examples, see [examples.md](examples.md).

---

## Key Enum Reference Summary

| Enum | Values | Description |
|---|---|---|
| `ReadFormattingOptions` | `Default`, `RawValue`, `DisplayValue` | How to read cell values |
| `ResizeWindowOptions` | `None`, `Minimize`, `Maximize` | Excel window state on open |
| `EmptyRowBehavior` | `Stop`, `StopAfterThreeConsecutiveEmptyRows`, `Skip`, `Process` | How to handle empty rows when reading |
| `CopyPasteRangeOptions` | `All`, `Values`, `Formulas`, `Formats` | What to include in copy/paste operations |
| `FindReplaceOptions` | `Find`, `Replace`, `ReplaceAll` | Find/replace operation mode |
| `LookInOptions` | `Values`, `Formulas` | Where to search in find operations |
| `InsertRowPosition` | `Start`, `End`, `Specific` | Where to insert new rows |
| `DeleteRowsOption` | `Specific`, `Visible`, `Hidden`, `Duplicates` | Which rows to delete |
| `ColumnRelativePosition` | `Before`, `After` | Column insert position |
| `ColumnsCompare` | `IndividualColumns`, `AllColumns` | How to compare columns for duplicates |
| `MatchType` | `LargestValueLessOrEqual`, `ExactlyEqual`, `SmallestValueGreaterOrEqual` | VLOOKUP match type |
| `LastRowConfiguration` | `LastPopulatedRow`, `FirstEmptyRow` | How to determine last row |
| `LogicalOperator` | `And`, `Or` | Logical operator for filters |
| `PdfSaveQuality` | `StandardQuality`, `MinimumQuality` | PDF export quality |
| `ExcelSaveAsType` | `OpenXmlWorkbook`, `BinaryWorkbook`, `MacroEnabledWorkbook`, `OldWorkbook` | Save-as file format |
| `ExcelLabelOperation` | `None`, `Add`, `Clear` | Sensitivity label operation |
| `DelimitatorOptions` | `Comma`, `Semicolon`, `Pipe`, `Caret`, `Tab` | CSV delimiter |
| `PivotTableLayoutRowType` | `Compact`, `Tabular`, `Outline` | Pivot table row layout |
| `ExcelChartAction` | `CopyToClipboard`, `SaveAsPicture` | Chart export action |
