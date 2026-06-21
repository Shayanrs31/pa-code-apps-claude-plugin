
# pa-code-apps: Claude Code Plugin for Power Apps Code Apps

A Claude Code plugin that gives you four specialised agents and 14 skills for building, reviewing, deploying, and documenting Power Apps code apps.

---

## Getting Started

### 1. Prerequisites

Before installing the plugin, make sure you have:

- Claude Code v1.0 or later
- Node.js v22 or later
- Power Platform CLI (`pac`) installed: https://aka.ms/PowerAppsCLI
- A Power Platform environment with code apps enabled

To enable code apps in your environment: Power Platform admin center → Environments → Settings → Product → Features → Enable code apps.

### 2. Install the plugin

Clone the repo into your Claude Code skills folder:

**Windows:**
```
git clone https://github.com/Shayanrs31/pa-code-apps-claude-plugin.git "%USERPROFILE%\.claude\skills\pa-code-apps"
```

**Mac / Linux:**
```
git clone https://github.com/Shayanrs31/pa-code-apps-claude-plugin.git ~/.claude/skills/pa-code-apps
```

Then restart Claude Code. The plugin loads automatically.

### 3. Build your first app

Navigate to the folder where you want to create the project, then run:

```
/create-code-app
```

The plugin will ask what you want to build, design a plan with UI mockups, scaffold the React/Vite project, initialise it against your Power Platform environment, and guide you through connector setup and deployment.

### 4. Add connectors

```
/add-dataverse
/add-sharepoint
/add-flows
```

### 5. Review and document

```
/create-code-app
/document
/code-review
```

---

## Using with VS Code Copilot

This repo also ships a `.github/` folder and `.vscode/` config that bring the same knowledge into GitHub Copilot agent mode in VS Code.

### Setup

Copy the `.github/` and `.vscode/` folders from this repo into your code app project root.

Update `.vscode/mcp.json` with your Dataverse environment URL:

```json
"DATAVERSE_URL": "https://your-org.crm11.dynamics.com"
```

### What you get

| File | Purpose |
|---|---|
| `.github/copilot-instructions.md` | Always-on rules loaded into every Copilot Chat session (hook pattern, identity, generated files) |
| `.github/instructions/development-standards.md` | Applied automatically to all `.ts` and `.tsx` files |
| `.github/instructions/known-issues.md` | Applied to all files — checked before every `pac code` command |
| `.github/prompts/create-code-app.prompt.md` | Reusable prompt for scaffolding a new app |
| `.github/prompts/add-dataverse.prompt.md` | Reusable prompt for adding Dataverse |
| `.github/prompts/deploy.prompt.md` | Reusable prompt for building and deploying |
| `.github/prompts/document.prompt.md` | Reusable prompt for generating documentation |
| `.github/prompts/add-flows.prompt.md` | Reusable prompt for Power Automate flows (preview) |
| `.github/prompts/update-csp.prompt.md` | Reusable prompt for Content Security Policy |
| `.vscode/mcp.json` | MCP servers: Microsoft Learn docs + Dataverse |
| `.vscode/settings.json` | Wires the instruction files into Copilot code generation |

### Using the prompts

In VS Code Copilot Chat (agent mode), type `/` to see available prompts or reference them directly:

```
/create-code-app
/deploy
/add-dataverse
```

The instruction files load automatically. No extra steps needed.

---

## Updating

To get the latest version, pull from inside the skills folder:

**Windows:**
```
cd "%USERPROFILE%\.claude\skills\pa-code-apps"
git pull
```

**Mac / Linux:**
```
cd ~/.claude/skills/pa-code-apps
git pull
```

Then restart Claude Code.

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
