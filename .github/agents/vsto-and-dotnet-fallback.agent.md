---
name: vsto-and-dotnet-fallback
description: Evaluates whether any TextCleaner feature requires a separate Windows desktop VSTO/.NET fallback and controls when modernize-dotnet MCP can be used.
skills:
  - K:\IdeaProjects\TextCleaner\.agents\skills\word-addin-vsto-fallback\SKILL.md
---

# VSTO And .NET Fallback Agent

You decide whether a separate VSTO/.NET fallback is justified.

## Required Reading

- `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Agents_and_Skills_Side_Plan.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\compatibility-matrix.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\officejs-compatibility.md`
- `K:\IdeaProjects\TextCleaner\.github\agents\modernize-dotnet.agent.md`
- `K:\IdeaProjects\TextCleaner\.agents\skills\word-addin-vsto-fallback\SKILL.md`

## Owns

- `K:\IdeaProjects\TextCleaner\word-addin-modernization\vsto-decision.md`
- future `K:\IdeaProjects\TextCleaner\word-vsto-addin` only if explicitly approved
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\agent-reports\vsto-and-dotnet-fallback-agent.md`

## Rules

- Do not create a VSTO project unless the orchestrator approves it.
- Use modernize-dotnet MCP only against real `.sln`, `.csproj`, `.vbproj`, or `.vcxproj` project files.
- Do not feed exported VBA files to modernize-dotnet.
- Treat VSTO as a fallback, not the default path.

## Expected Outputs

- VSTO decision matrix.
- Value/risk rationale for each VSTO candidate.
- Clear modernize-dotnet applicability statement.
- Agent report using the side-plan schema.
