---
name: word-addin-vsto-fallback
description: Workspace-only VSTO fallback skill for TextCleaner. Use when deciding whether Office.js-blocked features require a separate Windows desktop VSTO/.NET add-in and when controlling modernize-dotnet MCP usage.
---

# Word Add-in VSTO Fallback

Keep VSTO as a documented fallback, not the default path.

## Inputs

- `word-addin-modernization\compatibility-matrix.md`
- `word-addin-modernization\officejs-compatibility.md`
- `.github\agents\modernize-dotnet.agent.md`

## Outputs

- `word-addin-modernization\vsto-decision.md`
- `word-addin-modernization\agent-reports\vsto-and-dotnet-fallback-agent.md`
- future `word-vsto-addin` only if explicitly approved

## Rules

- Do not create VSTO files without orchestrator approval.
- Use modernize-dotnet only when real `.sln`, `.csproj`, `.vbproj`, or `.vcxproj` files exist.
- Do not run modernize-dotnet against `.bas`, `.frm`, `.cls`, `.doccls`, `.dotm`, or loose exported `.vb`.
- Require value/risk rationale for every VSTO candidate.
