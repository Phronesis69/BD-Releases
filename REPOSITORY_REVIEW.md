# Repository review — 2026-10-14

**What this repo does:** A single-user desktop editor builds bracelet designs, repeats their shapes, saves JSON projects, and exports CNC instructions. The expected load is one editor with ordinary designs, not a shared server or untrusted public upload service.

## Fixed in this pass

- Tests now use explicit in-memory workspace settings instead of the user's cached dimensions. Repeated dialog/default-store fixtures were consolidated in `BraceletDesigner.Tests/EditorTestFactory.cs`.
- UI and view-model tests use Avalonia's headless XUnit runtime. Real-window tests exercise routed keyboard events and the bound tree selection, including undo/redo.
- `Shape.Copy()` and `RepeatPattern.Copy()` replace duplicated per-shape copying. Copies retain custom names and wavy-road wave counts, have fresh IDs, and do not share mutable pattern lists.
- Undo/redo restores all selected shape IDs and the active reference-line selection instead of retaining only the primary object.
- Edit history is now bracelet-local. Undo restores the current bracelet in place without replacing other bracelets or resetting canvas pan/zoom. Same-value repeat-reference binding feedback no longer creates spurious history entries. Creating/deleting a bracelet is outside the per-bracelet edit history.
- Legacy circle migration retains custom names.
- Regression tests for names, wave counts, and multi-selection were observed failing before their fixes. The original green suite did not cover these losses.
- CI now runs Release tests on Linux and Windows, and retains failure reports separately from release packages.

## Must fix next

1. **Saving can destroy the last good project** (`BraceletDesigner.App/MainWindowViewModel.cs`, `WriteProjectAsync`).
   - **What this is:** Save writes directly to an existing project file.
   - **Problem:** `File.WriteAllTextAsync` truncates the file before writing. A full disk, process termination, or write failure can leave a partial file; the error message does not restore it.
   - **Fix:** Stage the complete output in the destination directory, then replace the destination. Only update `_currentPath` after a successful save. Test failed replacement and preservation of the previous project.
   - **If we skip it:** A failed save can lose the previous design.

2. **Invalid numeric values pass validation** (`BraceletDesigner.Core/DocumentServices.cs`, `DocumentValidator.Validate`).
   - **What this is:** Validation protects project saving and CNC export.
   - **Problem:** Comparisons such as `value <= 0` do not reject `double.NaN`. For example, a workspace width of NaN passes those checks and subsequently fails JSON history serialization. Coordinates and angles also lack finite-number checks.
   - **Fix:** Check `double.IsFinite` for workspace, shape, and repeat geometry at input boundaries, before history serialization or toolpath generation. Add tests for NaN and infinity without changing machine calibration values.
   - **If we skip it:** Invalid geometry can crash editing or reach later export calculations.

3. **Reference lines have inconsistent ownership** (`BraceletDesigner.App/MainWindowViewModel.cs`, `PasteSelectionAt`, `DeleteSelectedReferenceLine`).
   - **What this is:** A reference line is held both in `Shapes` and in `ReferenceLines`, and selection is stored separately.
   - **Problem:** Paste adds a copied reference line to `Shapes` but not `ReferenceLines`; it cannot appear as a repeat reference. The dedicated reference-delete command removes the line but leaves `_selectedShape` and `_selectedShapes` pointing to it.
   - **Fix:** Share shape insertion/removal logic that keeps guides, repeat references, and selection consistent. Test copying/pasting and deleting guides through the actual window.
   - **If we skip it:** Pasted guides cannot drive repeats, and deleted guides can remain editable as a ghost selection.

## Should fix

4. **Repeat counts can allocate unbounded geometry** (`BraceletDesigner.Core/PatternGenerator.cs`, `GeneratePlacements`, `GenerateAlongReference`).
   - **What this is:** Every repeated copy is eagerly expanded into line segments.
   - **Problem:** A project with a count near `int.MaxValue` passes the current count validation and attempts a huge list allocation. This can exhaust memory while opening or editing the project.
   - **Fix:** Define a documented, product-appropriate geometry budget and reject excessive projects before generating their instances. Test the boundary; do not silently truncate CNC output.
   - **If we skip it:** An accidental large count or externally edited project can make the editor unusable.

**Verdict:** Test isolation and shape-copy correctness are substantially better. Save safety, numeric validation, and guide ownership are the next focused changes; do not treat a passing test count as proof of hardware safety.

**Not checked:** Real CNC hardware, manual visual layout, Windows execution locally, release deployment, exhaustive pointer interactions, malformed-project fuzzing, and large-design performance. The canvas's full interaction state machine was not audited line by line. No media, existing features, or machine feed/depth calibration was removed.
