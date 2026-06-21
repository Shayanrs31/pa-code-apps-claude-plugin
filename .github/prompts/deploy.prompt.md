---
mode: agent
tools: [codebase, terminalLastCommand, runCommands, readFile, writeFile]
description: Build and deploy the current Power Apps code app to Power Platform
---

# Deploy Power Apps Code App

You are deploying the current code app to Power Platform.

---

## Step 1: Confirm

Ask the user:

> "This will build and deploy the current code to {environment-name}. Proceed?"

Do not deploy without explicit confirmation in the current message.

---

## Step 2: Read memory bank

Read `memory-bank.md` to get the app name, version, environment ID, and app URL.

---

## Step 3: Build

```powershell
npm run build
```

If the build fails: show the TypeScript error output and stop. Do not deploy a broken build.

Verify `dist/` was created and contains `index.html`.

---

## Step 4: Deploy

```powershell
pwsh -NoProfile -Command "pac code push"
```

If the command fails with a non-zero exit: report the exact error and stop.

---

## Step 5: Increment version

After a successful deploy:
- Increment the patch version in the version display (e.g., `v1.0.0` to `v1.0.1`)
- Update `memory-bank.md` with the new version

---

## Step 6: Report result

```
Deployed successfully.

App: {app-name} {new-version}
URL: {app-url}
Environment: {environment-name}

Changes are usually live within 30 seconds.
```

---

## Common errors

**"App not found":** `power.config.json` may have an incorrect `appId`. Run `pac code init` again or update `power.config.json` from the maker portal.

**"Unauthorized":** Auth profile may have expired. Run `pac auth list` to check. Clear and re-authenticate if needed.

**TS6133 (unused variable):** Remove the unused import or variable before building.
