# Office.js Compatibility Notes

Generated: 2026-04-27

Owner: Office Add-in Compatibility Agent

## Scope

This note covers the existing Word add-in scaffold in `K:\IdeaProjects\TextCleaner`, the current unified manifest, MVP text-cleaning compatibility, runtime requirement checks, APIs to avoid, WordApiOnline boundaries, the modernize-dotnet boundary, and validation/debug commands.

Files inspected:

- `K:\IdeaProjects\TextCleaner\Word_Add-in_Agents_and_Skills_Side_Plan.md`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`
- `K:\IdeaProjects\TextCleaner\package.json`
- `K:\IdeaProjects\TextCleaner\appPackage\manifest.json`
- `K:\IdeaProjects\TextCleaner\src\taskpane\word.ts`
- `K:\IdeaProjects\TextCleaner\src\commands\word.ts`

## Existing Scaffold

The repository already contains a Microsoft 365 Agents Toolkit-style Office add-in scaffold. The modernization should repair this scaffold instead of creating a second add-in project.

Current local facts:

- `package.json` uses Node `18 || 20 || 22`.
- `package.json` has `config.app_to_debug` set to `excel`, even though Word scripts already exist.
- `npm run start:desktop:word` maps to `office-addin-debugging start appPackage/manifest.json desktop --app word`.
- `npm run validate` maps to `office-addin-manifest validate appPackage/manifest.json`.
- `src\taskpane\word.ts` and `src\commands\word.ts` still insert a blue `Hello World` paragraph.
- `appPackage\manifest.json` still includes placeholder developer and ribbon labels such as `Contoso Add-in`.
- The manifest currently contains scopes for `mail`, `workbook`, `document`, and `presentation`, so it is still a multi-host scaffold rather than a Word-only private add-in.

Compatibility decision:

- Keep the existing scaffold for the MVP.
- Make Word the default debug host in a later Task Pane/Manifest Agent change.
- Remove non-Word host surfaces only when the manifest can be validated immediately afterward.
- Keep the first ribbon command safe: open the TextCleaner pane. Do not add a one-click destructive cleanup ribbon action until selected-range behavior is verified in Word.

## Manifest Type

`appPackage\manifest.json` is a unified Microsoft 365 manifest, not a classic XML add-in-only manifest.

Microsoft documents two Office add-in manifest types: the XML add-in-only manifest and the JSON unified manifest for Microsoft 365. The unified manifest can package Office add-ins with other Microsoft 365 app extensions, while the XML add-in-only manifest is Office-add-in-specific.

Decision for the MVP:

- Keep the unified manifest because this project was generated around it and the local scripts already validate and debug that path.
- Do not convert to XML only to begin the MVP.
- Reconsider a parallel XML manifest only if deployment must support clients that cannot directly install unified manifests.

Unified manifest support implications:

- Microsoft documents unified-manifest sideload support for Office on Windows Version 2304 Build 16320.20000 or later, and for Excel, PowerPoint, and Word on Mac Version 16.103 or later.
- For a desktop Word MVP on Windows, record the actual Word build before claiming support.
- If the target user is on older Windows Office, perpetual Office, Mac versions below the current unified-manifest sideload support, or iPad, treat unified-manifest deployment as a compatibility risk and evaluate an XML add-in-only manifest or admin deployment path.

## Word Debug Setup Implications

The default debug target is currently Excel. That conflicts with a Word-only modernization because `npm run start` and `npm run start:desktop` will follow `config.app_to_debug`.

Required later scaffold repair:

```json
"config": {
  "app_to_debug": "word",
  "app_type_to_debug": "desktop",
  "dev_server_port": 3000
}
```

Until that change is made, use the explicit Word script:

```powershell
npm run start:desktop:word
```

Expected implication:

- `npm run start:desktop:word` is the reliable manual command for this compatibility phase.
- `npm run start` and `npm run start:desktop` should not be treated as Word validation until `app_to_debug` is changed to `word`.

## MVP Feature Compatibility

Baseline rule: start with selected-range cleanup and small targeted replacements. Avoid whole-document rewrites until fixture documents prove formatting preservation.

| MVP feature | Office.js fit | Minimum expected API set | Compatibility decision |
|---|---|---:|---|
| Remove repeated spaces | Good for pure string rule. Word adapter should operate on selected range text or targeted search results. | `WordApi 1.1` for `Document.getSelection`, `Range.text`, `Range.insertText`, and search surfaces. | Port to Office.js; start selected-range only. |
| Remove repeated blank paragraphs | Feasible but riskier than simple text replacement because paragraph boundaries and styles matter. | `WordApi 1.1` for paragraph/range basics. Higher APIs only if implementation requires table/content-control-specific behavior. | Port only after fixture behavior is defined. |
| Normalize nonbreaking spaces | Good fit as pure string rule and selected-range replacement. | `WordApi 1.1`. | Port to Office.js; selected-range first. |
| Remove optional hyphens | Good fit as pure string rule and targeted replacement. | `WordApi 1.1`. | Port to Office.js; selected-range first. |
| Fix spaces around dashes | Good fit as pure string rule after exact VBA behavior is captured. | `WordApi 1.1`. | Port after before/after examples exist. |
| Remove highlighting | Feasible through `Range.font.highlightColor = null` or range highlight APIs, but must be fixture-tested in selected ranges and mixed-format text. | `WordApi 1.1` for `Font.highlightColor`. | Port cautiously; do not combine with broad document cleanup at first. |
| Basic cleanup preset | Feasible if it composes only already-tested rules. | Highest requirement of included rules. | Add after individual selected-range commands pass. |

Do not accept a feature as MVP merely because a TypeScript pure rule works. Office.js compatibility requires proof that the Word adapter preserves document structure, especially in selections containing styles, hyperlinks, fields, comments, bookmarks, tables, and tracked changes.

## Runtime Requirement Checks

Office add-ins can specify requirement sets in the manifest and can check support at runtime with `Office.context.requirements.isSetSupported`.

Required checks for this project:

```typescript
function isWordHost(info: Office.InitializationReason | Office.OfficeReadyInfo): boolean {
  return "host" in info && info.host === Office.HostType.Word;
}

function requireWordApi(version: string): boolean {
  return Office.context.requirements.isSetSupported("WordApi", version);
}

function requireAddinCommands(version = "1.1"): boolean {
  return Office.context.requirements.isSetSupported("AddinCommands", version);
}

function requireWordApiOnline(): boolean {
  return Office.context.requirements.isSetSupported("WordApiOnline", "1.1");
}
```

Implementation guidance:

- Gate all Word document mutation behind `info.host === Office.HostType.Word`.
- Check `WordApi 1.1` before calling MVP Word APIs.
- Check `AddinCommands 1.1` before relying on behavior introduced by that requirement set, especially command/task pane behavior.
- If a future feature requires `WordApi 1.3`, `WordApi 1.5`, or later, add a separate runtime guard and a disabled UI state with a clear message.
- Do not over-constrain the manifest with high WordApi requirements for optional features; use runtime guards so the add-in can still load and expose supported features.

## WordApiOnline Boundary

Microsoft defines `WordApiOnline` as a special requirement set for APIs that are only available for Word on the web. Microsoft also states that `WordApiOnline 1.1` is the only online-only version and that it cannot be specified as a manifest activation requirement.

Compatibility decision:

- `WordApiOnline` can be used only for optional Word on the web enhancements after a runtime check:

```typescript
if (Office.context.requirements.isSetSupported("WordApiOnline", "1.1")) {
  // Optional Word on the web-only feature.
}
```

- `WordApiOnline` must not be a desktop MVP dependency.
- `WordApiOnline` must not be added as an activation requirement in the manifest.
- Any feature that requires `WordApiOnline` belongs outside the desktop Word MVP and should be documented as web-only or deferred.

Why this matters:

- The current MVP target is Word desktop debug and private Word use.
- A desktop MVP that depends on `WordApiOnline` would fail on the main target surface.
- Manifest activation cannot protect the desktop target from `WordApiOnline` calls because the requirement set is not valid in manifest activation requirements.

## APIs And Features To Avoid

Avoid these in the MVP:

- Preview APIs and beta Office.js CDN references for production behavior.
- `WordApiOnline` APIs for desktop MVP features.
- `WordApiDesktop`, `WordApiHiddenDocument`, or other platform-specific sets unless guarded and explicitly justified.
- Whole-body destructive patterns such as `context.document.body.clear()` followed by inserting reconstructed text.
- Full OOXML replacement for normal text cleanup.
- Office APIs that move the user selection as the main editing mechanism when a stable range/search result can be used instead.
- VBA/COM concepts that do not map to Office.js: `CommandBars`, `Application.TaskPanes`, `VBProject`, `Application.Run`, `SendKeys`, registry `SaveSetting`/`GetSetting`, `PrivateProfileString`, native `Declare` calls, template/startup folder manipulation, and broad `Documents.Open` batch workflows.
- Macro installer/uninstaller behavior, VBA project self-modification, trust-center manipulation, and global keybindings.

Use instead:

- Pure TypeScript cleanup rules for deterministic text transforms.
- Selected-range Word adapters first.
- Targeted `Range.search` and range-level replacement where possible.
- Explicit user confirmation before document-wide commands.
- Add-in/task pane storage instead of registry or INI files.
- Manifest ribbon commands instead of legacy `CommandBars`.

## Validation And Debug Commands

Use these commands from `K:\IdeaProjects\TextCleaner`:

```powershell
npm install
npm run build
npm run validate
npm run validate -- -p
npm run start:desktop:word
npm run stop
```

Command meanings:

- `npm install`: restore scaffold dependencies before validation or debug.
- `npm run build`: verify webpack/TypeScript production build.
- `npm run validate`: validate `appPackage/manifest.json`; Microsoft documents this command for projects generated by Agents Toolkit or Yo Office.
- `npm run validate -- -p`: run production-level Microsoft 365 and Copilot store validation; this is stricter and should be used before deployment decisions.
- `npm run start:desktop:word`: start the dev server and sideload into Word desktop explicitly.
- `npm run stop`: stop the Office add-in debugging session.

Manual evidence to record after debug:

- Word version and build.
- Office channel.
- Windows version.
- Whether the add-in loads from the unified manifest.
- Whether the TextCleaner pane opens from the Word ribbon.
- Whether selected-range MVP commands preserve formatting in fixture documents.

## modernize-dotnet Boundary

modernize-dotnet is not applicable to the current VBA exports.

Current boundary:

- The current source material consists of exported VBA assets such as `.bas`, `.frm`, `.frx`, `.cls`, `.doccls`, `.dotm`, and combined `.vb` exports.
- No `.sln`, `.slnx`, `.csproj`, `.vbproj`, or `.vcxproj` files were found in the current workspace scan.
- Exported VBA modules are legacy source material for a port, not valid project inputs for .NET modernization assessment.

Future applicable use:

- modernize-dotnet becomes relevant only if the project later creates or imports a real .NET/VSTO project, such as:

```text
K:\IdeaProjects\TextCleaner\word-vsto-addin\TextCleanerVsto.sln
K:\IdeaProjects\TextCleaner\word-vsto-addin\TextCleanerVsto\TextCleanerVsto.csproj
K:\IdeaProjects\TextCleaner\word-vsto-addin\TextCleanerVsto\TextCleanerVsto.vbproj
```

Decision:

- Do not run modernize-dotnet tools against the current VBA exports.
- Do not create a VSTO project as part of this compatibility note.
- Revisit modernize-dotnet only after a separate VSTO fallback decision approves real .NET project files.

## Sources

Microsoft Learn sources used:

- [Office Add-ins manifest](https://learn.microsoft.com/en-us/office/dev/add-ins/develop/add-in-manifests)
- [How to use the requirements property in the unified manifest for Microsoft 365](https://learn.microsoft.com/en-us/office/dev/add-ins/develop/requirements-property-unified-manifest)
- [Sideload Office Add-ins that use the unified manifest for Microsoft 365](https://learn.microsoft.com/en-us/office/dev/add-ins/testing/sideload-add-in-with-unified-manifest)
- [Validate an Office Add-in's manifest](https://learn.microsoft.com/en-us/office/dev/add-ins/testing/troubleshoot-manifest)
- [Office versions and requirement sets](https://learn.microsoft.com/en-us/office/dev/add-ins/develop/office-versions-and-requirement-sets)
- [Word JavaScript API requirement sets](https://learn.microsoft.com/en-us/javascript/api/requirement-sets/word/word-api-requirement-sets)
- [Word JavaScript API requirement set 1.1](https://learn.microsoft.com/en-us/javascript/api/requirement-sets/word/word-api-1-1-requirement-set)
- [Word JavaScript API online-only requirement set](https://learn.microsoft.com/en-us/javascript/api/requirement-sets/word/word-api-online-requirement-set)
- [Add-in commands requirement sets](https://learn.microsoft.com/en-us/javascript/api/requirement-sets/common/add-in-commands-requirement-sets)
- [Word.Font class](https://learn.microsoft.com/en-us/javascript/api/word/word.font)
- [Word.Range class](https://learn.microsoft.com/en-us/javascript/api/word/word.range)
