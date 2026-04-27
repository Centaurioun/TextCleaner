# MVP Scope

Generated: 2026-04-28

Owner: VBA Behavior Classification Agent

## Recommended First MVP Features

| Priority | Feature | Source | MVP target | Rationale |
|---:|---|---|---|---|
| 1 | Normalize nonbreaking spaces | `vb-to-c\ETKPlus_Exported\RemoveNonbreakingSpaces.bas`; `FileCleanerLibrary.RemoveNonbreakingSpaces` | Pure rule plus selected-range Word adapter | Simple literal behavior: Word `^s` -> regular space. Low semantic risk and easy fixtures. |
| 2 | Remove optional hyphens | `vb-to-c\ETKPlus_Exported\RemoveOptionalHyphens.bas`; `FileCleanerLibrary.RemoveOptionalHyphens` | Pure rule plus selected-range Word adapter | Simple literal behavior: Word `^-` -> empty string. Also corroborated by TransTools `OpResetHyphens`. |
| 3 | Collapse repeated regular spaces | `vb-to-c\ETKPlus_Exported\FixDoubleSpaces.bas`; `FileCleanerLibrary.FCDoubleSpaces` | Pure rule plus selected-range Word adapter | Deterministic wildcard behavior. Needs fixture checks for formatting preservation across replacements. |
| 4 | Remove highlighting in selected range | `vb-to-c\ETKPlus_Exported\RemoveHighlighting.bas`; `Convert2Text.RemoveHighlighting` | Word adapter only, selected range first | Useful formatting cleanup and Office.js-compatible in principle. Not a pure text rule. |
| 5 | Collapse repeated paragraph marks | `vb-to-c\ETKPlus_Exported\FixDoubleReturns.bas`; `FileCleanerLibrary.FCDoubleReturns` | Word-adapter candidate after fixtures | Valuable but paragraph/story/footnote behavior is riskier than inline text replacements. |
| 6 | Remove regular spaces around em dashes | `vb-to-c\ETKPlus_Exported\FixSpacesAroundDashes.bas`; `FileCleanerLibrary.DeleteRegularSpacesAroundDashes` | Pure rule after dash fixtures, then selected-range adapter | Feasible, but the VBA comments warn against overbroad en-dash behavior. Needs exact em-dash-only fixtures before implementation. |

## Excluded From First MVP

| Excluded feature | Source | Reason |
|---|---|---|
| ETKPlus registration/reminder checks | ETKPlus wrappers via `ETKPlusLibrary.RegReminder` | Licensing/reminder gate is not cleanup behavior and does not belong in a private Office.js MVP. |
| Hidden revision/view toggling | `Common.CheckRevisions`, `ResetRevisions`, `CheckView`, `ResetView` | Uses `ActiveDocument`, `ActiveWindow`, and `SaveSetting`/`GetSetting`; changes document/UI state implicitly. Replace with explicit warnings and fixture tests. |
| ETKPlus custom undo grouping | `Application.UndoRecord` in `FileCleanerLibrary` | No direct Office.js equivalent. Keep operations narrow and previewable. |
| TransTools modeless DocCleaner form | `DocCleaner.Trans_DocCleaner`, `frmDocCleaner.frm` | Requires UI redesign into task pane controls and add-in storage. |
| TransTools default tag-cleaner preset | `DocCleaner.pCleanTags`, `modDocCleaner.bas` | Broad destructive suite spanning spacing, shading, hyphens, fonts, bookmarks, revisions, shapes, and stories. Defer until subfeatures are isolated. |
| Revision acceptance | `OCRRevisionsOff` | Destructive editorial change; drop from MVP. |
| Bookmark deletion | `TTDeleteBookmarks` | High document-integrity risk for fields, TOCs, cross-references, and navigation. Drop from MVP. |
| Native Windows and clipboard helpers | `modDocCleaner.bas`, `modClipboard.bas`, `CUndo.cls`, related classes | Not Office.js-compatible. VSTO-only infrastructure if exact desktop parity is later justified. |
| Shape/text-frame full coverage | TransTools `OCRReplaceTextShape`, `OCRSetRangeFormattingShape` | Useful later, but needs separate Office.js feasibility and fixture work. |

## Pure-Rule Candidates

These can be implemented and tested without Word first, but still require Word adapter tests before shipping.

| Rule ID candidate | Source behavior | Example fixture |
|---|---|---|
| `normalizeNonbreakingSpaces` | `^s` -> regular space | `A\u00A0B` -> `A B` |
| `removeOptionalHyphens` | `^-` -> empty string | `hy\u00ADphen` -> `hyphen` |
| `normalizeRepeatedSpaces` | two or more regular spaces -> one regular space | `A  B   C` -> `A B C` |
| `normalizeEmDashSpacing` | remove regular spaces adjacent to em dash only | `A \u2014 B` -> `A\u2014B`; `A \u2013 B` unchanged |

Not pure-rule first: remove highlighting, paragraph mark cleanup in real Word content, and TransTools formatting normalization.

## Word-Adapter Candidates

| Adapter candidate | Required behavior | Initial boundary |
|---|---|---|
| Selected-range text replacement | Apply pure text rules to the current selection without whole-document rewrite. | Selection only; refuse or warn on empty selection. |
| Targeted search replacement | Use Word search/range replacements for NBSP, optional hyphen, repeated spaces, and em-dash spacing where it preserves formatting better than replacing full selected text. | Start with one rule at a time and report replacement counts. |
| Selected-range highlight reset | Clear highlight formatting without changing text content. | Selection only; fixture-test mixed highlighted and non-highlighted runs. |
| Paragraph cleanup | Collapse extra paragraph marks while preserving paragraph styles and not damaging tables/fields. | Defer until paragraph fixtures pass. |
| Footnote/endnote cleanup | Match ETKPlus extra-return cleanup in notes. | Defer until main paragraph cleanup is stable. |

## Fixture Needs

Minimum pure text fixtures:

| Fixture | Purpose |
|---|---|
| `nbsp-basic` | NBSP between words becomes regular space. |
| `soft-hyphen-basic` | Optional hyphen is removed. |
| `repeated-spaces-basic` | Multiple regular spaces collapse to one. |
| `em-dash-spacing-basic` | Spaces around em dash are removed. |
| `en-dash-guard` | Spaces around en dash are not changed by em-dash rule. |

Minimum Word document fixtures:

| Fixture | Purpose |
|---|---|
| `selection-formatting-basic.docx` | Replacements preserve bold/italic/links around selected text. |
| `highlight-mixed.docx` | Remove highlighting only where selected and preserve text/font styling. |
| `paragraph-styles.docx` | Repeated paragraph cleanup preserves paragraph styles and headings. |
| `footnotes-endnotes-returns.docx` | Validate ETKPlus note-return behavior before enabling note cleanup. |
| `tables-fields-hyperlinks.docx` | Prove selected-range operations do not damage tables, fields, and hyperlinks. |
| `tracked-changes.docx` | Confirm behavior when track changes is enabled; MVP should warn rather than silently toggle state. |

## Acceptance Checks

First MVP acceptance:

- Pure rules for NBSP, optional hyphen, and repeated regular spaces have deterministic unit tests with source-mapped examples.
- Selected-range cleanup works without whole-document `body.clear()` or equivalent reconstructive rewrite.
- Replacement counts are reported per rule.
- Empty selection and unsupported Word API states produce clear non-mutating messages.
- Remove-highlighting command has a selected-range fixture with mixed formatting.
- Paragraph cleanup and em-dash spacing are disabled until their fixtures pass.
- No registry/profile behavior, UserForms, native Windows calls, revision acceptance, bookmark deletion, or template/macro mutation is carried into the MVP.
- `migration-log.md` rows map each implemented rule to the source files above.

## Handoff Notes

- Implementation agent should start with `normalizeNonbreakingSpaces`, `removeOptionalHyphens`, and `normalizeRepeatedSpaces`.
- Office.js adapter should prefer selected-range or targeted search operations over full selected-text reconstruction where formatting preservation is uncertain.
- Task pane should expose a small checklist of enabled rules plus preview/clean selection actions; TransTools DocCleaner profiles should not be recreated for MVP.
- Document-fidelity agent should test the exact guardrails for track changes, hyperlinks, fields, tables, footnotes/endnotes, and mixed formatting before any document-wide command is enabled.

