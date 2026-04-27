---
name: word-addin-officejs-compatibility
description: Workspace-only Office.js compatibility skill for TextCleaner. Use when checking Word JavaScript API requirement sets, unified manifest behavior, WordApiOnline boundaries, sideloading, validation, and Office add-in scaffold constraints.
---

# Word Add-in Office.js Compatibility

Define Office add-in compatibility boundaries before implementation.

## Inputs

- `package.json`
- `appPackage\manifest.json`
- `src\taskpane\word.ts`
- `src\commands\word.ts`
- Microsoft Learn Office Add-ins documentation

## Outputs

- `word-addin-modernization\officejs-compatibility.md`
- `word-addin-modernization\agent-reports\officejs-compatibility-agent.md`

## Rules

- Use Microsoft Learn for current external facts.
- Check runtime APIs with `Office.context.requirements.isSetSupported`.
- Treat `WordApiOnline` as online-only and outside the desktop Word MVP.
- Do not add `WordApiOnline` as an activation requirement.
- Keep unified manifest unless evidence requires a second manifest path.
- Do not modify code or manifests unless explicitly reassigned.
