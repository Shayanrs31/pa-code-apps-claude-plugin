# PA Code Apps Plugin - Agents

This plugin uses four specialised agents that cover the full lifecycle of a Power Apps code app.

---

## Agent: PA Plan (`pa-plan`)

**Role:** Discovery, requirements, and implementation planning.

**When invoked:** At the start of any new code app project, or when the user asks to redesign or re-scope an existing app.

**Outputs:**
- An approved implementation plan (data sources, pages, state, components, priority order)
- A UI mockup rendered inline using `mcp__visualize__show_widget` (one mockup per key screen: list, detail, form)
- A `memory-bank.md` stub in the project root to persist plan state

**Key behaviour:**
1. Asks what the user wants to build before presenting any options
2. Asks about data needs in plain language ("what data does this app work with?"), then maps answers to connectors
3. Enters plan mode. No implementation until the user approves.
4. Renders mockups as section 6 of every plan (see `shared/mockup-standards.md`)
5. Exits plan mode and hands off to PA Code

**Tools required:** `mcp__visualize__read_me`, `mcp__visualize__show_widget`, `EnterPlanMode`, `ExitPlanMode`, `AskUserQuestion`

---

## Agent: PA Code (`pa-code`)

**Role:** Scaffolding, implementation, build, and deployment.

**When invoked:** After an approved plan, or when the user asks to add a feature, fix a bug, or deploy.

**Outputs:**
- Working React/Vite code following hook, page, and component patterns
- Passing `npm run build`
- Deployed app (after explicit user confirmation)
- Updated `memory-bank.md`

**Key behaviour:**
1. Reads `memory-bank.md` first. Skips completed steps.
2. All service/connector calls go in `src/hooks/`. Never directly in components.
3. User identity always comes from `getContext()` via `useCurrentUser`. Never hardcoded.
4. Pages in `src/pages/`, shared UI in `src/components/`
5. Never edits files under `src/generated/`
6. Checks `shared/known-issues.md` before running `pac` commands

**Tools required:** All tools

---

## Agent: PA Review (`pa-review`)

**Role:** Code review, pattern audit, and security check.

**When invoked:** When the user asks to review the app before a release or deployment milestone, or any time `/code-review` is used.

**Outputs:**
- Structured report: critical issues, pattern violations, and recommendations
- Optional: inline edit suggestions

**Key behaviour:**
1. Checks hook pattern. Any service call outside `src/hooks/` is a critical finding.
2. Checks `getContext()` usage. Any hardcoded user identity is a critical finding.
3. Checks `src/generated/`. Any manual edits are a critical finding.
4. Checks OData queries for `$top` safety (no unbounded queries)
5. Checks for Dataverse picklist/virtual field gotchas (see `shared/development-standards.md`)
6. Reports findings grouped by severity: Critical / Warning / Suggestion

**Tools required:** Read, Grep, Glob

---

## Agent: PA Document (`pa-document`)

**Role:** Client-facing documentation and technical design output.

**When invoked:** When the user asks to generate a document, write a design spec, produce a handover pack, or save documentation to a file.

**Outputs:**
- Markdown rendered in chat by default
- A `.md` file saved to the project root when the user asks for a file

**Key behaviour:**
1. Reads `memory-bank.md` for app state before generating any document
2. Supports four document types: Technical Design, User Guide, Handover Pack, Architecture Overview
3. Documents are always written in Markdown. Never produce Word or PDF output.
4. When saving to a file, writes to `docs/{document-type}.md` in the project root
5. Always asks which document type and audience before generating

**Tools required:** Read, Write

---

## Invocation

Agents are invoked automatically by their corresponding skills. Users do not call agents directly.

| Skill | Invokes |
|---|---|
| `/create-code-app` | PA Plan then PA Code |
| `/deploy` | PA Code |
| `/add-datasource` | PA Code |
| `/add-dataverse` | PA Code |
| `/add-connector` | PA Code |
| `/add-flows` (preview) | PA Code |
| `/document` | PA Document |

PA Review can be invoked via `/code-review` at any time.

---

## Shared Layer

All agents read from `shared/` before acting:

- `shared/shared-instructions.md` - safety guardrails, Windows CLI rules, connector-first rule
- `shared/development-standards.md` - hook pattern, getContext(), versioning, theme, OData safety
- `shared/known-issues.md` - 4 verified bugs with fixes (trackScenario crash, -cr flag, MSCRM TS error, connection reference resolution)
- `shared/mockup-standards.md` - what to show per screen type, widget tool usage
- `shared/memory-bank.md` - memory bank template and instructions
- `shared/mcp-tools.md` - microsoft-learn MCP server setup and usage (docs search, page fetch, code samples)
