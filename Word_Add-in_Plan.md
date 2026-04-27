# Word Add-in Modernization Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` or `superpowers:executing-plans` if this plan is later implemented task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Modernize the Word VBA macro corpus in `K:\IdeaProjects\TextCleaner\vb-to-c` into a private Word add-in, while preserving useful text-cleaning behavior and avoiding unsupported Office APIs.

**Architecture:** Treat the exported VBA as legacy source material, not as a directly upgradeable VB.NET project. Reuse the existing Office add-in scaffold at `K:\IdeaProjects\TextCleaner` as the primary implementation target, port deterministic cleanup rules into testable TypeScript modules, and isolate Word-specific Office.js code behind adapters. Use VSTO only as a separately documented Windows desktop fallback for features that Office.js cannot support.

**Tech Stack:** Existing Microsoft 365 Agents Toolkit Office add-in scaffold, unified manifest in `appPackage\manifest.json`, Office.js Word JavaScript API, TypeScript, Node.js 18/20/22, webpack, Office add-in validation/debugging tools, optional VSTO/.NET Framework fallback.

---

## Six-Cycle Refinement Record

This version reflects six explicit refinement cycles over the original plan.

| Cycle | Main issue found | Change made in this plan |
|---|---|---|
| 1 | The original plan treated the add-in as not yet scaffolded. | Re-centered the plan on the existing root Office add-in scaffold: `package.json`, `src`, `appPackage\manifest.json`, and `m365agents.yml`. |
| 2 | The original plan was not strict enough about source provenance and duplicate template exports. | Added source-canonicalization gates, duplicate checks, and migration logs before any porting. |
| 3 | Compatibility risk was listed but not converted into a decision process. | Added an Office.js feasibility matrix with explicit decisions: port, redesign, VSTO-only, defer, or drop. |
| 4 | The implementation plan allowed formatting-destroying whole-document rewrites too early. | Added a range-preservation strategy, fixture documents, before/after tests, and explicit destructive-operation guardrails. |
| 5 | Private deployment, security, and macro-trust implications needed clearer handling. | Added deployment choices, manifest validation, HTTPS hosting, storage rules, and macro/VBA security boundaries. |
| 6 | The original sequence lacked a disciplined execution order and acceptance gates. | Added phase gates, exact commands, expected outcomes, and a completion checklist. |

## Current Workspace Findings

- Workspace root: `K:\IdeaProjects\TextCleaner`.
- Existing Office add-in scaffold is already present at the workspace root.
- Existing scaffold files include:
  - `K:\IdeaProjects\TextCleaner\package.json`
  - `K:\IdeaProjects\TextCleaner\appPackage\manifest.json`
  - `K:\IdeaProjects\TextCleaner\m365agents.yml`
  - `K:\IdeaProjects\TextCleaner\src\taskpane\word.ts`
  - `K:\IdeaProjects\TextCleaner\src\commands\word.ts`
- `package.json` currently has `config.app_to_debug` set to `excel`, but Word scripts already exist:
  - `npm run start:desktop:word`
  - `npm run start:web`
  - `npm run validate`
- `appPackage\manifest.json` is a unified Microsoft 365 manifest, not a classic XML add-in-only manifest.
- The manifest and scaffold still contain generic placeholder branding such as `Contoso` and `Full name for TextCleaner`.
- The workspace root is not currently a Git repository, so use explicit backups or file copies before large source transformations.
- The modernize-dotnet MCP namespace is available in this session, but its exposed tools target .NET project upgrade scenarios requiring `.sln`, `.slnx`, `.csproj`, `.vbproj`, or `.vcxproj` inputs.
- No `.sln`, `.slnx`, `.vbproj`, or `.csproj` files were found under `K:\IdeaProjects\TextCleaner\vb-to-c`, so `generate_dotnet_upgrade_assessment` is not applicable to the current VBA corpus.

## VBA Source Inventory

Source inspected: `K:\IdeaProjects\TextCleaner\vb-to-c`.

Observed file inventory:

| Extension | Count | Meaning |
|---|---:|---|
| `.bas` | 566 | VBA standard modules |
| `.frm` | 136 | VBA UserForm text exports |
| `.frx` | 136 | VBA UserForm binary assets |
| `.cls` | 36 | VBA class modules |
| `.vb` | 10 | Large combined exported code files |
| `.doccls` | 3 | Document/class exports |
| `.dotm` | 2 | Macro-enabled Word templates |

Main groups:

| Path | Files | Role |
|---|---:|---|
| `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported` | 349 | Editors Toolkit Plus exported modules/forms |
| `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported - Copy` | 349 | Likely duplicate export, must be hash-checked before use |
| `K:\IdeaProjects\TextCleaner\vb-to-c\TransTools_dotm_Exported` | 178 | TransTools exported modules/classes/forms |
| `K:\IdeaProjects\TextCleaner\vb-to-c\NewETKPlus\NewETKPlus.dotm` | 1 | Macro template |
| `K:\IdeaProjects\TextCleaner\vb-to-c\NewTransTools\NewTransTools.dotm` | 1 | Macro template |

Compatibility signal counts from the scan:

| Signal | Matches | Files | Migration meaning |
|---|---:|---:|---|
| `Selection.` | 8619 | 268 | Must be replaced with explicit ranges/search results where possible. |
| `ActiveDocument` | 4513 | 288 | Must be mapped to `context.document` or explicit user-selected documents. |
| `Application.Run` | 736 | 110 | Macro dispatch chains need call graph mapping before porting. |
| `SaveSetting` | 2396 | 185 | Registry/profile settings must move to add-in storage. |
| `GetSetting` | 4168 | 200 | Registry/profile settings must move to add-in storage. |
| `PrivateProfileString` | 427 | 11 | INI/settings behavior must be replaced or dropped. |
| `Declare` | 332 | 18 | Native API calls are not Office.js-compatible. |
| `PtrSafe` | 162 | 12 | Existing 64-bit VBA awareness does not help Office.js; relevant only to retained VBA/VSTO. |
| `LongPtr` | 492 | 43 | Same as above. |
| `CommandBars` | 69 | 10 | Replace with manifest ribbon commands or task pane UI. |
| `Application.TaskPanes` | 124 | 10 | Replace with Office task pane. |
| `MacScript` | 24 | 8 | Legacy Mac automation path; do not port directly. |
| `AppleScript` | 16 | 4 | Legacy Mac automation path; do not port directly. |
| `VBProject` | 26 | 10 | Remove; modern add-in should not automate VBA project access. |
| `UndoRecord` | 267 | 39 | No direct Office.js equivalent; design command-level confirmations and backups. |

## Modernization Position

The code can be modernized, but this is a port and product redesign, not a mechanical language upgrade.

- Do not try to convert all VBA at once.
- Do not run .NET upgrade assessment on exported `.bas`, `.frm`, `.cls`, `.doccls`, `.dotm`, or combined `.vb` export files.
- Do not preserve macro installer/uninstaller behavior.
- Do not preserve VBA project self-modification.
- Do not rely on `Selection`-style automation as the new internal design.
- Do port the smallest deterministic text-cleaning features first.
- Do keep a compatibility matrix so every macro has a decision.
- Do test against real Word, not only pure TypeScript tests.

## Recommended Target Structure

Use the current root project instead of creating `word-addin\` unless the scaffold becomes too noisy to repair.

```text
K:\IdeaProjects\TextCleaner\
  appPackage\
    manifest.json
  src\
    commands\
      commands.ts
      word.ts
    core\
      cleanupRules.ts
      cleanupTypes.ts
      protectedRanges.ts
      settings.ts
      telemetry.ts
      wordAdapters.ts
    taskpane\
      taskpane.html
      taskpane.ts
      taskpane.css
      word.ts
    tests\
      cleanupRules.test.ts
      fixtures\
  word-addin-modernization\
    source-inventory.md
    compatibility-matrix.md
    mvp-scope.md
    migration-log.md
    compatibility-results.md
    private-deployment.md
    vsto-decision.md
```

Responsibilities:

- `src\core\cleanupRules.ts`: pure, deterministic text transformations.
- `src\core\cleanupTypes.ts`: shared command IDs, option types, result types, and severity enums.
- `src\core\protectedRanges.ts`: rules for skipping protected text categories such as fields, hyperlinks, code-like regions, or user-selected exclusions.
- `src\core\settings.ts`: add-in storage abstraction over Office/add-in storage.
- `src\core\telemetry.ts`: local-only diagnostic counters and error summaries; no external telemetry unless explicitly added later.
- `src\core\wordAdapters.ts`: Office.js document adapters that preserve formatting where possible.
- `src\taskpane\word.ts`: Word task pane behavior.
- `src\commands\word.ts`: Word ribbon command handlers.
- `word-addin-modernization\*.md`: audit and decision artifacts.

## Compatibility Decision Rules

Use this decision model for every macro/module.

| Decision | Use when | Target |
|---|---|---|
| `Port to Office.js` | The behavior can be expressed through Word JavaScript APIs and can be tested without destroying formatting. | TypeScript core plus Office.js adapter |
| `Redesign UI` | Behavior is valid but old UI is UserForm, CommandBars, keybindings, or task pane manipulation. | Task pane and manifest commands |
| `VSTO-only` | Behavior requires desktop Word COM APIs, native Windows APIs, global keybindings, or file-system automation unavailable to Office.js. | Separate `word-vsto-addin` project only if needed |
| `Defer` | Useful but too large or ambiguous for the MVP. | Backlog with source mapping |
| `Drop` | Installer/uninstaller, registration, macro self-modification, abandoned legacy behavior, or behavior unsafe for private add-in use. | Document rationale only |

High-risk triggers:

- `CommandBars`
- `Application.TaskPanes`
- `VBProject`
- `OrganizerCopy`
- `SendKeys`
- `Declare`
- `PrivateProfileString`
- `SaveSetting` or `GetSetting`
- registry read/write
- `MacScript`
- `AppleScript`
- `Documents.Open` batch workflows
- heavy `Selection.` cursor movement
- any macro that changes templates, startup folders, or trust settings

## MVP Scope

Start with features likely to be deterministic and Office.js-feasible:

| Feature | Candidate VBA source | First target |
|---|---|---|
| Remove repeated spaces | `vb-to-c\ETKPlus_Exported\FixDoubleSpaces.bas` | Pure text rule plus range-preserving Word adapter |
| Remove repeated blank paragraphs | `vb-to-c\ETKPlus_Exported\FixDoubleReturns.bas` | Paragraph/range adapter, not whole-body rewrite |
| Normalize nonbreaking spaces | `vb-to-c\ETKPlus_Exported\RemoveNonbreakingSpaces.bas` and TransTools equivalents | Pure text rule plus selected range support |
| Remove optional hyphens | `vb-to-c\ETKPlus_Exported\RemoveOptionalHyphens.bas` | Search/replace adapter |
| Fix spaces around dashes | `vb-to-c\ETKPlus_Exported\FixSpacesAroundDashes.bas` | Pure text rule with before/after fixtures |
| Remove highlighting | `vb-to-c\ETKPlus_Exported\RemoveHighlighting.bas` and TransTools equivalents | Office.js formatting adapter if API support is sufficient |
| Basic cleanup preset | Derived from selected low-risk rules | Task pane button and ribbon command |

Exclude from MVP:

- complex UserForms
- registration/licensing code
- `.dotm` installer/uninstaller macros
- global keybinding macros
- VBA project manipulation
- CommandBars recreation
- batch processing across unopened files
- native Windows API behavior
- MacScript/AppleScript automation
- anything that relies on exact Word cursor movement

## Phase 0: Stabilize The Workspace

**Files:**
- Read: `K:\IdeaProjects\TextCleaner\package.json`
- Read: `K:\IdeaProjects\TextCleaner\appPackage\manifest.json`
- Read: `K:\IdeaProjects\TextCleaner\src\taskpane\word.ts`
- Read: `K:\IdeaProjects\TextCleaner\src\commands\word.ts`
- Create: `K:\IdeaProjects\TextCleaner\word-addin-modernization\source-inventory.md`

- [ ] **Step 0.1: Create the modernization artifact folder**

Run:

```powershell
New-Item -ItemType Directory -Force -Path "K:\IdeaProjects\TextCleaner\word-addin-modernization" | Out-Null
```

Expected: `K:\IdeaProjects\TextCleaner\word-addin-modernization` exists.

- [ ] **Step 0.2: Record the current scaffold**

Run:

```powershell
@(
  "# Existing Add-in Scaffold"
  ""
  "Generated: $(Get-Date -Format s)"
  ""
  "## package.json scripts"
  ""
  (Get-Content -LiteralPath "K:\IdeaProjects\TextCleaner\package.json" | Out-String)
) | Set-Content -LiteralPath "K:\IdeaProjects\TextCleaner\word-addin-modernization\source-inventory.md" -Encoding UTF8
```

Expected: `source-inventory.md` records the current scaffold before edits.

- [ ] **Step 0.3: Verify the scaffold before modernization**

Run:

```powershell
cd "K:\IdeaProjects\TextCleaner"
npm install
npm run build
npm run validate
```

Expected:

- dependencies install successfully
- webpack build succeeds
- manifest validation succeeds

If validation fails, document the exact failure in `word-addin-modernization\source-inventory.md` before changing code.

## Phase 1: Canonicalize The VBA Source

**Files:**
- Read: `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported`
- Read: `K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported - Copy`
- Read: `K:\IdeaProjects\TextCleaner\vb-to-c\TransTools_dotm_Exported`
- Modify: `K:\IdeaProjects\TextCleaner\word-addin-modernization\source-inventory.md`

- [ ] **Step 1.1: Generate file type counts**

Run:

```powershell
$root = "K:\IdeaProjects\TextCleaner\vb-to-c"
$out = "K:\IdeaProjects\TextCleaner\word-addin-modernization\source-inventory.md"
"`n## VBA source file counts`n" | Add-Content -LiteralPath $out -Encoding UTF8
Get-ChildItem -LiteralPath $root -Recurse -File |
  Group-Object Extension |
  Sort-Object Count -Descending |
  ForEach-Object { "- $($_.Name): $($_.Count)" } |
  Add-Content -LiteralPath $out -Encoding UTF8
```

Expected: the counts match the current workspace inventory.

- [ ] **Step 1.2: Compare likely duplicate ETKPlus exports**

Run:

```powershell
$a = "K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported"
$b = "K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported - Copy"
$out = "K:\IdeaProjects\TextCleaner\word-addin-modernization\source-inventory.md"
"`n## ETKPlus duplicate hash spot checks`n" | Add-Content -LiteralPath $out -Encoding UTF8
"FileCleanerLibrary.bas" | ForEach-Object {
  $fa = Join-Path $a $_
  $fb = Join-Path $b $_
  $ha = (Get-FileHash -Algorithm SHA256 -LiteralPath $fa).Hash
  $hb = (Get-FileHash -Algorithm SHA256 -LiteralPath $fb).Hash
  "- $_: source=$ha copy=$hb match=$($ha -eq $hb)" | Add-Content -LiteralPath $out -Encoding UTF8
}
```

Expected: matching hashes mean the copy can be ignored for MVP source selection. Mismatches require a file-level diff and a canonical-source decision.

- [ ] **Step 1.3: Choose canonical source roots**

Record one of these decisions in `source-inventory.md`:

- Use `ETKPlus_Exported` and ignore `ETKPlus_Exported - Copy`.
- Use `ETKPlus_Exported - Copy` if it contains newer unique changes.
- Use both only after listing which files differ and why each file is needed.

Expected: no macro is ported from an untracked duplicate source.

## Phase 2: Build The Compatibility Matrix

**Files:**
- Create: `K:\IdeaProjects\TextCleaner\word-addin-modernization\compatibility-matrix.md`

- [ ] **Step 2.1: Create the matrix shell**

Run:

```powershell
$out = "K:\IdeaProjects\TextCleaner\word-addin-modernization\compatibility-matrix.md"
@"
# Compatibility Matrix

| Source file | Macro/Class/Form | Feature area | Signals | Office.js fit | Decision | Risk | Notes |
|---|---|---|---|---|---|---|---|
"@ | Set-Content -LiteralPath $out -Encoding UTF8
```

Expected: `compatibility-matrix.md` exists with a table header.

- [ ] **Step 2.2: Seed the first source files**

Start with this ordered seed set:

```text
K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\FixDoubleSpaces.bas
K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\FixDoubleReturns.bas
K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\RemoveNonbreakingSpaces.bas
K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\RemoveOptionalHyphens.bas
K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\FixSpacesAroundDashes.bas
K:\IdeaProjects\TextCleaner\vb-to-c\ETKPlus_Exported\RemoveHighlighting.bas
K:\IdeaProjects\TextCleaner\vb-to-c\TransTools_dotm_Exported\DocCleaner.bas
K:\IdeaProjects\TextCleaner\vb-to-c\TransTools_dotm_Exported\modDocCleaner.bas
```

Expected: the first MVP candidates are classified before broader exploration.

- [ ] **Step 2.3: Record high-risk signals**

Run:

```powershell
$files = Get-ChildItem -LiteralPath "K:\IdeaProjects\TextCleaner\vb-to-c" -Recurse -File -Include *.bas,*.cls,*.frm,*.vb,*.doccls
$patterns = @(
  "CommandBars",
  "Application.TaskPanes",
  "VBProject",
  "OrganizerCopy",
  "SendKeys",
  "Declare ",
  "PrivateProfileString",
  "SaveSetting",
  "GetSetting",
  "MacScript",
  "AppleScript",
  "Documents.Open",
  "Selection\."
)
$out = "K:\IdeaProjects\TextCleaner\word-addin-modernization\compatibility-matrix.md"
"`n## Signal summary`n" | Add-Content -LiteralPath $out -Encoding UTF8
foreach ($pattern in $patterns) {
  $matches = @($files | Select-String -Pattern $pattern -CaseSensitive:$false)
  "- `$pattern`: $($matches.Count) matches in $(@($matches | Select-Object -ExpandProperty Path -Unique).Count) files" |
    Add-Content -LiteralPath $out -Encoding UTF8
}
```

Expected: the matrix records where Office.js risks concentrate.

## Phase 3: Repair The Existing Add-in Scaffold For Word

**Files:**
- Modify: `K:\IdeaProjects\TextCleaner\package.json`
- Modify: `K:\IdeaProjects\TextCleaner\appPackage\manifest.json`
- Modify: `K:\IdeaProjects\TextCleaner\src\taskpane\taskpane.html`
- Modify: `K:\IdeaProjects\TextCleaner\src\taskpane\taskpane.ts`
- Modify: `K:\IdeaProjects\TextCleaner\src\taskpane\word.ts`
- Modify: `K:\IdeaProjects\TextCleaner\src\commands\commands.ts`
- Modify: `K:\IdeaProjects\TextCleaner\src\commands\word.ts`

- [ ] **Step 3.1: Make Word the default debug host**

Change `package.json`:

```json
"config": {
  "app_to_debug": "word",
  "app_type_to_debug": "desktop",
  "dev_server_port": 3000
}
```

Expected: default debug behavior targets Word instead of Excel.

- [ ] **Step 3.2: Remove non-Word surfaces unless you deliberately want multi-host support**

For a personal Word-only cleaner, remove or disable Excel, PowerPoint, and Outlook command/taskpane routing from the manifest and code after confirming that the unified manifest schema still validates.

Expected:

- `npm run validate` passes.
- `npm run start:desktop:word` starts Word.
- Excel/PowerPoint/Outlook scripts are no longer part of the actual add-in behavior unless deliberately retained.

- [ ] **Step 3.3: Replace placeholder branding**

Update manifest values:

- short name: `TextCleaner`
- full name: `TextCleaner Word Add-in`
- developer name: your private name or local organization name
- description: `Private Word add-in for modernized text-cleaning commands.`
- production URL: replace `https://www.contoso.com/` in `webpack.config.js` before any real deployment.

Expected: no user-facing `Contoso`, `Hello World`, or generic template text remains in Word.

- [ ] **Step 3.4: Add requirement-set checks**

In Word entry points, use runtime checks before calling Word APIs:

```typescript
function requireWordApi(version: string): boolean {
  return Office.context.requirements.isSetSupported("WordApi", version);
}
```

Expected: unsupported Word builds show a clear task pane message instead of failing silently.

## Phase 4: Port Pure Cleanup Rules

**Files:**
- Create: `K:\IdeaProjects\TextCleaner\src\core\cleanupTypes.ts`
- Create: `K:\IdeaProjects\TextCleaner\src\core\cleanupRules.ts`
- Create: `K:\IdeaProjects\TextCleaner\src\tests\cleanupRules.test.ts`
- Create: `K:\IdeaProjects\TextCleaner\word-addin-modernization\migration-log.md`

- [ ] **Step 4.1: Define rule IDs and results**

Create `src\core\cleanupTypes.ts`:

```typescript
export type CleanupRuleId =
  | "normalizeSpaces"
  | "normalizeBlankParagraphs"
  | "normalizeNonbreakingSpaces"
  | "removeOptionalHyphens"
  | "normalizeDashSpacing";

export interface CleanupResult {
  ruleId: CleanupRuleId;
  input: string;
  output: string;
  replacements: number;
}
```

- [ ] **Step 4.2: Implement pure rules first**

Create `src\core\cleanupRules.ts`:

```typescript
import { CleanupResult } from "./cleanupTypes";

function result(ruleId: CleanupResult["ruleId"], input: string, output: string, replacements: number): CleanupResult {
  return { ruleId, input, output, replacements };
}

export function normalizeSpaces(input: string): CleanupResult {
  const output = input.replace(/[ \t]{2,}/g, " ");
  return result("normalizeSpaces", input, output, input === output ? 0 : 1);
}

export function normalizeNonbreakingSpaces(input: string): CleanupResult {
  const output = input.replace(/\u00a0/g, " ");
  const replacements = (input.match(/\u00a0/g) ?? []).length;
  return result("normalizeNonbreakingSpaces", input, output, replacements);
}

export function removeOptionalHyphens(input: string): CleanupResult {
  const output = input.replace(/\u00ad/g, "");
  const replacements = (input.match(/\u00ad/g) ?? []).length;
  return result("removeOptionalHyphens", input, output, replacements);
}
```

This first implementation intentionally covers only safe pure-string rules. Paragraph-aware and dash-spacing rules should be added with fixtures after their exact VBA behavior is documented.

- [ ] **Step 4.3: Add tests before connecting to Word**

Use the test runner selected for the project, such as Jest or Vitest. If no runner exists, add one deliberately in a separate change and record it in `migration-log.md`.

Example test content:

```typescript
import { normalizeNonbreakingSpaces, normalizeSpaces, removeOptionalHyphens } from "../core/cleanupRules";

describe("cleanupRules", () => {
  it("normalizes repeated spaces", () => {
    expect(normalizeSpaces("A  B   C").output).toBe("A B C");
  });

  it("normalizes nonbreaking spaces", () => {
    expect(normalizeNonbreakingSpaces("A\u00a0B").output).toBe("A B");
  });

  it("removes optional hyphens", () => {
    expect(removeOptionalHyphens("hy\u00adphen").output).toBe("hyphen");
  });
});
```

Expected: pure rules have deterministic tests before any document mutation.

- [ ] **Step 4.4: Log source traceability**

Create `word-addin-modernization\migration-log.md`:

```markdown
# Migration Log

| New rule | Source VBA | Behavior covered | Known differences | Test file |
|---|---|---|---|---|
| `normalizeSpaces` | `vb-to-c\ETKPlus_Exported\FixDoubleSpaces.bas` | Collapses repeated spaces in plain text | Protected ranges not implemented in first pass | `src\tests\cleanupRules.test.ts` |
| `normalizeNonbreakingSpaces` | `vb-to-c\ETKPlus_Exported\RemoveNonbreakingSpaces.bas` | Converts U+00A0 to normal spaces | Does not yet distinguish intentional nonbreaking spaces | `src\tests\cleanupRules.test.ts` |
| `removeOptionalHyphens` | `vb-to-c\ETKPlus_Exported\RemoveOptionalHyphens.bas` | Removes U+00AD | None for plain text | `src\tests\cleanupRules.test.ts` |
```

Expected: every new rule maps to source and tests.

## Phase 5: Connect Rules To Word Without Destroying Formatting

**Files:**
- Create: `K:\IdeaProjects\TextCleaner\src\core\wordAdapters.ts`
- Modify: `K:\IdeaProjects\TextCleaner\src\taskpane\word.ts`
- Modify: `K:\IdeaProjects\TextCleaner\src\commands\word.ts`

- [ ] **Step 5.1: Prefer targeted range edits**

Use Office.js search/range APIs where possible. Avoid this prototype pattern in production:

```typescript
body.clear();
body.insertText(cleaned, Word.InsertLocation.start);
```

Reason: whole-body rewrites can destroy formatting, fields, comments, bookmarks, revisions, tables, and cross-references.

- [ ] **Step 5.2: Implement selected-range first**

First adapter target:

- if user has selected text, clean only that selection
- if no selection, ask the user to choose selected text or run a document-wide command with confirmation

Expected: early tests cannot accidentally rewrite the whole document.

- [ ] **Step 5.3: Add document-wide cleanup only after fixtures pass**

Document-wide cleanup must pass sample documents containing:

- headings
- character styles
- tables
- footnotes or endnotes
- comments
- track changes
- hyperlinks
- fields
- bookmarks
- text boxes if Office.js access is needed

Expected: no document-wide command ships before fixture evidence is recorded.

## Phase 6: Build The Task Pane And Commands

**Files:**
- Modify: `K:\IdeaProjects\TextCleaner\src\taskpane\taskpane.html`
- Modify: `K:\IdeaProjects\TextCleaner\src\taskpane\taskpane.css`
- Modify: `K:\IdeaProjects\TextCleaner\src\taskpane\word.ts`
- Modify: `K:\IdeaProjects\TextCleaner\src\commands\word.ts`
- Modify: `K:\IdeaProjects\TextCleaner\appPackage\manifest.json`

- [ ] **Step 6.1: Replace template UI**

Task pane controls:

- checkbox: `Normalize repeated spaces`
- checkbox: `Normalize nonbreaking spaces`
- checkbox: `Remove optional hyphens`
- button: `Clean selection`
- button: `Preview selection`
- status area: replacements and warnings

Expected: no `Hello World` or sample insert-paragraph action remains.

- [ ] **Step 6.2: Add a safe ribbon command**

Add one Word command first:

- Label: `Open TextCleaner`
- Action: show task pane

Only add `Run cleanup` as a ribbon command after the task pane selection workflow is stable.

- [ ] **Step 6.3: Make destructive actions explicit**

Before any document-wide cleanup:

- show count/summary when feasible
- state that formatting-sensitive content may be affected
- require explicit user action from the task pane

Expected: accidental broad mutation risk is controlled.

## Phase 7: Test And Validate

**Files:**
- Create: `K:\IdeaProjects\TextCleaner\word-addin-modernization\compatibility-results.md`
- Create: `K:\IdeaProjects\TextCleaner\word-addin-modernization\test-documents`

- [ ] **Step 7.1: Run build and manifest validation**

Run:

```powershell
cd "K:\IdeaProjects\TextCleaner"
npm run build
npm run validate
```

Expected:

- build succeeds
- manifest validates

- [ ] **Step 7.2: Run Word desktop debug**

Run:

```powershell
cd "K:\IdeaProjects\TextCleaner"
npm run start:desktop:word
```

Expected:

- Word opens
- TextCleaner loads
- task pane opens or can be opened
- no Excel/PowerPoint/Outlook host behavior appears unless deliberately retained

- [ ] **Step 7.3: Record actual Word compatibility**

Create `compatibility-results.md`:

```markdown
# Compatibility Results

| Date | Word version/build | Office channel | Windows version | Manifest validation | Desktop sideload | Notes |
|---|---|---|---|---|---|---|
```

Expected: compatibility claims are tied to real Word builds.

- [ ] **Step 7.4: Test against fixture documents**

For each command, record:

```markdown
| Command | Fixture | Result | Formatting preserved | Replacements | Notes |
|---|---|---|---|---:|---|
| Normalize nonbreaking spaces | `nbsp-basic.docx` | Pass | Yes | 4 | Selection-only test. |
```

Expected: no MVP feature is accepted without a fixture result.

## Phase 8: Private Deployment

**Files:**
- Modify: `K:\IdeaProjects\TextCleaner\webpack.config.js`
- Modify: `K:\IdeaProjects\TextCleaner\appPackage\manifest.json`
- Create: `K:\IdeaProjects\TextCleaner\word-addin-modernization\private-deployment.md`

- [ ] **Step 8.1: Choose one private deployment path**

Use one:

- local sideload only on this machine
- private HTTPS internal host
- Azure Static Web Apps from `m365agents.yml`
- Microsoft 365 admin center centralized deployment to only your account, if you administer a tenant

- [ ] **Step 8.2: Replace production placeholder URL**

Change `webpack.config.js`:

```javascript
const urlProd = "https://your-private-textcleaner-host.example/";
```

Expected: production builds do not point to `https://www.contoso.com/`.

- [ ] **Step 8.3: Validate the distributable manifest**

Run:

```powershell
cd "K:\IdeaProjects\TextCleaner"
npm run build
npm run validate
```

Expected: production manifest validates after URL replacement.

## Phase 9: VSTO Fallback Decision

**Files:**
- Create: `K:\IdeaProjects\TextCleaner\word-addin-modernization\vsto-decision.md`

- [ ] **Step 9.1: List Office.js-blocked features**

Move features into `vsto-decision.md` only after the compatibility matrix proves Office.js cannot support them.

- [ ] **Step 9.2: Apply the VSTO decision test**

Use VSTO only if all statements are true:

- The feature is valuable enough to maintain separately.
- You only need Windows desktop Word.
- The feature needs Word COM APIs unavailable in Office.js.
- You accept Visual Studio/VSTO tooling and desktop installation.
- You accept that VSTO is not a cross-platform web add-in path.

- [ ] **Step 9.3: Keep VSTO separate**

If selected, create a separate project:

```text
K:\IdeaProjects\TextCleaner\word-vsto-addin\
```

Expected: Office.js and VSTO do not become a tangled hybrid.

## Phase 10: modernize-dotnet MCP Usage Boundary

**Files:**
- Read: `K:\IdeaProjects\TextCleaner\.github\agents\modernize-dotnet.agent.md`

- [ ] **Step 10.1: Use MCP only for real .NET project inputs**

The current MCP tools are appropriate when there is a solution/project such as:

```text
K:\IdeaProjects\TextCleaner\word-vsto-addin\TextCleanerVsto.sln
K:\IdeaProjects\TextCleaner\word-vsto-addin\TextCleanerVsto\TextCleanerVsto.csproj
```

- [ ] **Step 10.2: Do not call .NET assessment for VBA exports**

Do not call `generate_dotnet_upgrade_assessment` for:

- `.bas`
- `.frm`
- `.frx`
- `.cls`
- `.doccls`
- `.dotm`
- combined exported `.vb` files that are not in a `.vbproj`

- [ ] **Step 10.3: If the plugin must be installed in another session**

The command shape supplied by the user is relevant for plugin-enabled Codex environments:

```text
/plugin marketplace add dotnet/modernize-dotnet
/plugin install modernize-dotnet@modernize-dotnet-plugins
```

Expected: in this current session, the MCP namespace is already available; installation commands are for environments where the plugin is absent.

## Phase Gates

Do not proceed past a gate until its evidence file exists.

| Gate | Required evidence | Blocks |
|---|---|---|
| Source gate | `word-addin-modernization\source-inventory.md` | Any porting |
| Compatibility gate | `word-addin-modernization\compatibility-matrix.md` | MVP implementation |
| Scaffold gate | `npm run build` and `npm run validate` results | Manifest/UI changes |
| Test gate | pure rule tests and fixture records | Document-wide cleanup |
| Deployment gate | `private-deployment.md` and validated production URL | Use outside dev machine |
| VSTO gate | `vsto-decision.md` | Any VSTO project creation |

## Acceptance Criteria

The modernization reaches a credible first milestone when all items are true:

- `npm run build` passes.
- `npm run validate` passes.
- `npm run start:desktop:word` opens Word with TextCleaner.
- The manifest no longer has placeholder branding.
- Word is the default debug host.
- At least three pure cleanup rules have tests.
- At least one cleanup command works on selected text in Word.
- Formatting-preservation behavior is tested on at least one `.docx` fixture.
- Every ported feature has a row in `migration-log.md`.
- Every known blocked feature is in `compatibility-matrix.md` or `vsto-decision.md`.
- No macro installer, VBA project mutation, or registry-based settings behavior is carried into the Office.js MVP.

## Resources

Use current Microsoft documentation as the source of truth before implementing API-specific behavior:

- Office Add-ins overview: https://learn.microsoft.com/en-us/office/dev/add-ins/overview/office-add-ins
- Office Add-ins manifests: https://learn.microsoft.com/en-us/office/dev/add-ins/develop/add-in-manifests
- Office versions and requirement sets: https://learn.microsoft.com/en-us/office/dev/add-ins/develop/office-versions-and-requirement-sets
- Word JavaScript API requirement sets: https://learn.microsoft.com/en-us/javascript/api/requirement-sets/word/word-api-requirement-sets
- Word add-ins programming overview: https://learn.microsoft.com/en-us/office/dev/add-ins/word/word-add-ins-programming-overview
- Validate an Office Add-in manifest: https://learn.microsoft.com/en-us/office/dev/add-ins/testing/troubleshoot-manifest
- Sideload Office Add-ins for testing: https://learn.microsoft.com/en-us/office/dev/add-ins/testing/sideload-office-add-ins-for-testing
- Add-in commands requirement sets: https://learn.microsoft.com/en-us/javascript/api/requirement-sets/common/add-in-commands-requirement-sets
- Word VBA object model reference: https://learn.microsoft.com/en-us/office/vba/api/overview/word
- VSTO Office solutions overview: https://learn.microsoft.com/en-us/visualstudio/vsto/office-solutions-development-overview-vsto
- .NET modernization overview: https://dotnet.microsoft.com/en-us/platform/modernize

## Recommended Next Action

Execute Phase 0 and Phase 1 first. They verify the existing scaffold and establish canonical source provenance. After that, complete the compatibility matrix for the MVP files in Phase 2 before writing any TypeScript cleanup logic.
