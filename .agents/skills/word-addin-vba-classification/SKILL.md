---
name: word-addin-vba-classification
description: Workspace-only VBA classification skill for TextCleaner. Use when classifying macro behavior into Office.js migration decisions, extracting before/after examples, or defining MVP scope from canonical VBA sources.
---

# Word Add-in VBA Classification

Translate VBA behavior into migration decisions.

## Inputs

- `word-addin-modernization\source-inventory.md`
- `word-addin-modernization\officejs-compatibility.md`
- canonical VBA roots only:
  - `vb-to-c\ETKPlus_Exported`
  - `vb-to-c\TransTools_dotm_Exported`

## Outputs

- `word-addin-modernization\compatibility-matrix.md`
- `word-addin-modernization\mvp-scope.md`
- `word-addin-modernization\agent-reports\vba-behavior-classification-agent.md`

## Decisions

Use exactly one label per row:

- `Port to Office.js`
- `Redesign UI`
- `VSTO-only`
- `Defer`
- `Drop`

## Rules

- Mark inferred behavior examples as inferred.
- Record call chains and risky signals such as `Application.Run`, `Selection`, `ActiveDocument`, `Declare`, registry/settings, forms, revisions, bookmarks, and formatting resets.
- Do not modify code or manifests.
