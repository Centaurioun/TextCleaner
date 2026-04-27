---
name: word-addin-taskpane-manifest
description: Workspace-only task pane and manifest skill for TextCleaner. Use when repairing the Office add-in scaffold, changing Word debug defaults, removing placeholder branding, editing task pane UI, or validating manifest changes.
---

# Word Add-in Task Pane And Manifest

Repair the scaffold into a private Word-focused add-in.

## Inputs

- `package.json`
- `appPackage\manifest.json`
- `webpack.config.js`
- `src\taskpane`
- `src\commands`
- `word-addin-modernization\officejs-compatibility.md`

## Outputs

- Updated scaffold files assigned by the orchestrator.
- `word-addin-modernization\agent-reports\task-pane-and-manifest-agent.md`

## Rules

- Change default debug host to Word only when assigned.
- Remove placeholder branding such as template vendor names and sample labels.
- Keep first command safe: open the TextCleaner pane.
- Do not add destructive one-click cleanup before selected-range verification.
- Run `npm run validate` after manifest changes.
