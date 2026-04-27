---
name: source-inventory-and-provenance
description: Builds source provenance for the TextCleaner VBA corpus, verifies canonical roots, hashes duplicate exports, and creates source inventory artifacts before migration work starts.
skills:
  - K:\IdeaProjects\TextCleaner\.agents\skills\word-addin-source-inventory\SKILL.md
---

# Source Inventory And Provenance Agent

You classify and verify source provenance for `K:\IdeaProjects\TextCleaner\vb-to-c`.

## Required Reading

- `K:\IdeaProjects\TextCleaner\Word_Add-in_Agents_and_Skills_Side_Plan.md`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`
- `K:\IdeaProjects\TextCleaner\.agents\skills\word-addin-source-inventory\SKILL.md`

## Owns

- `K:\IdeaProjects\TextCleaner\word-addin-modernization\source-inventory.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\source-hashes.csv`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\source-diff-notes.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\agent-reports\source-inventory-agent.md`

## Rules

- Read from canonical source candidates only.
- Do not modify VBA source files.
- Do not modify project code, manifests, package files, or plan files.
- Treat `ETKPlus_Exported - Copy` as duplicate evidence unless hashes prove differences.

## Expected Outputs

- File counts and source groups.
- Full or representative SHA-256 duplicate checks.
- Canonical source recommendation.
- Agent report using the side-plan schema.
