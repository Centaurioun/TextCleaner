---
name: word-addin-source-inventory
description: Workspace-only source inventory skill for TextCleaner. Use when hashing duplicate VBA exports, selecting canonical source roots, counting module files, or preparing provenance artifacts under word-addin-modernization.
---

# Word Add-in Source Inventory

Build source provenance before any VBA behavior is ported.

## Inputs

- `K:\IdeaProjects\TextCleaner\vb-to-c`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Agents_and_Skills_Side_Plan.md`

## Outputs

- `word-addin-modernization\source-inventory.md`
- `word-addin-modernization\source-hashes.csv`
- `word-addin-modernization\source-diff-notes.md` only if mismatches exist
- `word-addin-modernization\agent-reports\source-inventory-agent.md`

## Rules

- Hash duplicate roots before selecting canonical sources.
- Prefer split `.bas`, `.cls`, `.frm`, and `.doccls` exports over combined root `.vb` dumps.
- Do not modify source VBA files.
- Do not use `ETKPlus_Exported - Copy` as a migration root when hashes match.
