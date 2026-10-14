# Tests: what they prove

Run the suite with the .NET 10 SDK:

```sh
dotnet test BraceletDesigner.slnx --configuration Release
```

CI runs on Linux and Windows. A failure uploads a TRX report. The number of passing cases is not a coverage or correctness score: a theory runs once for every input row.

For every remaining test's purpose and the removal/replacement map, see [Test inventory](TEST_INVENTORY.md). The suite was consolidated from 121 named tests / 207 cases to 91 / 160 without changing the application.

## Keep three kinds of checks

| Kind | Examples | What belongs here |
| --- | --- | --- |
| Calculations and file contracts | `GeometryTests`, `ElementContractTests`, `ShapeGeometryTests`, `PatternGeneratorTests`, `DocumentTests`, `ToolpathAndGCodeTests` | Known coordinates, shape-specific properties, independent copies, legacy JSON, complete machine command sequences. |
| Editor operations | `UndoRedoTests`, `GroupRepeatTests`, `FillInputTests`, `ExplodeTests` | Grouping, excluded copies, history isolation, repeat inputs, and operations on model state. |
| Real boundaries | `EditorWorkflowTests`, `WindowInteractionTests`, `HexagonTests`, `ProjectFileTests`, `ExportTests` | Linked editing operations, keyboard routing, control bindings, actual project files, actual exported files, cancellation, and rejected input. |

The same behavior should not be tested in three places just to increase the count. Keep a small calculation test to locate a failure and a real-boundary test where binding, file access, or routing can break the feature independently.

## Current protection

- **Guide-fill workflow:** draw two guides, fill squares and hexagons independently, select both in the real tree, repeat them together, fill across bracelet height, and undo/redo. Intermediate coordinates, both inner patterns, shared group bounds, and the exact restored arrangement are checked. Intentional overflow omits individual out-of-bounds shapes from export, not the entire repeated group.
- **Centering:** known group bounds and margins give independently calculated coordinates. Window tests send C followed by both screen axes and exercise the actual undo shortcut.
- **Explode:** shape-specific line counts and geometry are checked separately from the window's E / Ctrl+Z / Ctrl+Y workflow. The window tests also reject Ctrl+E and repeated E on already independent lines.
- **Copy/paste:** the window test actually pastes at a known cursor position; a handler returning `true` is not enough.
- **History:** multi-selection, continuous edits, bracelet-local undo/redo, and preservation of pan/zoom are checked. Shape copying also checks fresh IDs and independently mutable pattern lists.
- **Persistence:** actual Save/Open commands round-trip named shapes, repeat references, and two bracelets. Missing, malformed, unsupported, and invalid projects must leave the current design and history intact. Cancelled operations and validation failure must not overwrite an existing file.
- **Export:** complete expected output checks both machines, active-bracelet scope, selection scope, and selecting one repeat instance. Expected motion coordinates and feeds are fixed values, not generated with the production postprocessor. Rejected/cancelled exports preserve existing output.
- **Line endings:** generic output may use the host's line endings; tests normalize those only. U00 output must retain its required CRLF endings and is compared without normalization.

## Rules for changing tests

1. State a user-visible result or a risky calculation that the test protects. If two tests protect exactly the same boundary and result, merge them.
2. Prefer literal expected coordinates or a checked file example. Do not calculate the expected answer by calling the same function being tested.
3. Serializer round trips do not prove compatibility with old projects. Keep independent, literal legacy JSON examples.
4. Use `[AvaloniaFact]` / `[AvaloniaTheory]` for UI or view-model work, and `EditorTestFactory` for in-memory settings. Use plain xUnit attributes for pure calculations.
5. Use `TestDirectory` for disposable files. Do not read/write the developer's persisted defaults, reuse fixed paths, or leave test output behind.
6. Run deferred selection work with the dispatcher. Do not add sleeps to make a UI test pass.
7. When a real product bug is exposed, do not change the expected result to match the bug. Report it and agree on a production fix separately.

## Workflow consolidation

`EditorWorkflowTests.FillTwoGuidePatternsAcrossBraceletHeightThenUndoRedo` replaces three looser checks:

- `TwoExistingPatternsCanBeRepeatedAsOneUnit`: both filled patterns retain their inner copies when wrapped in a counted group.
- The `axis:width` row of `AxisFillMovesEveryMemberBySameOuterStep`: bracelet-height fill translates both patterns together.
- `ReferenceFilledPatternCanBeRepeatedWithAnotherObjectWithoutLosingItsCopies`: the original guide-filled placements survive grouping.

The new scenario additionally checks the real tree and reference dropdown, undo/redo via the window, and intentional clipping: 15 copies are displayed, 3 are omitted, and 58 line segments are exported. The user explicitly confirmed that overflow is intentional; the test does not impose an all-groups-must-fit rule.

Circumference fill remains a separate case. Mixed repeated/plain selections, diagonal guides, selecting a guide with its pattern, resizing, excluded copies, and nested-group duplication remain focused tests because this workflow does not cover those branches. Consolidating those blindly would lose protection.

The single consolidated workflow was also run against temporary application copies with four persistent faults: lost inner copies, shared placements between the two patterns, half-size union bounds, and exporting overflow instead of omitting it. It failed for each fault and passed after restoration. A transient wrong-bounds assignment was not detected because subsequent bound refresh corrected it; this scenario checks completed editing stages, not every transient rendering frame.

## Limits

These tests do not certify CNC hardware, machine calibration, crash-safe saving, or every pointer interaction. G-code expectations preserve the existing software contract; they are not an independently machine-approved reference program. The known production risks are recorded in [Repository review](REPOSITORY_REVIEW.md).

## Fault-detection evidence — 2026-10-14

A passing baseline was copied into a temporary directory. Each change below was applied alone, compiled, and tested; a failing assertion or test exception was required to count as detection. The original working-tree production files were fingerprinted and stayed unchanged. Restoring the copied source made the baseline pass again.

| Deliberately introduced fault | Detection boundary | Result |
| --- | --- | --- |
| Share histories between bracelets | Bracelet-local undo/redo | Detected |
| Reverse C → X / Y | Routed window keyboard input | Detected |
| Ignore export selection | Complete exported files for both machines | Detected |
| Share a copied shape's mutable pattern | Copy independence and editor operations | Detected |
| Reuse a copied shape's ID | Copy identity and editor operations | Detected |
| Use positive engraving depth | Toolpath commands and complete output | Detected |
| Replace the bracelet during undo | Viewport and history workflows | Detected |
| Save an empty project instead of the design | Actual Save/Open workflow | Detected |
| Discard the loaded project | Actual Save/Open workflow | Detected |
| Clear the current design after failed Open | Malformed/missing project cases | Detected |
| Lose Cartier's inner stroke width | Exploded component geometry | Detected |
| Distort hexagon side length | Known lengths and the real edge-length control | Detected |

This is a targeted sample, **not** a full mutation-testing score or a claim that every bug is caught. It is more informative than adding a test that merely repeats an implementation or returns `true`.
