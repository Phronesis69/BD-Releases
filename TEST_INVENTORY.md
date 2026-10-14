# Test inventory

The suite has **91 named tests / 160 input cases**, down from **121 / 207**. A named theory has several cases when different shapes, directions, machines, or invalid inputs take different paths. Those rows are kept visible rather than hidden in a giant loop.

Start with the workflow sections. The calculation and safety sections are the smaller checks that explain a workflow failure. This inventory identifies every remaining named test; helpers and fixture factories are not tests.

All files below are under `BraceletDesigner.Tests/`. Run a section using, for example:

```sh
dotnet test BraceletDesigner.slnx --filter FullyQualifiedName~FillInputTests
```

## Design workflows

### `EditorWorkflowTests.cs`
| Test | Protects |
| --- | --- |
| `FillTwoGuidePatternsAcrossBraceletHeightThenUndoRedo` | Two guide fills, tree multi-selection, counted grouping, height fill, exact undo/redo, and intentional overflow/export omission. |

### `FillInputTests.cs`
| Test | Protects |
| --- | --- |
| `FillInputsStayInSyncThroughDriverSwitchResizeAndGuideEdit` | One editing session changes gap/count drivers, rejects disabled-field feedback, resizes the shape, edits its guide, and returns to normal Repeat. |
| `DriverRoundTripsAndSurvivesDuplicateUndoAndRedo` | Fill-driver state survives saving, duplication, and undo/redo without altering the original. |
| `SelectingCrossAndReferencePreservesFillAndRejectsRepeatBindingFeedback` | Selecting a filled cross with its guide must not reset its fill. Mouse/list selection and selection order are separate regression inputs. |
| `CountIsClampedToAvoidNegativeGapsAndSingleItemIsFinite` | Excessive count is clamped; one item has no divide-by-zero or negative gap. |
| `CountDrivenDiagonalWavesUseCalculatedGapAndPreserveExclusions` | Diagonal, reversed wave fill keeps excluded instance numbers. |
| `WorkspaceAxisFillSupportsBothDrivers` | Count and gap drivers produce consistent layouts on both workspace axes. |

### `GroupRepeatTests.cs`
| Test | Protects |
| --- | --- |
| `RepeatingMixedSelectionPreservesInnerCopiesAndUsesFullUnitBounds` | A repeated shape plus a plain shape can enter Count or Fill; changing/removing the outer mode preserves the original unit. |
| `CircumferenceFillMovesEveryMemberBySameOuterStep` | Circumference fill translates all group members together; this is not the height-fill branch. |
| `ReferenceFillPreservesUnitAndUsesReferenceLengthForOuterCenters` | A grouped unit can fill a diagonal external guide, not just a bracelet axis. |
| `ReferenceFilledCrossAndItsGuideCanBeRepeatedTogether` | A guide selected as a group member moves with its pattern but never exports. |
| `ResizingMemberUpdatesOuterUnitBoundsAndSpacing` | Editing one member refreshes the shared group bounds and spacing. |
| `GapDrivenGroupFillUsesGroupSizeAndKeepsInnerOffsets` | Group fill driven by a minimum gap uses the complete unit's size. |
| `MirroringGroupUnitPreservesAndReflectsItsInnerCopies` | Both mirror directions transform the stored inner placements correctly. |
| `GroupsCanBeCombinedIntoAnotherRepeatUnit` | A previously grouped pattern can be grouped again without losing its existing copies. |
| `GroupUnitsSurviveSerializationUndoRedoAndDuplication` | Nested placement data survives load/history/copy; duplicated groups have fresh group IDs and translated bounds. |
| `ExcludedInnerCopiesStayExcludedAndOuterInstancesCanBeDetached` | Hidden inner copies remain hidden when grouping; detaching a flattened instance retains the other copies. |

## Editing, history, and actual window controls

### `UndoRedoTests.cs`
| Test | Protects |
| --- | --- |
| `AddResizeDuplicateAndDragLineUndoRedoInOrder` | A line is created, moved, resized with a fixed bottom endpoint, rotated, configured, duplicated, and dragged. Every undo/redo restores the exact preceding document; one drag is one history step. |
| `IncreasingWavyRoadHeightKeepsItsBottomEdgeFixed` | Wave-height editing uses the same fixed-edge rule as line length, but a different shape setter. |
| `ToolpathExcludesOverflowingRepeatInstances` | Ordinary counted overflow remains visible in the design but is omitted from evaluated export paths. |
| `UndoAndRedoPreserveTheWholeSelection` | All selected IDs, their relative placement, and the primary selection survive history. |
| `EachBraceletHasIndependentUndoAndRedoWithoutReplacingOtherBracelets` | Editing one bracelet cannot undo or clear another bracelet's history; object identity remains stable. |
| `SwitchingBraceletsCompletesOnlyTheOriginalContinuousEdit` | A gesture in progress is committed to its original bracelet before switching. |
| `UndoRestoresWorkspaceAndGuidesOnlyInTheActiveBracelet` | Workspace and guide edits undo independently; guide objects remain shared with the shape list. |
| `DeletingAnotherBraceletDoesNotEnterOrClearTheRemainingHistory` | Deleting a bracelet does not resurrect it through another bracelet's edit undo. |
| `ScaledBraceletStartsWithItsOwnEmptyHistory` | A scaled copy starts a new history and does not consume the source bracelet's history. |

### `WindowInteractionTests.cs`
| Test | Protects |
| --- | --- |
| `SelectRepeatInstanceDetachMoveAndUndoRedoThroughWindow` | A real tree child is selected, L detaches it, moving it leaves the parent alone, and keyboard history restores both steps. |
| `CenterShortcutUsesTheScreenAxisAndMargins` | Actual C → X/Y routing uses screen axes and margins; keyboard undo restores position. |
| `ExplodeShortcutProducesIndependentLinesAndOneUndoStep` | E works through the window for the supported composites; Ctrl+E and repeated E do not explode again, and keyboard undo/redo restores IDs. |
| `EscapeCancelsCenteringAndControlCAndVActuallyCopyAndPaste` | Escape cancels a pending command; copy/paste produces a new object at the cursor, not merely a successful handler return. |
| `PanAndZoomNeverEnterHistoryOrResetOnUndoAndRedo` | Real mouse navigation stays outside history and remains unchanged during edits' undo/redo. |
| `TreeMultiSelectionSurvivesMoveUndoAndRedo` | Selection is synchronized in both directions between the actual tree and restored model objects. |

### Other editor checks
| File / test | Protects |
| --- | --- |
| `CenteringTests.CentersSelectionBoundsAndPreservesRelativePositions` | Known multi-object bounds give exact centered positions on each axis, including undo/redo. |
| `CenteringTests.CenteringRequiresSelection` | An empty selection cannot start centering. |
| `SelectionShortcutTests.ShortcutsFollowSelectionAndOnlyOfferApplicableActions` | Hints appear/disappear as selection changes; E and L are only offered when applicable. |
| `WorkspaceDefaultsTests.WorkspaceDefaultsLoadUpdateAndApplyToTheNextEditor` | Stored dimensions initialize the editor, update on editing, and initialize the next editor. |
| `HexagonTests.SizeControlsEditCartierDiameterAndHexagonEdgeIndependently` | Actual numeric controls interpret Cartier size as diameter and hexagon size as edge length; editing one does not resize the other. |

## Element contracts, geometry, and transformations

### `ElementContractTests.cs`
| Test | Protects |
| --- | --- |
| `ElementGeometryAndPropertiesSurviveCopyAndProjectReload` | For each element: expected path count/topology, independent copy and ID, all copied properties, saved type/geometry/style/repeat data, reload, and independent width/height where supported. Legacy circles reload as ellipses. |

There is one input row per supported type, not four unrelated suites per type. Type rows remain separate so a failure names the broken element.

### `ShapeGeometryTests.cs`
| Test | Protects |
| --- | --- |
| `CrossCreatesTwoCenteredStrokesAtTheRequestedLengthAndAngle` | The cross's physical stroke lengths, shared center, and crossing angle—not just segment count. |
| `GeometryCreatesClosedEllipseWithExpectedBounds` | Known unequal ellipse dimensions and stroke width produce the right bounds and a closed contour. |
| `CountPatternRepeatsCompleteEllipsesAtWidthGap` | Non-line paths repeat as complete contours with stroke-aware footprint spacing. |
| `DotCreatesStationaryToolpathPointButRetainsItsVisualBounds` | Dots engrave stationary points while keeping circular placement bounds. |
| `WavyRoadCreatesContinuousOpenPath` | Waves are continuous, open, and respect the configured dimensions. |
| `MoonCrescentDepthMovesInnerArc` | Crescent depth changes the inner arc without treating it as another overall size. |
| `EdgeDistancesUseStrokedShapeBounds` | Boundary labels measure visible stroked edges rather than centerline bounds. |

### `GeometryTests.cs`
| Test | Protects |
| --- | --- |
| `LineEndpointsMatchKnownCoordinates` | Known endpoints at horizontal, vertical, and diagonal angles. |
| `RotatedLineBoundingBoxUsesAxisAlignedDimensions` | Rotated lines' placement footprints at different angles. |
| `LineWidthExpandsBoundingBoxOnEveryEdge` | Stroke expands all four sides, not just length. |

### `HexagonTests.cs` calculations
| Test | Protects |
| --- | --- |
| `HexagonHasSixEqualEdgesAtTheRequestedLength` | Side length and center remain correct across sizes/rotations; inherited height cannot distort a regular hexagon. |
| `RepeatSpacingUsesFullHexagonBoundsNotItsEdgeLength` | A hexagon's repeat footprint is larger than one edge. |

### `LineShapeTransformsTests.cs`
| Test | Protects |
| --- | --- |
| `MirrorPreservesWavyRoadWaveCountAndCustomName` | The past bug where mirroring reset wave count and name. |
| `HorizontalMirrorReflectsAcrossDisplayedWidth` | Position, angle, fresh ID, stroke, and independently copied pattern survive horizontal mirroring. |
| `VerticalMirrorReflectsAroundCircumference` | The different circumference reflection formula and angle. |
| `HorizontalMirrorReflectsEntireRepeatedGroupAcrossBracelet` | A repeated line's direction reverses so all copies—not only the prototype—are reflected. |

### `LineSnapperTests.cs`
| Test | Protects |
| --- | --- |
| `SnapsLineBoundingBoxToUsableLeftEdge` | Margin snapping accounts for stroke width. |
| `SnapsRotatedBoundingBoxToOuterRightEdge` | Rotated shapes snap to the physical outside edge as well as margins. |
| `DoesNotSnapOutsideTolerance` | A distant object is not moved by snapping. |

## Repeat calculations and explode

### `PatternGeneratorTests.cs`
| Test | Protects |
| --- | --- |
| `CountModeKeepsSpacingAndRequestedCopiesAfterMoveAndResize` | Known stroke-aware steps stay correct after moving into overflow and requesting many long copies. |
| `CircumferenceFillStartsAtMasterAndEndsAtMatchingTopOffset` | Circumference fill preserves symmetric offsets and distributes count along the available span. |
| `VerticalFillStopsAtBottomMargin` | The implicit vertical gap-fill calculation stops at its bottom margin. This differs from explicit axis-reference overflow. |
| `SharedBoundsRepeatMovesEverySelectedShapeByTheSameGroupStep` | Shared bounds, not individual shape lengths, determine a group's step. |
| `WaveReferenceSpacingRecalculatesAfterResizeAndAddingGap` | Waves touch at zero gap after resizing; adding gap displaces along the reference. Different aspect ratios, rotations, wave counts, and reverse direction are necessary inputs. |
| `IncreasingReferenceFillGapIncreasesSpacingAndReducesCount` | Gap-driven reference fill decreases count while still reaching the guide endpoint. |
| `ReferenceFillDistributesRemainderAndPreservesExcludedIndices` | Remainder spacing and filtered instance numbers are preserved together. |
| `ReferenceCrossFillAnchorsFirstAndLastCentersToEndpoints` | Count/gap and forward/reverse combinations respect diagonal endpoints. |
| `ReferenceLineSpacingAccountsForStrokeAndRelativeAngle` | Parallel, perpendicular, and diagonal reference contact use stroke and relative angle. |
| `ReferenceGroupSpacingUsesDirectionalContactRatherThanProjection` | A thin diagonal group touches correctly instead of being over-spaced by a projected bounding box. |
| `ReferenceDotSpacingUsesDotDiameterNotStrokeWidth` | Dot spacing uses its visual diameter despite a stationary engraving path. |

### `ExplodeTests.cs`
| Test | Protects |
| --- | --- |
| `ExplodePreservesEverySegmentAndStrokeWidth` | Every explodable type keeps exact endpoints, per-component stroke widths, fresh IDs, and no inherited repeat. |
| `ExplodesRepeatCopiesWithoutRestoringExcludedInstances` | Exploding a repeated pattern materializes only the copies that exist. |
| `MixedSelectionExplodesShapesButKeepsDotsAndExistingLines` | Unsupported members remain intact in mixed/empty selection. |

## Actual files and compatibility

### `ProjectFileTests.cs`
| Test | Protects |
| --- | --- |
| `SaveAndOpenRestoreBothBraceletsTheirGuidesAndClearAllEditHistories` | Actual file version/pen/type, geometry/style, references, two bracelets, and fresh histories after opening. |
| `InvalidOrMissingProjectDoesNotReplaceDesignSelectionOrHistory` | Malformed, empty, unsupported, invalid, and missing files cannot discard the current design. |
| `CancellingSaveOrOpenLeavesDesignAndHistoryUnchanged` | Cancellation does not replace selection/state or consume history. |
| `InvalidDesignDoesNotOverwriteAnExistingProject` | Validation failure leaves previous file contents untouched. |

### `DocumentTests.cs`
| Test | Protects |
| --- | --- |
| `LegacyProjectMigratesBraceletsElementsAndDefaultsWithoutLosingDimensions` | One literal old project exercises scaled-variant migration, old circles, square height fallback, Cartier radius/name, and default pen. |
| `InvalidMarginsProduceReadableValidationMessage` | Invalid margins are rejected with actionable feedback. |

### `ExportTests.cs`
| Test | Protects |
| --- | --- |
| `ExportWritesExactlyTheActiveBraceletOrItsSelection` | Complete files for both machines and both export scopes; other bracelets and unselected shapes cannot leak into output. |
| `SelectingOneRepeatInstanceExportsItsWholeParentGroup` | Selecting a child exports its parent pattern in both dialects, not only that child. |
| `RejectedOrCancelledExportDoesNotOverwriteAnExistingFile` | Invalid input, no selection, cancellation, and write errors preserve previous output and design state. |

## CNC safety and simulator mathematics

### `ToolpathAndGCodeTests.cs`
| Test | Protects |
| --- | --- |
| `DisconnectedLineAndDotEachGetRapidPlungeCutAndRetract` | Both operations use safe travel, plunge, cut, and retract; disconnected shapes cannot be joined by an engraving move. |
| `ConnectedCircleSegmentsUseOnePlungeAndRetract` | Connected contours do not retract and plunge at every short segment. |
| `GCodeIncludesUnitsCoordinatesFeedsAndProgramEndUsingInvariantCulture` | Complete Cartesian output keeps decimal dots, feeds, safe Z, and program end under comma-decimal culture. |
| `RotaryToolpathAndOutputMapOneRevolutionToLinearXAndRotaryA` | Both internal commands and complete output map one circumference to A=360 with linear X and no Y. |
| `CartesianMachineMapsBraceletWidthToCenteredXAndCircumferenceToReversedY` | U00's axis mapping and calibrated feeds/depth differ from generic rotary settings. |
| `U00GCodeMatchesKnownMachineDialect` | Exact header/footer, fractional coordinates, formatting, and CRLF for the existing U00 software contract. |
| `U00GCodeRejectsRotaryCommands` | Unsupported commands are rejected rather than silently emitted. |

### `RotaryProjectionTests.cs`
| Test | Protects |
| --- | --- |
| `MapsCircumferencePositionsAroundCylinder` | Quarter-turn radius/depth and axial position. |
| `FullRevolutionIsTessellatedAndReturnsToStart` | Full turns retain enough subdivisions and return to the starting radial position while advancing axially. |
| `SeamCrossingUsesShortContinuousSweep` | A small seam-crossing move is not drawn as a full revolution. |
| `RejectsInvalidCircumference` | Zero, negative, and NaN dimensions are rejected. |

## What was consolidated in this pass

| Previous checks | Replacement |
| --- | --- |
| Copy properties (14 type rows), basic JSON round trips (8), ellipse/cross round trips, closed basic contours (6), independent basic dimensions (5) | One element-contract scenario per type (14 rows). Geometry and migration regressions remain separate where they have different expectations. |
| Seven separate gap/count/switch/resize/guide-edit/normal-Repeat checks | `FillInputsStayInSyncThroughDriverSwitchResizeAndGuideEdit`, with checkpoints after every transition. |
| Create, property edit, fixed-edge line resize, duplication, and continuous-drag undo checks | `AddResizeDuplicateAndDragLineUndoRedoInOrder`, with one saved state per operation and a complete reverse/forward history walk. |
| Repeat-child selection and independent detachment | The real-window select/detach/move/history workflow. |
| Four legacy migration/default checks plus the old Cartier fixture | One literal legacy-project migration test. Current version/pen/type output moved to the actual file workflow. |
| Separate Cartier and hexagon size-editing checks | One real-window size-control workflow with independent elements. |
| Three ordinary Count generation checks | One move/resize/overflow Count scenario. |
| Wave zero-gap, resize-cache, and nonzero-gap checks | One reference-spacing scenario per important geometry configuration. |
| Separate line and stationary-dot toolpaths | One disconnected-job safety sequence checking both and the boundary between them. |
| Rotary mapping and partial rotary-output checks | One mapping-to-complete-output test. Partial A coordinates remain protected by file export tests. |
| Mirror-property copying and horizontal mirror position | One horizontal mirror contract with copied pattern assertions. |
| Defaults initialization and defaults update | One lifecycle: load settings, edit, initialize the next editor. |

No application code was changed. No cases were silently skipped. Tests that are retained protect different branches or different boundaries; narrowing those further requires a deliberate behavior decision, not just a target count.

<details>
<summary>Exact previous test names and replacements</summary>

| Previous name(s) | Replacement |
| --- | --- |
| `CopyPreservesAllPropertiesWithIndependentIdentityAndPattern`, `BasicShapeGeometryCreatesClosedPath`, `BasicShapesRespectIndependentWidthAndHeight`, `BasicShapeTypesRoundTrip`, `EllipseRoundTripsWithIndependentDimensions`, `XShapeRoundTripsThroughProjectJson` | `ElementContractTests.ElementGeometryAndPropertiesSurviveCopyAndProjectReload` |
| `GapInputUpdatesCountAndCanvasImmediately`, `GapInputStaysAtRequestedMinimumWhileRemainderIsDistributed`, `CountInputUpdatesGapAndGeneratedCountImmediately`, `SwitchingDriverKeepsCountAndDisabledInputCannotOverwriteCalculatedValue`, `ResizingRecalculatesDependentValue`, `EditingReferenceLengthUpdatesDependentCount`, `NormalRepeatKeepsBothInputsEditable` | `FillInputTests.FillInputsStayInSyncThroughDriverSwitchResizeAndGuideEdit` |
| `PropertyChangeCanBeUndoneAndRedone`, `AddLineCanBeUndone`, `DuplicateSelectedCopiesTheShapeAndAppliesTheOffset`, `ContinuousDragCreatesOneUndoStep`, `IncreasingLineLengthKeepsItsBottomEdgeFixed` | `UndoRedoTests.AddResizeDuplicateAndDragLineUndoRedoInOrder` |
| `RepeatedShapeExposesSelectableInstanceChildren`, `DetachingARepeatedInstanceCreatesAnIndependentShape` | `WindowInteractionTests.SelectRepeatInstanceDetachMoveAndUndoRedoThroughWindow` |
| `LegacyScaledVariantMigratesToIndependentBracelet`, `SavedCircleMigratesToEqualDimensionEllipse`, `BasicShapeWithoutHeightKeepsLegacyProportions`, `ProjectWithoutPenUsesGenericEngravingPen`, `LegacyCartierFileRetainsRadiusAndCustomName` | `DocumentTests.LegacyProjectMigratesBraceletsElementsAndDefaultsWithoutLosingDimensions` |
| `ProjectRoundTripsAsVersionedJson` | `ProjectFileTests.SaveAndOpenRestoreBothBraceletsTheirGuidesAndClearAllEditHistories` for actual file/version/pen/multiple-bracelet state, plus element contracts for line type and geometry. |
| `CartierDisplaysAndEditsFullDiameter`, `EdgeLengthControlEditsOneSideWithoutDistortingTheHexagon` | `HexagonTests.SizeControlsEditCartierDiameterAndHexagonEdgeIndependently` |
| `CountModeProducesRequestedHorizontalInstancesUpwardAtBoundingBoxGap`, `CountModeIncludesInstancesOutsideUsableBoundary`, `CountModeGeneratesEveryRequestedInstanceRegardlessOfLength` | `PatternGeneratorTests.CountModeKeepsSpacingAndRequestedCopiesAfterMoveAndResize` |
| `ZeroReferenceGapMakesResizedWavyPathsTouch`, `ReferenceSpacingRecalculatesAfterChangingWaveDimensions`, `NonzeroReferenceGapAddsDistanceAlongReference` | `PatternGeneratorTests.WaveReferenceSpacingRecalculatesAfterResizeAndAddingGap` |
| `LineBecomesRapidPlungeCutAndRetract`, `DotBecomesStationaryPlungeAndRetract` | `ToolpathAndGCodeTests.DisconnectedLineAndDotEachGetRapidPlungeCutAndRetract` |
| `RotaryToolpathMapsCircumferenceToZeroThroughThreeSixtyDegrees`, `RotaryGCodeUsesLinearXAndRotaryA` | `ToolpathAndGCodeTests.RotaryToolpathAndOutputMapOneRevolutionToLinearXAndRotaryA`, plus exact partial-angle file exports. |
| `MirrorCopiesPatternSettingsWithoutSharingPatternObject` | `LineShapeTransformsTests.HorizontalMirrorReflectsAcrossDisplayedWidth` |
| `NewDocumentUsesCachedWorkspaceDimensions`, `WorkspaceDimensionChangesUpdateDefaults` | `WorkspaceDefaultsTests.WorkspaceDefaultsLoadUpdateAndApplyToTheNextEditor` |
| `GeometryCreatesTwoDiagonalStrokesAtTheRequestedLength` | Moved and renamed to `ShapeGeometryTests.CrossCreatesTwoCenteredStrokesAtTheRequestedLengthAndAngle`; assertions retained. |

</details>

## Verification

- All 160 cases pass in Debug and Release, with no skipped cases.
- Every remaining named test is present in this inventory; every removed/renamed test is mapped above.
- Before/after coverage comparison retained every previously covered production line and branch outcome: **2,506 → 2,507 lines**, **1,215 → 1,223 branch outcomes**. These counts are execution evidence, not proof of assertion quality.
- The simplified suite detected all **12 sampled application faults** from the earlier experiment. The grouped guide-fill workflow separately detected four persistent placement/bounds/clipping faults. Restored copies passed again.
- Production source fingerprints match the pre-cleanup baseline. Windows execution locally, CNC hardware, exhaustive mutations, and every pointer interaction remain unchecked.
