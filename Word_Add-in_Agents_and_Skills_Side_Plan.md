# Word Add-in Agents And Skills Side Plan

> **For agentic workers:** Use this side-plan together with `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`. This document assigns specialized agents and skills to the main plan's phases and defines the artifacts each agent must produce.

**Goal:** Use specialized agents and skills to execute the Word add-in modernization efficiently, with strong source traceability, Office.js compatibility discipline, and verification gates.

**Architecture:** Keep one orchestrator responsible for sequencing, scope control, and acceptance. Use focused specialist agents for bounded research, source classification, Office.js implementation, UI/manifest work, verification, and fallback analysis. Skills are applied at the phase level so each agent has the right workflow guardrails before it touches files.

**Tech Stack:** Existing Microsoft 365 Agents Toolkit Office add-in scaffold in `K:\IdeaProjects\TextCleaner`, Office.js Word JavaScript APIs, TypeScript, webpack, `office-addin-*` tooling, optional modernize-dotnet MCP for future real .NET/VSTO projects only.

---

## Research Inputs Used

Local project inputs:

- `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`
- `K:\IdeaProjects\TextCleaner\package.json`
- `K:\IdeaProjects\TextCleaner\appPackage\manifest.json`
- `K:\IdeaProjects\TextCleaner\src\taskpane\word.ts`
- `K:\IdeaProjects\TextCleaner\src\commands\word.ts`
- `K:\IdeaProjects\TextCleaner\.github\agents\modernize-dotnet.agent.md`
- `K:\IdeaProjects\TextCleaner\.github\plugin\marketplace.json`

Current external documentation inputs:

- Office Add-ins manifests: https://learn.microsoft.com/en-us/office/dev/add-ins/develop/add-in-manifests
- Word JavaScript API requirement sets: https://learn.microsoft.com/en-us/javascript/api/requirement-sets/word/word-api-requirement-sets
- Add-in commands requirement sets: https://learn.microsoft.com/en-us/javascript/api/requirement-sets/common/add-in-commands-requirement-sets
- Requirements and Set manifest elements: https://learn.microsoft.com/en-us/javascript/api/manifest/requirements and https://learn.microsoft.com/en-us/javascript/api/manifest/set
- WordApiOnline requirement set: https://learn.microsoft.com/en-us/javascript/api/requirement-sets/word/word-api-online-requirement-set

Key research conclusions:

- The root already contains an Office add-in scaffold and should be repaired before creating a second add-in project.
- The scaffold uses a unified JSON manifest in `appPackage\manifest.json`; the plan must include manifest-specific validation.
- `package.json` defaults to Excel debugging, but has a Word debug script; agent work must correct and verify Word targeting.
- The VBA corpus is not a .NET project. The modernize-dotnet MCP agent is useful only if a real `.sln`/`.csproj`/`.vbproj` project is later created, such as a VSTO fallback.
- Office add-ins must gate behavior by requirement sets. Agents must not assume a Word JavaScript API is available without checking docs and/or `Office.context.requirements.isSetSupported`.
- WordApiOnline APIs are online-only and cannot be specified as activation requirements in the manifest; agents must not build the desktop MVP around them.

## Six-Cycle Iterative Refinement

| Cycle | Research/design question | Finding | Adjustment made |
|---|---|---|---|
| 1 | What agents are actually needed? | The project has multiple separable workstreams: VBA audit, compatibility mapping, Office.js implementation, UI/manifest, testing, deployment, and VSTO fallback. | Defined seven specialist agents plus one orchestrator instead of one broad "modernization" worker. |
| 2 | Which local agents/skills already exist? | Local `.github\agents\modernize-dotnet.agent.md` exists, but it is for .NET projects. No repo-local custom skills were found under `.agents`. | Scoped modernize-dotnet to future VSTO/.NET only and selected available Codex/Superpowers skills for the current Office.js path. |
| 3 | How should agents avoid duplicating work? | The main plan already has phases. Parallel work is safe only when write sets and artifacts are disjoint. | Mapped each agent to explicit owned artifacts and forbidden write areas. |
| 4 | What will prevent Office.js compatibility mistakes? | Requirement sets and platform-specific APIs are central. Unified manifest behavior also matters. | Added a dedicated Office Add-in Compatibility Agent and required citation-backed compatibility decisions before implementation. |
| 5 | What will prevent destructive document edits? | Naive whole-body replacement can destroy fields, comments, formatting, bookmarks, revisions, and tables. | Added a Document Fidelity and Verification Agent with fixture ownership and a rule that document-wide cleanup follows selected-range success. |
| 6 | What makes the agent system efficient rather than bureaucratic? | Too many agents can slow the project if every phase waits on broad reports. | Reduced each agent output to concise decision artifacts, made the orchestrator the only sequencer, and defined stop/go gates instead of timelines. |

## Selected Skills

| Skill | Use | Required for | Why |
|---|---|---|---|
| `superpowers:writing-plans` | Plan creation and refinement | Side-plan and any future task-level plans | Keeps phases explicit, file-specific, and executable. |
| `superpowers:subagent-driven-development` | Coordinated specialist execution | Implementation after this side-plan is approved | Supports fresh bounded agents per task and review checkpoints. |
| `superpowers:executing-plans` | Inline execution fallback | If work is done in one session without multiple agents | Keeps phase execution gated and auditable. |
| `superpowers:test-driven-development` | Rule and adapter implementation | Pure cleanup rules, Word adapters, fixture tests | Prevents undocumented behavior drift from VBA to TypeScript. |
| `superpowers:systematic-debugging` | Build/runtime failures | `npm install`, `npm run build`, `npm run validate`, sideload failures | Forces evidence-first failure analysis. |
| `superpowers:verification-before-completion` | Completion claims | Every phase gate | Ensures build, validation, and test evidence exists before marking work done. |
| `chatgpt-apps` | Not selected for implementation | N/A | It targets ChatGPT Apps SDK/MCP widgets, not Office Word add-ins. Use only if a ChatGPT companion app is later requested. |
| `openai-docs` | Not selected for Office implementation | N/A | It is for OpenAI products. Office add-in docs should come from Microsoft Learn. |
| `modernize-dotnet` MCP agent | Conditional fallback | Future VSTO or helper .NET project with `.sln`/`.csproj`/`.vbproj` | Current VBA exports are not valid inputs for the .NET upgrader. |

## Selected Agents

### 1. Orchestrator Agent

**Purpose:** Own sequencing, scope, gates, and final integration.

**Skills:**

- `superpowers:subagent-driven-development`
- `superpowers:verification-before-completion`
- `superpowers:executing-plans` if running inline

**Owns:**

- `K:\IdeaProjects\TextCleaner\Word_Add-in_Plan.md`
- `K:\IdeaProjects\TextCleaner\Word_Add-in_Agents_and_Skills_Side_Plan.md`
- final phase gate decisions

**Responsibilities:**

- Start each phase only after the previous phase's required evidence exists.
- Assign agents with disjoint write scopes.
- Prevent unapproved broad rewrites.
- Keep the modernize-dotnet boundary clear.
- Integrate reports into the main plan artifacts.

**Must not delegate:**

- Final acceptance decisions.
- Scope expansion.
- Whether VSTO is introduced.
- Whether a blocked Office.js feature is dropped or deferred.

### 2. Source Inventory And Provenance Agent

**Purpose:** Turn the VBA corpus into a reliable source map.

**Skills:**

- `superpowers:executing-plans`
- `superpowers:verification-before-completion`

**Owns:**

- `K:\IdeaProjects\TextCleaner\word-addin-modernization\source-inventory.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\source-hashes.csv`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\source-diff-notes.md`

**Read scope:**

- `K:\IdeaProjects\TextCleaner\vb-to-c`

**Write scope:**

- `K:\IdeaProjects\TextCleaner\word-addin-modernization`

**Design/implementation work:**

- Hash likely duplicate files between `ETKPlus_Exported` and `ETKPlus_Exported - Copy`.
- Identify canonical source roots.
- List all modules/classes/forms by feature area.
- Identify large combined `.vb` exports and decide whether they are redundant with `.bas`/`.cls`/`.frm` exports.
- Produce a "do not port from this copy" list for duplicate or stale files.

**Acceptance evidence:**

- File counts by extension.
- Canonical source decision.
- Duplicate hash table.
- First-pass module category list.

### 3. VBA Behavior Classification Agent

**Purpose:** Translate legacy macro behavior into migration decisions.

**Skills:**

- `superpowers:executing-plans`
- `superpowers:test-driven-development` for extracting behavior examples

**Owns:**

- `K:\IdeaProjects\TextCleaner\word-addin-modernization\compatibility-matrix.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\mvp-scope.md`

**Read scope:**

- canonical files selected by the Source Inventory Agent

**Write scope:**

- `word-addin-modernization\compatibility-matrix.md`
- `word-addin-modernization\mvp-scope.md`

**Design/implementation work:**

- Build the migration table with decisions: `Port to Office.js`, `Redesign UI`, `VSTO-only`, `Defer`, `Drop`.
- Extract before/after examples from MVP macros.
- Classify settings, UI, document mutation, selection use, native API use, and template manipulation.
- Identify macro call chains where `Application.Run` hides real dependencies.

**Acceptance evidence:**

- Every MVP candidate has source path, behavior summary, risk, and decision.
- Every high-risk signal has a rationale.
- MVP scope is small enough to implement and verify.

### 4. Office Add-in Compatibility Agent

**Purpose:** Decide which Office.js APIs and manifest patterns are valid for the target Word add-in.

**Skills:**

- `superpowers:writing-plans`
- `superpowers:verification-before-completion`

**Owns:**

- `K:\IdeaProjects\TextCleaner\word-addin-modernization\officejs-compatibility.md`
- compatibility notes inside `compatibility-matrix.md`

**Read scope:**

- `appPackage\manifest.json`
- `package.json`
- Microsoft Learn Office Add-ins docs

**Write scope:**

- `word-addin-modernization\officejs-compatibility.md`

**Design/implementation work:**

- Identify required WordApi versions for each MVP command.
- Decide whether the unified manifest remains appropriate or whether classic XML manifest would be safer for private use.
- Define runtime checks using `Office.context.requirements.isSetSupported`.
- Flag WordApiOnline-only APIs as non-MVP for desktop Word.
- Define add-in command requirements for ribbon buttons.
- Specify manifest validation commands.

**Acceptance evidence:**

- Each MVP feature has a WordApi requirement decision.
- The manifest path is documented: keep unified manifest, convert to XML, or split later.
- Any online-only API is explicitly excluded from desktop MVP.

### 5. Office.js Implementation Agent

**Purpose:** Implement tested TypeScript cleanup logic and Word adapters.

**Skills:**

- `superpowers:test-driven-development`
- `superpowers:systematic-debugging`
- `superpowers:verification-before-completion`

**Owns:**

- `K:\IdeaProjects\TextCleaner\src\core\cleanupTypes.ts`
- `K:\IdeaProjects\TextCleaner\src\core\cleanupRules.ts`
- `K:\IdeaProjects\TextCleaner\src\core\protectedRanges.ts`
- `K:\IdeaProjects\TextCleaner\src\core\wordAdapters.ts`
- `K:\IdeaProjects\TextCleaner\src\tests\cleanupRules.test.ts`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\migration-log.md`

**Write restrictions:**

- Do not edit `appPackage\manifest.json` unless assigned by the Orchestrator.
- Do not rewrite task pane UI except for explicit function wiring.
- Do not implement whole-document cleanup until selected-range cleanup passes fixture tests.

**Design/implementation work:**

- Implement pure cleanup rules first.
- Add tests before Office.js wiring.
- Implement selected-range cleanup before document-wide cleanup.
- Track replacements and warnings in structured result objects.
- Preserve formatting by using targeted ranges/search where possible.
- Record source-to-TypeScript mapping in `migration-log.md`.

**Acceptance evidence:**

- Pure rule tests pass.
- `npm run build` passes.
- Each implemented rule has a migration-log row.
- Selected-range command works in Word before document-wide commands are added.

### 6. Task Pane And Manifest Agent

**Purpose:** Turn the scaffold into a Word-focused private TextCleaner add-in.

**Skills:**

- `superpowers:test-driven-development` for UI behavior where practical
- `superpowers:verification-before-completion`

**Owns:**

- `K:\IdeaProjects\TextCleaner\package.json`
- `K:\IdeaProjects\TextCleaner\appPackage\manifest.json`
- `K:\IdeaProjects\TextCleaner\webpack.config.js`
- `K:\IdeaProjects\TextCleaner\src\taskpane\taskpane.html`
- `K:\IdeaProjects\TextCleaner\src\taskpane\taskpane.css`
- `K:\IdeaProjects\TextCleaner\src\taskpane\word.ts`
- `K:\IdeaProjects\TextCleaner\src\commands\word.ts`
- `K:\IdeaProjects\TextCleaner\src\commands\commands.ts`

**Write restrictions:**

- Do not alter cleanup core behavior.
- Do not introduce public deployment settings without a private deployment decision.
- Do not remove multi-host scaffold files until the manifest still validates.

**Design/implementation work:**

- Change default debug host from Excel to Word.
- Remove placeholder branding.
- Add task pane controls for selected MVP rules.
- Add status and warning surfaces.
- Add one safe command first: open TextCleaner pane.
- Wire commands to `Office.js` adapter functions after implementation tests pass.
- Keep manifest requirements minimal enough to load on target Word builds.

**Acceptance evidence:**

- `npm run validate` passes.
- `npm run start:desktop:word` opens Word.
- No user-facing template `Hello World` behavior remains.
- Ribbon command opens the TextCleaner pane.

### 7. Document Fidelity And Verification Agent

**Purpose:** Prove that cleanup behavior does not damage real Word documents.

**Skills:**

- `superpowers:systematic-debugging`
- `superpowers:verification-before-completion`

**Owns:**

- `K:\IdeaProjects\TextCleaner\word-addin-modernization\test-documents`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\compatibility-results.md`
- `K:\IdeaProjects\TextCleaner\word-addin-modernization\verification-log.md`

**Design/implementation work:**

- Create or curate fixture `.docx` files with:
  - plain paragraphs
  - headings
  - styles
  - tables
  - comments
  - hyperlinks
  - fields
  - bookmarks
  - track changes
  - footnotes/endnotes
  - multilingual text
- Run selected-range tests first.
- Run document-wide tests only after selected-range tests pass.
- Record formatting-preservation outcomes.
- Capture Word version/build and Office channel.

**Acceptance evidence:**

- `compatibility-results.md` includes actual Word version/build.
- Each MVP command has a fixture result.
- Failures include reproduction steps and affected command IDs.

### 8. VSTO And .NET Fallback Agent

**Purpose:** Determine whether any high-value feature truly requires a Windows desktop COM add-in.

**Skills:**

- `modernize-dotnet` MCP agent only if a real `.sln`/`.csproj`/`.vbproj` exists
- `superpowers:writing-plans`
- `superpowers:verification-before-completion`

**Owns:**

- `K:\IdeaProjects\TextCleaner\word-addin-modernization\vsto-decision.md`
- future `K:\IdeaProjects\TextCleaner\word-vsto-addin` only if approved

**Design/implementation work:**

- Review `VSTO-only` rows from `compatibility-matrix.md`.
- Check whether each blocked feature is worth a second add-in.
- If VSTO is approved, create a separate VSTO plan and only then use modernize-dotnet MCP against actual project files.
- Do not try to feed exported VBA modules into .NET upgrade tools.

**Acceptance evidence:**

- Every VSTO candidate has a value/risk rationale.
- No VSTO project exists unless approved by the Orchestrator.
- modernize-dotnet MCP calls are limited to real .NET projects.

## Agent Execution Model

Use a gated pipeline with selective parallelism:

```text
Orchestrator
  |
  +-- Source Inventory Agent
  |
  +-- VBA Behavior Classification Agent
  |     depends on canonical source decision
  |
  +-- Office Add-in Compatibility Agent
  |     can run in parallel with behavior classification
  |
  +-- Office.js Implementation Agent
  |     depends on MVP scope and compatibility decisions
  |
  +-- Task Pane And Manifest Agent
  |     can repair scaffold early, but command wiring depends on implementation
  |
  +-- Document Fidelity And Verification Agent
  |     starts fixture design early, verifies after implementation
  |
  +-- VSTO And .NET Fallback Agent
        starts only after compatibility matrix has VSTO-only candidates
```

Parallel-safe work:

- Source inventory and Office add-in compatibility research can run at the same time.
- Fixture design can begin while implementation is still writing pure rules.
- Manifest branding cleanup can happen before cleanup-rule implementation if the manifest validation command is run immediately afterward.

Not parallel-safe:

- Two agents editing `appPackage\manifest.json`.
- Two agents editing `src\taskpane\word.ts`.
- Implementing Word adapters before the compatibility matrix identifies supported APIs.
- Creating VSTO project files before the VSTO decision gate.

## Required Artifacts

| Artifact | Owner | Purpose |
|---|---|---|
| `source-inventory.md` | Source Inventory Agent | File counts, canonical source roots, duplicate decisions |
| `source-hashes.csv` | Source Inventory Agent | Hash evidence for duplicate exports |
| `compatibility-matrix.md` | VBA Behavior Classification Agent | Feature-by-feature migration decisions |
| `mvp-scope.md` | VBA Behavior Classification Agent | Small first set of add-in features |
| `officejs-compatibility.md` | Office Add-in Compatibility Agent | Requirement sets, manifest strategy, API constraints |
| `migration-log.md` | Office.js Implementation Agent | Source-to-TypeScript traceability |
| `compatibility-results.md` | Document Fidelity Agent | Actual Word build and feature results |
| `verification-log.md` | Document Fidelity Agent | Commands run and observed outcomes |
| `private-deployment.md` | Task Pane/Manifest Agent with Orchestrator | Private installation and hosting path |
| `vsto-decision.md` | VSTO Fallback Agent | Whether a separate VSTO add-in is justified |

## Implementation Strategy By Component

### Component A: Governance And Handoff Schema

**Design:**

Each agent report must use this schema:

```markdown
# Agent Report: <agent name>

## Scope
Files read:
Files changed:
Files intentionally not touched:

## Findings

## Decisions

## Evidence
Commands run:
Outputs summarized:

## Risks

## Next handoff
```

**Implementation:**

- Orchestrator creates `word-addin-modernization\agent-reports`.
- Every agent writes one report per assigned phase.
- Reports must include exact paths and command evidence.

### Component B: Source And Behavior Assessment

**Design:**

Source provenance precedes code migration. Behavior examples precede implementation.

**Implementation:**

- Source Inventory Agent creates source counts and hashes.
- VBA Behavior Classification Agent reviews MVP candidates first.
- Compatibility decisions are recorded before TypeScript work starts.

### Component C: Office.js Compatibility Design

**Design:**

Every feature has a requirement-set decision. Desktop Word MVP avoids WordApiOnline-only APIs.

**Implementation:**

- Office Add-in Compatibility Agent documents each selected API.
- Runtime guards are specified for APIs above the baseline requirement set.
- Manifest requirements remain minimal unless a feature genuinely requires a higher requirement set.

### Component D: Core Cleanup Engine

**Design:**

Rules are pure functions first, Word adapters second.

**Implementation:**

- Office.js Implementation Agent adds `cleanupTypes.ts`, `cleanupRules.ts`, tests, and migration log.
- Tests cover deterministic before/after examples from VBA behavior classification.
- Word adapters expose selected-range commands before document-wide commands.

### Component E: Task Pane And Commands

**Design:**

The UI exposes a safe selected-text workflow first. Ribbon command opens the pane before it runs destructive actions.

**Implementation:**

- Task Pane And Manifest Agent changes Word debug defaults and branding.
- Adds checkbox-based rule selection.
- Adds status output with replacement counts.
- Adds "Preview selection" and "Clean selection".
- Adds document-wide cleanup only after verification passes.

### Component F: Verification And Document Fidelity

**Design:**

Real Word documents prove safety. Pure TypeScript tests prove deterministic rules but do not prove document fidelity.

**Implementation:**

- Document Fidelity Agent creates fixtures.
- Runs `npm run build`, `npm run validate`, and `npm run start:desktop:word`.
- Records Word build/channel.
- Records before/after results per fixture.

### Component G: Private Deployment

**Design:**

Keep deployment private and avoid public publishing.

**Implementation:**

- Decide on local sideload, private HTTPS hosting, Azure Static Web Apps, or tenant-scoped deployment.
- Replace `https://www.contoso.com/`.
- Validate production manifest.
- Document exact install path in `private-deployment.md`.

### Component H: VSTO Fallback

**Design:**

VSTO is a last resort for high-value desktop-only capabilities.

**Implementation:**

- VSTO Fallback Agent only starts after the compatibility matrix has blocked features.
- If approved, create a separate plan for `word-vsto-addin`.
- Use modernize-dotnet MCP only after real .NET project files exist.

## Final Skill-Agent Mapping

| Main plan phase | Primary agent | Supporting agents | Required skills |
|---|---|---|---|
| Phase 0: Stabilize workspace | Orchestrator | Task Pane And Manifest Agent | `executing-plans`, `verification-before-completion`, `systematic-debugging` |
| Phase 1: Canonicalize VBA source | Source Inventory Agent | Orchestrator | `executing-plans`, `verification-before-completion` |
| Phase 2: Build compatibility matrix | VBA Behavior Classification Agent | Office Add-in Compatibility Agent | `writing-plans`, `test-driven-development` |
| Phase 3: Repair add-in scaffold | Task Pane And Manifest Agent | Office Add-in Compatibility Agent | `test-driven-development`, `verification-before-completion` |
| Phase 4: Port pure cleanup rules | Office.js Implementation Agent | VBA Behavior Classification Agent | `test-driven-development`, `systematic-debugging` |
| Phase 5: Connect rules to Word | Office.js Implementation Agent | Document Fidelity Agent | `test-driven-development`, `verification-before-completion` |
| Phase 6: Build task pane and commands | Task Pane And Manifest Agent | Office.js Implementation Agent | `test-driven-development`, `verification-before-completion` |
| Phase 7: Test and validate | Document Fidelity Agent | Orchestrator | `systematic-debugging`, `verification-before-completion` |
| Phase 8: Private deployment | Task Pane And Manifest Agent | Orchestrator | `verification-before-completion` |
| Phase 9: VSTO fallback decision | VSTO And .NET Fallback Agent | Office Add-in Compatibility Agent | `writing-plans`, `modernize-dotnet` only if real .NET project exists |
| Phase 10: modernize-dotnet boundary | Orchestrator | VSTO And .NET Fallback Agent | `verification-before-completion` |

## Completion Criteria

This side-plan is successful when:

- Every main-plan phase has one primary agent.
- Every agent has a bounded write scope.
- Every agent has required evidence artifacts.
- Office.js compatibility decisions are documented before implementation.
- Source traceability exists before porting.
- Tests and real Word verification exist before broad document cleanup.
- VSTO is not introduced unless the compatibility matrix proves it is needed.
- modernize-dotnet MCP is used only for actual .NET projects, not exported VBA.

## End-Of-Turn Protocol

At the end of every turn in this workspace, include:

- Agent and skill warnings: which agent and skills should be activated for the next step, and which ones should be avoided to prevent scope drift, tool misuse, or inefficient execution.
- User preflight warnings: anything the user should do before sending the next prompt so the next execution goes smoothly.
- Next-step prompt: one copy-paste-ready prompt in a fenced code block.
- Plan persistence: if these operating rules need to survive future sessions, keep this section in this side-plan and update it only when the user changes the protocol.

## Immediate Next Step

Create `K:\IdeaProjects\TextCleaner\word-addin-modernization\agent-reports`, then assign the first two parallel-safe jobs:

1. Source Inventory Agent: produce `source-inventory.md` and `source-hashes.csv`.
2. Office Add-in Compatibility Agent: produce `officejs-compatibility.md` for the MVP feature set and the existing unified manifest.
