# Source Inventory And Provenance

Generated: 2026-04-27 23:46:28 +03:00

Source root: `K:\IdeaProjects\TextCleaner\vb-to-c`.

## Summary

- Total files under `vb-to-c`: 889.
- `ETKPlus_Exported` and `ETKPlus_Exported - Copy` each contain 349 files.
- Full relative-path SHA-256 comparison found 349/349 matching ETKPlus file pairs and 0 mismatches.
- Canonical source recommendation: use `vb-to-c\ETKPlus_Exported` for ETKPlus module-level porting; treat `ETKPlus_Exported - Copy` as a byte-identical duplicate backup and do not port from it.
- Use `vb-to-c\TransTools_dotm_Exported` as the canonical TransTools module export root.
- Treat root-level `*-VBACode.vb` files as combined exports for provenance/search only unless a needed module is absent from the split exports.

## File Counts By Extension

| Extension | Count |
| --- | --- |
| .bas | 566 |
| .frm | 136 |
| .frx | 136 |
| .cls | 36 |
| .vb | 10 |
| .doccls | 3 |
| .dotm | 2 |

## Main Source Groups

| Path | Files |
| --- | --- |
| ETKPlus_Exported | 349 |
| ETKPlus_Exported - Copy | 349 |
| NewETKPlus | 1 |
| NewTransTools | 1 |
| TransTools_dotm_Exported | 178 |

## Root Files And Combined Exports

| Path | Length |
| --- | --- |
| Author Tools-VBACode.vb | 10284 |
| Automacs-VBACode.vb | 2426 |
| CCorrectionListEntry.cls | 22255 |
| Editioning-VBACode.vb | 6862 |
| ETKPlus -Copy-VBACode.vb | 1578546 |
| ETKPlus-VBACode.vb | 662993 |
| ETKPlus.dotm-VBACode.vb | 1578922 |
| Install-ETKPlus-VBACode.vb | 6544 |
| SpaceCadet-VBACode.vb | 31716 |
| TransTools-dot-VBACode.vb | 4005457 |
| TransTools-dotm-VBACode.vb | 4006189 |

## Module Export Counts By Group

| Group | Extension | Count |
| --- | --- | --- |
| [root combined exports] | .cls | 1 |
| [root combined exports] | .vb | 10 |
| ETKPlus_Exported - Copy | .bas | 253 |
| ETKPlus_Exported - Copy | .cls | 1 |
| ETKPlus_Exported - Copy | .doccls | 1 |
| ETKPlus_Exported - Copy | .frm | 47 |
| ETKPlus_Exported - Copy | .frx | 47 |
| ETKPlus_Exported | .bas | 253 |
| ETKPlus_Exported | .cls | 1 |
| ETKPlus_Exported | .doccls | 1 |
| ETKPlus_Exported | .frm | 47 |
| ETKPlus_Exported | .frx | 47 |
| NewETKPlus | .dotm | 1 |
| NewTransTools | .dotm | 1 |
| TransTools_dotm_Exported | .bas | 60 |
| TransTools_dotm_Exported | .cls | 33 |
| TransTools_dotm_Exported | .doccls | 1 |
| TransTools_dotm_Exported | .frm | 42 |
| TransTools_dotm_Exported | .frx | 42 |

## Main Source Groups And Use

| Group | Role | Recommended handling |
|---|---|---|
| `ETKPlus_Exported` | Editors Toolkit Plus split VBA exports: standard modules, one class, one document class, forms, and FRX assets. | Primary ETKPlus source for behavior classification and MVP porting. |
| `ETKPlus_Exported - Copy` | Same relative paths and byte-identical content as `ETKPlus_Exported`. | Ignore for porting; keep only as duplicate evidence. |
| `TransTools_dotm_Exported` | TransTools split VBA exports with larger class/module set. | Primary TransTools source when classifying TransTools features. |
| `NewETKPlus` / `NewTransTools` | Macro-enabled `.dotm` templates. | Provenance/reference only; do not port directly or mutate. |
| Root `*-VBACode.vb` files | Large combined template exports and smaller combined macro packages. | Search/reference only unless split export is missing or later diff proves a newer source. |

## Combined `.vb` Exports Versus Module Exports

- The 10 root `.vb` files are combined exports, including large ETKPlus and TransTools template dumps.
- Split exports are more suitable for migration because they preserve module/class/form boundaries and produce stable source paths for `migration-log.md`.
- Combined ETKPlus exports (`ETKPlus.dotm-VBACode.vb`, `ETKPlus -Copy-VBACode.vb`, `ETKPlus-VBACode.vb`) overlap conceptually with the split ETKPlus module exports; do not mix them as independent source roots without a later diff-backed reason.
- Combined TransTools exports (`TransTools-dotm-VBACode.vb`, `TransTools-dot-VBACode.vb`) overlap conceptually with `TransTools_dotm_Exported`; use split TransTools files first.
- Smaller root combined exports (`Author Tools-VBACode.vb`, `Automacs-VBACode.vb`, `Editioning-VBACode.vb`, `Install-ETKPlus-VBACode.vb`, `SpaceCadet-VBACode.vb`) are separate provenance inputs, but not recommended MVP sources unless the behavior agent explicitly selects them.

## Combined `.vb` File Sizes

| RelativePath | Length |
| --- | --- |
| Author Tools-VBACode.vb | 10284 |
| Automacs-VBACode.vb | 2426 |
| Editioning-VBACode.vb | 6862 |
| ETKPlus -Copy-VBACode.vb | 1578546 |
| ETKPlus-VBACode.vb | 662993 |
| ETKPlus.dotm-VBACode.vb | 1578922 |
| Install-ETKPlus-VBACode.vb | 6544 |
| SpaceCadet-VBACode.vb | 31716 |
| TransTools-dot-VBACode.vb | 4005457 |
| TransTools-dotm-VBACode.vb | 4006189 |

## Compatibility Signal Snapshot

These counts are included only to help the next behavior-classification handoff choose risk areas; no code was changed.

## High-Risk VBA Signals

| Signal | Matches | Files |
| --- | --- | --- |
| Selection\. | 8619 | 268 |
| ActiveDocument | 4513 | 288 |
| Application.Run | 736 | 110 |
| SaveSetting | 2396 | 185 |
| GetSetting | 4168 | 200 |
| PrivateProfileString | 427 | 11 |
| Declare  | 332 | 18 |
| PtrSafe | 162 | 12 |
| LongPtr | 492 | 43 |
| CommandBars | 69 | 10 |
| Application.TaskPanes | 124 | 10 |
| MacScript | 24 | 8 |
| AppleScript | 16 | 4 |
| VBProject | 26 | 10 |
| UndoRecord | 267 | 39 |

## Hash Evidence

- Full hash table: `word-addin-modernization\source-hashes.csv`.
- Compared all relative paths present in `ETKPlus_Exported` or `ETKPlus_Exported - Copy`.
- All 349 compared rows matched by SHA-256 and byte length.
- No `source-diff-notes.md` was created because there were no mismatches.


