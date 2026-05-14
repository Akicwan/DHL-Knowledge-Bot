# Testing Activities API Reference

Reference for the `testing` service from `UiPath.Testing.Activities` package.

**Required package:** `"UiPath.Testing.Activities": "[25.10.0]"`

**Auto-imported namespaces:** `System`, `System.Collections.Generic`, `UiPath.Testing.Activities.TestDataQueues.Enums`, `UiPath.Testing.Activities.TestData`, `UiPath.Testing.Enums`, `UiPath.Testing`, `UiPath.Testing.Activities.Models`, `UiPath.Testing.Activities.Api.Models`

**Service accessor:** `testing` (type `ITestingService`)

---

## API Categories

The Testing API has three main areas:

1. **Verification / Assertions** — Methods for verifying expressions, equality, comparisons, ranges, containment, and regex matching. See [testing-verification.md](testing-verification.md).

2. **Data Generation & Test Data Queues** — Random data generation (names, numbers, dates, strings, addresses) and Orchestrator test data queue operations (add, get, delete items). See [testing-data.md](testing-data.md).

3. **Document & Text Comparison** — PDF document and text comparison with rules, semantic analysis, and diff output. See [testing-comparison.md](testing-comparison.md).

For full coded workflow examples, see [examples.md](examples.md).

---

## Key Type Reference

### Enums

| Enum | Values | Description |
|---|---|---|
| `Comparison` | `Equality`, `Inequality`, `GreaterThan`, `GreaterThanOrEqual`, `LessThan`, `LessThanOrEqual`, `Contains`, `RegexMatch` | Comparison operator for verification methods |
| `VerificationType` | `IsWithin`, `IsNotWithin` | Range verification type |
| `ComparisonType` | `Line`, `Word`, `Character` | Granularity for text/document comparison |
| `Case` | `LowerCase`, `UpperCase`, `CamelCase`, `Mixed` | String case for random data generation |
| `TestDataQueueItemStatus` | `All`, `OnlyConsumed`, `OnlyNotConsumed` | Filter for test data queue items |
| `Operation` | `Inserted`, `Deleted`, `Equal` | Diff operation type in comparison results |
| `DocumentOutputDiffType` | `None`, `Pdf`, `Unidiff`, `Html` | Output format for document comparison diffs |

### Classes

| Class | Description |
|---|---|
| `TestDataQueueItem` | Test data queue item with `Id`, `Content` (Dictionary), `IsConsumed` |
| `ComparisonResult` | Comparison result with `AreEquivalent`, `Differences`, `SemanticDifferences` |
| `Difference` | Single difference with `Operation` and `Text` |
| `SemanticDifferences` | Semantic analysis with `AreSemanticallyEquivalent`, `Explanation`, `Differences` |
| `SemanticDifference` | Single semantic difference with `Explanation` |
| `RegexRule` | Comparison rule using regex. Constructor: `RegexRule(name, pattern, usePlaceholder)` |
| `WildcardRule` | Comparison rule using wildcards (`*`, `?`). Constructor: `WildcardRule(name, pattern, usePlaceholder)` |
