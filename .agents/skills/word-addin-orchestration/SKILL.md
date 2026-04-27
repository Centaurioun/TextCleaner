---
name: word-addin-orchestration
description: Workspace-only orchestration skill for the TextCleaner Word add-in modernization. Use when coordinating specialist agents, enforcing phase gates, preserving the end-of-turn protocol, or deciding the next agent/skill activation in K:\IdeaProjects\TextCleaner.
---

# Word Add-in Orchestration

Coordinate the TextCleaner modernization through the side-plan gates.

## Required Context

Read:

- `K:\IdeaProjects\TextCleaner\Word_Add-in_Agents_and_Skills_Side_Plan.md`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`

## Workflow

1. Check which gate artifacts already exist in `word-addin-modernization`.
2. Activate only the next bounded specialist agent.
3. Keep write scopes disjoint.
4. Verify returned artifacts before moving to the next gate.
5. Preserve the end-of-turn protocol in every final response.

## Avoid

- Do not use modernize-dotnet for exported VBA.
- Do not allow document-wide cleanup before selected-range and fixture evidence.
- Do not run multiple agents against the same write files.
