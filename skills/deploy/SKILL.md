---
name: deploy
description: Build and deploy the current Power Apps code app to Power Platform. Requires explicit user confirmation before pushing.
user-invocable: true
allowed-tools: Read, Write, Bash
model: sonnet
---

# Skill: deploy

Build and deploy the current code app to Power Platform.

---

## When invoked

The user wants to push the current state of the app to their Power Apps environment.

---

## Workflow

### Step 1: Confirm

Ask the user to confirm:

> "This will build and deploy the current code to {environment-name}. Proceed?"

Do not deploy without explicit confirmation in the current turn.

### Step 2: Read memory bank

Read `memory-bank.md` to get:
- App name and version
- Environment ID
- App URL (to verify after deploy)

### Step 3: Build

```powershell
npm run build
```

If the build fails: show the TypeScript error output and stop. Do not deploy a broken build.

Verify `dist/` was created and contains `index.html`.

### Step 4: Deploy

```powershell
pwsh -NoProfile -Command "pac code push"
```

Capture the output. If the command fails with a non-zero exit: report the exact error and stop.

### Step 5: Increment version

After a successful deploy:
- Increment the patch version in the version display (e.g., `v1.0.0` → `v1.0.1`)
- Update `memory-bank.md` with the new version

### Step 6: Confirm

Report the result:

```
Deployed successfully.

App: {app-name} {new-version}
URL: {app-url}
Environment: {environment-name}

To verify: open the URL above. Changes are usually live within 30 seconds.
```

---

## Common errors

**`pac code push` returns "App not found":**
The `power.config.json` may have an incorrect `appId`. Run `pac code init` again or manually update `power.config.json` with the correct app ID from the Power Apps maker portal.

**`pac code push` returns "Unauthorized":**
The auth profile may have expired. Run `pac auth list` to check. Clear and re-authenticate if needed.

**`npm run build` fails with TS6133 (unused variable):**
Remove the unused import or variable. TypeScript strict mode does not allow unused declarations.
