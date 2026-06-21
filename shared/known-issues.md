# Known Issues

Three verified bugs in the Power Apps Code Apps toolchain. Check this file before running any `pac code` command.

---

## Issue 1: trackScenario crash on app load

**Symptom:** The app throws `TypeError: Cannot read properties of undefined (reading 'trackScenario')` immediately on load in the Power Apps player. The app works fine in `npm run dev` but fails after `pac code push`.

**Root cause:** The telemetry module is not initialised before the first render completes. Calling `trackScenario` synchronously in a component body (not inside a `useEffect`) hits the uninitialised SDK.

**Fix:**
```typescript
// Wrong - synchronous call in component body
import { trackScenario } from '@microsoft/power-apps/telemetry';
export default function App() {
  trackScenario('app-load'); // crashes
  return <div>...</div>;
}

// Correct - wrapped in useEffect
import { useEffect } from 'react';
import { trackScenario } from '@microsoft/power-apps/telemetry';
export default function App() {
  useEffect(() => {
    trackScenario('app-load');
  }, []);
  return <div>...</div>;
}
```

---

## Issue 2: -cr flag invalid on pac code add-data-source

**Symptom:** Running `pac code add-data-source -cr <connection-id>` returns `Unrecognized option '-cr'`.

**Root cause:** The short flag for `--connectionReference` changed between pac CLI versions. The `-cr` alias was removed in 2.4.x.

**Fix:** Always use the full flag name:
```powershell
# Wrong
pwsh -NoProfile -Command "pac code add-data-source -a dataverse -cr conn-xyz"

# Correct
pwsh -NoProfile -Command "pac code add-data-source --action dataverse --connectionReference conn-xyz"
```

Check available flags with:
```powershell
pwsh -NoProfile -Command "pac code add-data-source --help"
```

---

## Issue 3: MSCRM ambient module TypeScript error

**Symptom:** After running `pac code add-data-source` for Dataverse, `npm run build` fails with:
```
error TS2306: File 'src/generated/models/CrmModel.ts' is not a module.
```
or
```
error TS7016: Could not find a declaration file for module '@types/xrm'
```

**Root cause:** The generated Dataverse service files reference MSCRM types that require an ambient declaration. The scaffolded `tsconfig.json` does not include them by default.

**Fix:** Add the missing ambient type to `tsconfig.json`:
```json
{
  "compilerOptions": {
    "types": ["@types/xrm"]
  }
}
```

Then install the type package:
```bash
npm install --save-dev @types/xrm
```

If the error persists, add a declaration file at `src/global.d.ts`:
```typescript
/// <reference types="@types/xrm" />
declare namespace Xrm {}
```

---

## Issue 4: Connection reference resolution fails with "Failed to resolve connection ID"

**Symptom:** Running `pac code add-data-source -cr <connection-reference-logical-name>` fails with:
```
Failed to resolve connection ID for reference '<cr-logical-name>'
```
The connection reference exists in Dataverse and has a connection bound to it, but pac CLI still cannot resolve it.

**Root cause:** pac CLI resolves the connection reference by looking up the `solutionid` stored on the CR record in Dataverse. The system-internal solution GUID on that record does not match your custom solution, and does not appear in `pac solution list`. The CLI cannot find it without the correct GUID being passed explicitly.

**Fix:**

Step 1. Query the Dataverse Web API to get the real `solutionid` off the CR record:

```
GET <org-url>/api/data/v9.2/connectionreferences?$filter=connectionreferencelogicalname eq '<your-cr-logical-name>'&$select=solutionid,connectionreferencelogicalname
```

You can run this in the browser while signed into your environment, or use the `mcp__DataverseMCPServer__read_query` tool if the Dataverse MCP server is connected.

Step 2. Pass the `solutionid` from the response as the `-s` parameter:

```powershell
pwsh -NoProfile -Command "pac code add-data-source -a <connector-api-name> -cr <cr-logical-name> -s <solutionid-from-step-1>"
```

Step 3. After the command succeeds, verify that `power.config.json` now contains a `connectionReferences` entry for the CR. This confirms the connection reference is wired up correctly for ALM.
