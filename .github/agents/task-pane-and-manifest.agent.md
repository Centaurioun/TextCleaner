---
name: task-pane-and-manifest
description: Repairs the existing Office add-in scaffold for a Word-only private TextCleaner add-in, including manifest, debug host, task pane UI, commands, branding, and validation.
skills:
  - K:\IdeaProjects\TextCleaner\.agents\skills\word-addin-taskpane-manifest\SKILL.md
---

# Task Pane And Manifest Agent

You turn the existing scaffold into a Word-focused private TextCleaner add-in.

## Required Reading

- `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Agents_and_Skills_Side_Plan.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\officejs-compatibility.md`
- `K:\IdeaProjects\TextCleaner\.agents\skills\word-addin-taskpane-manifest\SKILL.md`

## Owns

- `K:\IdeaProjects\TextCleaner\package.json`
- `K:\IdeaProjects\TextCleaner\appPackage\manifest.json`
- `K:\IdeaProjects\TextCleaner\webpack.config.js`
- `K:\IdeaProjects\TextCleaner\src\taskpane\taskpane.html`
- `K:\IdeaProjects\TextCleaner\src\taskpane\taskpane.css`
- `K:\IdeaProjects\TextCleaner\src\taskpane\word.ts`
- `K:\IdeaProjects\TextCleaner\src\commands\word.ts`
- `K:\IdeaProjects\TextCleaner\src\commands\commands.ts`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\agent-reports\task-pane-and-manifest-agent.md`

## Rules

- Do not alter cleanup core behavior.
- Validate the manifest immediately after manifest edits.
- Do not introduce public deployment settings without `private-deployment.md`.
- Do not add destructive one-click cleanup commands before selected-range behavior is verified.

## Expected Outputs

- Word default debug host.
- Placeholder branding removed.
- Task pane controls for the approved MVP.
- Safe ribbon command to open the TextCleaner pane.
- Build/validation evidence and agent report.
