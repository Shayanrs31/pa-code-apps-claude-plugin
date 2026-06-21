---
applyTo: "**"
---

# Known Issues

Check this before running any `pac code` command.

---

## Issue 1: trackScenario crash on app load

**Symptom:** `TypeError: Cannot read properties of undefined (reading 'trackScenario')` in the Power Apps player. Works in `npm run dev` but fails after deploy.

**Fix:** Wrap `trackScenario` in `useEffect`, never call it synchronously in a component body.

```typescript
// Wrong
trackScenario('app-load');

// Correct
useEffect(() => { trackScenario('app-load'); }, []);
```

---

## Issue 2: -cr flag invalid

**Symptom:** `pac code add-data-source -cr <id>` returns `Unrecognized option '-cr'`.

**Fix:** Use the full flag name:

```powershell
pwsh -NoProfile -Command "pac code add-data-source --action dataverse --connectionReference conn-xyz"
```

---

## Issue 3: MSCRM TypeScript error after add-data-source

**Symptom:** `npm run build` fails with `error TS2306` or `error TS7016` after adding a Dataverse data source.

**Fix:**

```json
// tsconfig.json
{ "compilerOptions": { "types": ["@types/xrm"] } }
```

```bash
npm install --save-dev @types/xrm
```

If the error persists, add `src/global.d.ts`:

```typescript
/// <reference types="@types/xrm" />
declare namespace Xrm {}
```

---

## Issue 4: Connection reference resolution fails

**Symptom:** `pac code add-data-source -cr <name>` fails with `Failed to resolve connection ID for reference`.

**Fix:**

Step 1. Query Dataverse Web API for the real `solutionid`:

```
GET <org-url>/api/data/v9.2/connectionreferences?$filter=connectionreferencelogicalname eq '<cr-name>'&$select=solutionid
```

Step 2. Pass it as `-s`:

```powershell
pwsh -NoProfile -Command "pac code add-data-source -a <api-name> -cr <cr-name> -s <solutionid>"
```

Step 3. Verify `power.config.json` contains a `connectionReferences` entry for the CR.
