# Agent Report: VBA Behavior Classification Agent

## Scope

Files read:

- `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Agents_and_Skills_Side_Plan.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\source-inventory.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\officejs-compatibility.md`
- `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\FixDoubleSpaces.bas`
- `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\FixDoubleReturns.bas`
- `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\RemoveNonbreakingSpaces.bas`
- `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\RemoveOptionalHyphens.bas`
- `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\FixSpacesAroundDashes.bas`
- `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\RemoveHighlighting.bas`
- `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\FileCleanerLibrary.bas`
- `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\Common.bas`
- `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\Convert2Text.bas`
- `K:\IdeaProjects\TextCleaner\vb-to-c\TransTools_dotm_Exported\DocCleaner.bas`
- `K:\IdeaProjects\TextCleaner\vb-to-c\TransTools_dotm_Exported\modDocCleaner.bas`
- `K:\IdeaProjects\TextCleaner\vb-to-c\TransTools_dotm_Exported\CUndo.cls`

Files changed:

- `K:\IdeaProjects\TextCleaner\word-addin-modernization\compatibility-matrix.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\mvp-scope.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\agent-reports\vba-behavior-classification-agent.md`

Files intentionally not touched:

- Code files under `src`
- `package.json`
- `appPackage\manifest.json`
- plan files
- all source VBA exports
- `vb-to-c\ETKPlus_Exported - Copy`

## Findings

- The ETKPlus MVP candidate wrapper modules are small, but several call into shared modules for actual behavior.
- `FixDoubleSpaces`, `FixDoubleReturns`, `RemoveOptionalHyphens`, and `FixSpacesAroundDashes` use registration, revision, view, and undo helpers that should not be carried into the Office.js MVP.
- The strongest pure-rule candidates are nonbreaking space normalization, optional hyphen removal, repeated regular-space collapse, and em-dash spacing normalization after guard fixtures.
- `RemoveHighlighting` is feasible as a Word adapter command but is not a pure text rule.
- `FixDoubleReturns` is useful but higher risk because the VBA explicitly handles story ranges and footnotes/endnotes.
- TransTools `DocCleaner` is a broad tag-cleaning system, not a small cleanup macro. It uses UserForms, progress classes, story ranges, shapes, bookmarks, font normalization, native APIs, clipboard behavior, and revision operations.
- TransTools contains useful corroborating behavior for optional hyphen removal and highlight removal, but its full implementation should be deferred.

## Decisions

- Classified ETKPlus NBSP, optional hyphen, repeated space, repeated paragraph, em-dash spacing, and highlighting candidates as `Port to Office.js`.
- Classified TransTools `Trans_DocCleaner` UI launcher as `Redesign UI`.
- Classified TransTools native/helper infrastructure as `VSTO-only`.
- Classified TransTools default tag-cleaning preset and formatting subfeatures as `Defer`.
- Classified revision acceptance and bookmark deletion as `Drop` for the Office.js MVP.
- Recommended first MVP order: NBSP normalization, optional hyphen removal, repeated spaces, selected-range highlight removal, then paragraph/dash rules after fixtures.

## Evidence

Commands run:

- `Get-Content` on the required plans and modernization docs.
- `Get-Content` on all eight requested candidate source files.
- Targeted `Select-String` searches for `Application.Run`, `Selection.`, `ActiveDocument`, `Find`, `Replacement`, `SaveSetting`, `GetSetting`, `PrivateProfileString`, `CommandBars`, `UndoRecord`, and highlighting signals.
- Targeted line-range reads from `FileCleanerLibrary.bas`, `Common.bas`, `Convert2Text.bas`, `DocCleaner.bas`, `modDocCleaner.bas`, and `CUndo.cls`.
- Existence checks for the three owned output paths.

Outputs summarized:

- `source-inventory.md` states that all 349 files in `ETKPlus_Exported - Copy` match `ETKPlus_Exported`; the copy was not used.
- `FixDoubleSpaces.Main` calls `FileCleanerLibrary.FCDoubleSpaces`; implementation replaces multiple regular spaces with one regular space via Word wildcard find.
- `FixDoubleReturns.Main` calls `FileCleanerLibrary.FCDoubleReturns`; implementation collapses repeated paragraph marks and separately cleans footnotes/endnotes.
- `RemoveNonbreakingSpaces.Main` directly replaces Word `^s` with regular space using `Selection.Find`.
- `RemoveOptionalHyphens.Main` runs `FileCleanerLibrary.RemoveOptionalHyphens`; implementation replaces Word `^-` with empty string.
- `FixSpacesAroundDashes.Main` calls `DeleteRegularSpacesAroundDashes`; code removes regular spaces around em dashes and comments out en-dash cleanup as dangerous.
- `RemoveHighlighting.Main` calls `Convert2Text.RemoveHighlighting`; implementation selects the document range and sets highlight to `wdNoHighlight`.
- `DocCleaner.Trans_DocCleaner` launches `frmDocCleaner` modeless and depends on an application window handle.
- `DocCleaner.pCleanTags` orchestrates multiple broad operations, including bookmarks and font normalization.
- `modDocCleaner.bas` includes native `Declare` calls, story/shape traversal, selection cut/paste leveling, and `ActiveDocument` revision/bookmark operations.
- `rg` was attempted first but failed with an access-denied error from the packaged app path, so PowerShell targeted reads/searches were used.

## Risks

- Exact Word wildcard semantics do not always map 1:1 to JavaScript regex or Office.js search. Implementation must write tests from the recorded examples and validate in Word.
- Repeated paragraph cleanup can damage paragraph styles or document structures if implemented as whole-text replacement.
- Highlight clearing and dash spacing can affect mixed formatting unless the adapter works on ranges carefully.
- Track changes behavior is unresolved for MVP. VBA silently toggles or accepts revision state in places; Office.js MVP should warn and avoid hidden revision changes.
- TransTools shape/text-frame behavior may be valuable later, but Office.js coverage must be verified separately.
- Some PowerShell sessions emitted profile-related PSReadLine warnings and occasional CLIXML import warnings; source reads and target file writes still completed.

## Next handoff

- Office.js Implementation Agent should implement pure tests first for NBSP, optional hyphens, and repeated spaces, then add selected-range adapters.
- Task Pane/Manifest Agent should expose only the selected MVP rules and avoid TransTools profile/UI parity.
- Document Fidelity Agent should create fixtures for mixed formatting, tracked changes, fields, hyperlinks, tables, highlighting, paragraph styles, and footnotes/endnotes before any document-wide cleanup is enabled.
- VSTO/Fallback Agent should review only the `VSTO-only` infrastructure row if exact native/desktop TransTools parity becomes a requirement.

