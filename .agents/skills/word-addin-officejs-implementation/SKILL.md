---
name: word-addin-officejs-implementation
description: Workspace-only Office.js implementation skill for TextCleaner. Use when implementing tested TypeScript cleanup rules, selected-range Word adapters, migration-log entries, or MVP cleanup behavior.
---

# Word Add-in Office.js Implementation

Implement cleanup behavior with tests and traceability.

## Inputs

- `word-addin-modernization\compatibility-matrix.md`
- `word-addin-modernization\mvp-scope.md`
- `word-addin-modernization\officejs-compatibility.md`

## Outputs

- `src\core\cleanupTypes.ts`
- `src\core\cleanupRules.ts`
- `src\core\protectedRanges.ts`
- `src\core\wordAdapters.ts`
- `src\tests\cleanupRules.test.ts`
- `word-addin-modernization\migration-log.md`
- `word-addin-modernization\agent-reports\officejs-implementation-agent.md`

## Rules

- Write tests before implementation for pure rules.
- Start with pure rules: NBSP normalization, optional hyphen removal, repeated regular spaces.
- Keep adapters selected-range only until fixture validation exists.
- Do not edit manifests, package files, task pane files, command files, or VBA files unless explicitly reassigned.
- Run the smallest meaningful verification command and record exact results.
