# pa-code-apps: Claude Code Plugin for Power Apps Code Apps

A Claude Code plugin that gives you four specialised agents and 14 skills for building, reviewing, deploying, and documenting Power Apps code apps.

## Install
# pa-code-apps: Claude Code Plugin for Power Apps Code Apps

A Claude Code plugin that gives you four specialised agents and 14 skills for building, reviewing, deploying, and documenting Power Apps code apps.


Requires Claude Code v1.0 or later.

---

## What are Power Apps Code Apps?

Code apps let you build custom React/Vite/TypeScript single-page applications hosted inside Power Platform. You get Microsoft Entra authentication, access to 1,500+ Power Platform connectors, and managed DLP and Conditional Access policies without writing any auth code.

---

## Agents

This plugin includes four agents that cover the full app lifecycle. They are invoked automatically by skills and do not need to be called directly.

| Agent | Role |
|---|---|
| `pa-plan` | Gathers requirements, designs the implementation plan, renders UI mockups, writes the memory bank stub |
| `pa-code` | Scaffolds the app, wires connectors, implements hooks and pages, builds and deploys |
| `pa-review` | Audits code for hook pattern violations, hardcoded identity, OData safety, and Dataverse quirks |
| `pa-document` | Generates Technical Design Documents, User Guides, Handover Packs, and Architecture Overviews in Markdown |

---

## Skills

### Core workflow

| Skill | What it does |
|---|---|
| `/create-code-app` | Full scaffold-to-deploy workflow: plan, scaffold, init, build, add connectors, implement, deploy |
| `/deploy` | Build and push the current app to Power Platform |
| `/document` | Generate a Technical Design Document, User Guide, Handover Pack, or Architecture Overview |

### Add connectors

| Skill | Connector |
|---|---|
| `/add-dataverse` | Microsoft Dataverse |
| `/add-sharepoint` | SharePoint Online |
| `/add-office365` | Office 365 Outlook |
| `/add-teams` | Microsoft Teams |
| `/add-onedrive` | OneDrive for Business |
| `/add-excel` | Excel Online |
| `/add-azuredevops` | Azure DevOps |
| `/add-flows` | Power Automate cloud flows (preview) |
| `/add-connector` | Any other Power Platform connector |
| `/add-datasource` | Generic data source discovery and add |

### Utilities

| Skill | What it does |
|---|---|
| `/list-connections` | List available connections in the current environment |

---

## Prerequisites

- Node.js v22 or later
- Power Platform CLI (`pac`) installed
- An active Power Platform environment with code apps enabled
- Claude Code v1.0 or later

To enable code apps in your environment: Power Platform admin center → Environments → Settings → Product → Features → Enable code apps.

---

## Usage

### Start a new app

/create-code-app

The plugin will ask what you want to build, design a plan with UI mockups, scaffold the React/Vite project, initialise it against your environment, and guide you through connector setup and deployment.

### Add a connector to an existing app

/add-dataverse
/add-sharepoint
/add-flows


Each skill handles discovery, adds the data source via CLI, generates typed TypeScript services, and prompts you to wire a hook.

### Review before releasing

/code-review

Audits for critical issues (hook pattern violations, hardcoded identity, modified generated files), warnings (missing OData $top, no loading state), and suggestions.
### Generate documentation
/document

Choose from Technical Design Document, User Guide, Handover Pack, or Architecture Overview. Output is always Markdown, rendered in chat or saved to `docs/` in your project.
---
## How it works
The plugin enforces a strict pattern:
- All connector and service calls go in `src/hooks/`. Never directly in components.
- User identity always comes from `getContext()` via `useCurrentUser`. Never hardcoded.
- Pages in `src/pages/`, shared UI in `src/components/`
- Files under `src/generated/` are never edited manually
- `pac code push` requires explicit confirmation before every deploy
A `memory-bank.md` file is written to the project root after planning and updated throughout the build. This lets Claude pick up context across sessions without re-asking questions.
---
## Known issues
The plugin includes a `shared/known-issues.md` file with verified fixes for four common problems:
1. `trackScenario` / libuv crash: Node.js version below v22
2. `-cr` flag invalid: use `--connectionReference` long form
3. `MSCRM.IncludeMipSensitivityLabel` TypeScript error: add a `postgenerate` script
4. Connection reference resolution failure: query Dataverse Web API for the real `solutionid`
---
## Author
Shayan R S: https://www.linkedin.com/in/shayan-rs
