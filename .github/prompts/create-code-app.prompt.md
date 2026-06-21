---
mode: agent
tools: [codebase, terminalLastCommand, runCommands, readFile, writeFile, editFile]
description: Scaffold and deploy a new Power Apps code app from scratch
---

# Create Power Apps Code App

You are helping scaffold and deploy a new Power Apps code app.

Follow `.github/instructions/known-issues.md` before running any pac command.
Follow `.github/instructions/development-standards.md` for all code written.

---

## Step 1: Validate prerequisites

```powershell
node --version
pwsh -NoProfile -Command "pac help"
```

Node must be v22 or later. If pac is missing, stop and tell the user to install it from https://aka.ms/PowerAppsCLI.

---

## Step 2: Tutorial mode

Ask the user before anything else:

> "Would you like tutorial mode? In tutorial mode, each command is shown and explained before it runs and waits for your approval. (yes / no)"

---

## Step 3: Auth and environment

```powershell
pwsh -NoProfile -Command "pac auth list"
pwsh -NoProfile -Command "pac env list"
```

Let the user pick the correct environment. Run:

```powershell
pwsh -NoProfile -Command "pac env select --environment <id>"
```

---

## Step 4: Gather requirements

Ask the user:
- What would you like to build? Describe it in your own words.
- What data does the app need to work with?
- How many screens? What does each screen show?
- Dark or light theme? (default: dark)

Map data answers to connectors:
- Custom business records: Dataverse
- SharePoint lists: SharePoint Online
- Emails and calendar: Office 365 Outlook
- Files: OneDrive for Business
- Teams messages: Microsoft Teams
- Work items: Azure DevOps
- Custom API: HTTP connector

Present a plan and wait for approval before writing any code.

---

## Step 5: Scaffold

Ask the user for a folder name. Default: `powerapps-{app-name-slugified}`.

```powershell
npx degit microsoft/PowerAppsCodeApps/templates/vite <folder> --force
cd <folder>
npm install
```

---

## Step 6: Initialise

```powershell
pwsh -NoProfile -Command "pac code init --displayName '<app-name>' --environment <environment-id>"
```

Read `power.config.json` and verify `environmentId` matches. Correct if mismatched.

---

## Step 7: Baseline build and deploy

```powershell
npm run build
pwsh -NoProfile -Command "pac code push"
```

Capture the app URL from output. Create `memory-bank.md` now with app name, environment ID, app URL, and `tutorial_mode` preference.

---

## Step 8: Add data sources

Run the relevant prompt for each data source planned:
- `#add-dataverse` for Dataverse tables
- `#add-sharepoint` for SharePoint
- `#add-flows` for Power Automate flows

---

## Step 9: Implement

Follow `.github/instructions/development-standards.md`:
- All service calls in `src/hooks/`
- Pages in `src/pages/`
- Shared UI in `src/components/`
- Show version in the UI
- Run `npm run build` after every meaningful change

---

## Step 10: Final deploy

Ask the user:

> "Ready to deploy to {environment name}? This will update the live app."

Wait for explicit confirmation. Then:

```powershell
npm run build
pwsh -NoProfile -Command "pac code push"
```

Update `memory-bank.md` with final state and new version.
