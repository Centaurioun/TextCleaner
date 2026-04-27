---
name: word-addin-document-verification
description: Workspace-only document fidelity skill for TextCleaner. Use when verifying builds, manifest validation, Word sideloading, fixture documents, formatting preservation, selected-range behavior, or Word version evidence.
---

# Word Add-in Document Verification

Prove behavior against real Word documents and commands.

## Inputs

- `word-addin-modernization\mvp-scope.md`
- implemented cleanup code
- project npm scripts
- actual Word desktop environment

## Outputs

- `word-addin-modernization\test-documents`
- `word-addin-modernization\compatibility-results.md`
- `word-addin-modernization\verification-log.md`
- `word-addin-modernization\agent-reports\document-fidelity-and-verification-agent.md`

## Rules

- Run selected-range tests before document-wide tests.
- Record Word version/build and Office channel.
- Verify build and manifest commands before compatibility claims.
- Include fixtures for styles, tables, comments, hyperlinks, fields, bookmarks, tracked changes, and footnotes/endnotes when those behaviors are in scope.
