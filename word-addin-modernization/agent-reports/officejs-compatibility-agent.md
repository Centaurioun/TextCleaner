# Agent Report: Office.js Compatibility Agent

## Scope

Files read:

- `K:\IdeaProjects\TextCleaner\Word_Add-in_Agents_and_Skills_Side_Plan.md`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`
- `K:\IdeaProjects\TextCleaner\package.json`
- `K:\IdeaProjects\TextCleaner\appPackage\manifest.json`
- `K:\IdeaProjects\TextCleaner\src\taskpane\word.ts`
- `K:\IdeaProjects\TextCleaner\src\commands\word.ts`

Files changed:

- `K:\IdeaProjects\TextCleaner\word-addin-modernization\officejs-compatibility.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\agent-reports\officejs-compatibility-agent.md`

Files intentionally not touched:

- `K:\IdeaProjects\TextCleaner\package.json`
- `K:\IdeaProjects\TextCleaner\appPackage\manifest.json`
- `K:\IdeaProjects\TextCleaner\src\taskpane\word.ts`
- `K:\IdeaProjects\TextCleaner\src\commands\word.ts`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Agents_and_Skills_Side_Plan.md`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`

## Findings

- The existing project is already an Office add-in scaffold and should be repaired in place for the MVP.
- `package.json` still defaults `config.app_to_debug` to `excel`, but includes `npm run start:desktop:word`.
- `appPackage\manifest.json` is a unified Microsoft 365 JSON manifest using `manifestVersion: devPreview`.
- The manifest still includes multi-host scopes for mail, workbook, document, and presentation.
- Placeholder branding remains in the manifest, including `Contoso` and `Contoso Add-in`.
- Word task pane and command code still use template `Hello World` behavior.
- MVP text cleanup features should be built around selected-range and targeted range operations before any document-wide cleanup.
- `WordApiOnline` is online-only, cannot be a manifest activation requirement, and must not be a desktop MVP dependency.
- modernize-dotnet is not applicable to the current VBA export corpus; it applies only to future real .NET/VSTO project files.

## Decisions

- Keep the unified manifest for the MVP and validate it after any later manifest edits.
- Use `npm run start:desktop:word` until `app_to_debug` is changed from `excel` to `word`.
- Treat `WordApi 1.1` as the expected baseline for selected-range MVP behavior unless a later implementation chooses higher APIs with explicit runtime guards.
- Guard Word API use with `Office.context.requirements.isSetSupported("WordApi", version)`.
- Guard ribbon-command assumptions with `Office.context.requirements.isSetSupported("AddinCommands", "1.1")` where relevant.
- Exclude `WordApiOnline` from desktop MVP feature requirements.
- Avoid whole-document rewrites and preview/platform-specific APIs unless separately justified and guarded.
- Do not run modernize-dotnet against `.bas`, `.frm`, `.cls`, `.doccls`, `.dotm`, or loose exported `.vb` files.

## Evidence

Commands run:

```powershell
Get-Content -Raw 'Word_Add-in_Agents_and_Skills_Side_Plan.md'
Get-Content -Raw 'Word_Add-in_Plan.md'
Get-Content -Raw 'package.json'
Get-Content -Raw 'appPackage\manifest.json'
Get-Content -Raw 'src\taskpane\word.ts'
Get-Content -Raw 'src\commands\word.ts'
if (Test-Path 'word-addin-modernization\officejs-compatibility.md') { Get-Content -Raw 'word-addin-modernization\officejs-compatibility.md' } else { 'MISSING' }
if (Test-Path 'word-addin-modernization\agent-reports\officejs-compatibility-agent.md') { Get-Content -Raw 'word-addin-modernization\agent-reports\officejs-compatibility-agent.md' } else { 'MISSING' }
Select-String -Path 'package.json','appPackage\manifest.json','Word_Add-in_Plan.md','Word_Add-in_Agents_and_Skills_Side_Plan.md','src\taskpane\word.ts','src\commands\word.ts' -Pattern 'WordApiOnline|modernize-dotnet|start:desktop:word|app_to_debug|manifest|FixDoubleSpaces|RemoveHighlighting|Hello World|Contoso'
Get-ChildItem -Recurse -File -Include *.sln,*.slnx,*.csproj,*.vbproj,*.vcxproj
```

Outputs summarized:

- Assigned output files were missing before this agent wrote them.
- `package.json` line evidence showed `app_to_debug` set to `excel`, Word-specific debug scripts present, and `validate` mapped to `office-addin-manifest validate appPackage/manifest.json`.
- `appPackage\manifest.json` evidence showed `manifestVersion: devPreview`, placeholder `Contoso` values, and multi-host scaffold content.
- `src\taskpane\word.ts` and `src\commands\word.ts` evidence showed template `Hello World` paragraph insertion.
- No `.sln`, `.slnx`, `.csproj`, `.vbproj`, or `.vcxproj` files were found by the workspace scan.

Microsoft Learn documentation consulted:

- Office Add-ins manifest
- Requirements property in the unified manifest for Microsoft 365
- Sideload Office Add-ins that use the unified manifest for Microsoft 365
- Validate an Office Add-in's manifest
- Office versions and requirement sets
- Word JavaScript API requirement sets
- Word JavaScript API requirement set 1.1
- Word JavaScript API online-only requirement set
- Add-in commands requirement sets
- Word.Font class
- Word.Range class

## Risks

- The actual installed Word desktop version/build has not been recorded in this compatibility pass.
- `npm install`, `npm run build`, `npm run validate`, and `npm run start:desktop:word` were not run because this agent was assigned documentation output only and was told not to modify code or plan files.
- Unified manifest deployment may be a risk for older or unsupported Office clients; validate against the actual target Word build before relying on it.
- Remove-highlighting and document-wide cleanup can damage formatting if implemented as broad text reconstruction instead of targeted range operations.
- Future agents may accidentally treat `WordApiOnline` as a desktop-compatible feature set unless the runtime/manifest boundary is preserved.

## Next handoff

Task Pane And Manifest Agent:

- Change `package.json` default debug host to Word.
- Remove or disable non-Word scaffold surfaces only with immediate manifest validation.
- Replace placeholder branding and template `Hello World` behavior.
- Validate with `npm run validate` and debug with `npm run start:desktop:word`.

Office.js Implementation Agent:

- Implement pure cleanup rules before Word adapters.
- Start with selected-range operations.
- Add runtime checks before Word API calls.
- Avoid document-wide cleanup until fixture evidence exists.

Document Fidelity And Verification Agent:

- Record actual Word version/build and Office channel.
- Verify selected-range behavior on `.docx` fixtures containing styles, links, fields, comments, bookmarks, tables, and tracked changes.
