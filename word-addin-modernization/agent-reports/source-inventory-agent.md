# Agent Report: Source Inventory And Provenance Agent

## Scope
Files read:
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Agents_and_Skills_Side_Plan.md`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`
- `K:\IdeaProjects\TextCleaner\vb-to-c`

Files changed:
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\source-inventory.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\source-hashes.csv`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\agent-reports\source-inventory-agent.md`

Files intentionally not touched:
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Agents_and_Skills_Side_Plan.md`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`
- `K:\IdeaProjects\TextCleaner\vb-to-c\**`
- `K:\IdeaProjects\TextCleaner\src\**`
- `K:\IdeaProjects\TextCleaner\appPackage\manifest.json`
- `K:\IdeaProjects\TextCleaner\package.json`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\source-diff-notes.md` (not needed; no mismatches found)

## Findings
- `vb-to-c` contains 889 files: 566 `.bas`, 136 `.frm`, 136 `.frx`, 36 `.cls`, 10 `.vb`, 3 `.doccls`, and 2 `.dotm`.
- `ETKPlus_Exported` and `ETKPlus_Exported - Copy` each contain 349 files.
- Complete relative-path SHA-256 comparison found 349 matching ETKPlus pairs and 0 mismatches.
- `TransTools_dotm_Exported` contains 178 files and should be treated as its own canonical split-export source root.
- Root `*-VBACode.vb` files are combined exports and are less suitable as primary migration sources than split `.bas`/`.cls`/`.frm` exports.

## Decisions
- Recommend `vb-to-c\ETKPlus_Exported` as canonical ETKPlus source.
- Recommend ignoring `vb-to-c\ETKPlus_Exported - Copy` for porting because it is byte-identical to `ETKPlus_Exported` across all 349 relative paths.
- Recommend `vb-to-c\TransTools_dotm_Exported` as canonical TransTools source.
- Do not create `source-diff-notes.md` unless a future refresh finds mismatches.

## Evidence
Commands run:
- `Get-ChildItem -LiteralPath K:\IdeaProjects\TextCleaner\vb-to-c -Recurse -File | Group-Object Extension`
- `Get-ChildItem` group counts for top-level `vb-to-c` directories.
- SHA-256 comparison using `Get-FileHash -Algorithm SHA256` for every relative path in `ETKPlus_Exported` and `ETKPlus_Exported - Copy`.
- `Select-String` high-risk signal scan across `.bas`, `.cls`, `.frm`, `.vb`, and `.doccls` files.

Outputs summarized:
- File counts and source groups are recorded in `source-inventory.md`.
- Full 349-row hash comparison is recorded in `source-hashes.csv`.
- Hash summary: 349 matches, 0 mismatches.

## Risks
- Combined root `.vb` exports may contain historical or alternate template states; this pass did not perform semantic module-level diffs against split exports.
- `.frx` files are binary UserForm assets; this pass hashed only ETKPlus duplicate paths for provenance and did not interpret binary form content.
- The `Selection.` and `ActiveDocument` signal counts are high, so later porting must avoid direct cursor-model translation.

## Next handoff
- VBA Behavior Classification Agent should read `source-inventory.md` and use canonical split-export roots only.
- Start MVP classification with `ETKPlus_Exported\FixDoubleSpaces.bas`, `FixDoubleReturns.bas`, `RemoveNonbreakingSpaces.bas`, `RemoveOptionalHyphens.bas`, `FixSpacesAroundDashes.bas`, and `RemoveHighlighting.bas`.
- Treat `ETKPlus_Exported - Copy` as duplicate evidence, not an independent source.

