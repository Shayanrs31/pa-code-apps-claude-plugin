---
name: add-flows
description: Add Power Automate cloud flows (preview) to the current code app. Use when the app needs to trigger a flow with a PowerApps trigger. npm CLI only - not available via pac code.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash
model: sonnet
---

# Skill: add-flows (preview)

Add Power Automate cloud flows to a code app. This feature is **preview** as of June 2026.

Reference: https://learn.microsoft.com/en-us/power-apps/developer/code-apps/how-to/add-flows

---

## Prerequisites

Before running any commands, verify:

1. The app was initialised with `npx power-apps init` (npm CLI, not `pac code init`)
2. `@microsoft/power-apps` version is **1.1.1 or later**:
   ```powershell
   node -e "console.log(require('./node_modules/@microsoft/power-apps/package.json').version)"
   ```
   If below 1.1.1, upgrade: `npm install @microsoft/power-apps@latest`
3. The target flow meets all three criteria:
   - Trigger type: **PowerApps** (manual instant flow)
   - Belongs to a **solution** (non-solution flows are not listed)
   - The maker running this skill has access to the flow and its underlying connections

If the user's flow is not yet in a solution, ask them to add it at make.powerautomate.com before continuing.

---

## Workflow

### Step 1: List available flows

```powershell
npx power-apps list-flows
```

This lists all solution-aware flows in the current environment with name, status, and flow ID.

To filter by name:

```powershell
npx power-apps list-flows --search <keyword>
```

Copy the **Flow ID** of the flow to add.

### Step 2: Add the flow

```powershell
npx power-apps add-flow --flow-id <flow-id>
```

When successful, the CLI confirms: `Flow added successfully.`

**What this command does:**

- Downloads the flow's OpenAPI definition
- Generates typed TypeScript files in `src/services/` and `src/models/`
- Creates `schemas/logicflows/<FlowName>.Schema.json` (do not edit manually)
- Updates `power.config.json` with the flow's connection references

**Generated file locations** (NOT in `src/generated/` - these go in `src/services/` and `src/models/`):

```
src/
  services/
    <FlowName>Service.ts   <- generated service with Run() method
  models/
    <FlowName>Model.ts     <- typed inputs and outputs
schemas/
  logicflows/
    <FlowName>.Schema.json <- OpenAPI schema (do not edit)
```

**Note:** If the command fails with an authorization error, the maker does not have access to the flow's underlying connections (for example, an Office 365 Outlook connection used by the flow). The connection owner must share access before re-running.

### Step 3: Build to verify

```powershell
npm run build
```

Fix any TypeScript errors before continuing. The generated service file shows the exact input and output types for the flow.

### Step 4: Wire a hook

Wrap all flow calls in a hook under `src/hooks/`. Never call the service directly from a component.

**Flow with input parameters:**

```typescript
// src/hooks/useApprovalWorkflow.ts
import { useState } from 'react';
import { ApprovalWorkflowService } from '../services/ApprovalWorkflowService';

export function useApprovalWorkflow() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function trigger(requester: string, amount: number) {
    setLoading(true);
    setError(null);
    try {
      const result = await ApprovalWorkflowService.Run({ requester, amount });
      if (!result.success) throw new Error(String(result.error));
      return result.data;
    } catch (e) {
      setError((e as Error).message);
      return null;
    } finally {
      setLoading(false);
    }
  }

  return { trigger, loading, error };
}
```

**Flow with no input parameters:**

```typescript
// src/hooks/useSendNotification.ts
import { SendNotificationService } from '../services/SendNotificationService';

export function useSendNotification() {
  async function trigger() {
    const result = await SendNotificationService.Run();
    if (!result.success) throw new Error(String(result.error));
  }
  return { trigger };
}
```

**Result object shape:**

| Property | Type | Description |
|---|---|---|
| `success` | `boolean` | `true` if the flow triggered successfully |
| `data` | varies | Typed response from the flow, if any |
| `error` | `Error` (optional) | Details when `success` is `false` |

Open the generated service file to see the exact types. Parameters marked `x-ms-visibility: internal` with a default value are inlined by the code generator and not exposed in the method signature.

### Step 5: Update memory bank

Record the flow in `memory-bank.md`:

```markdown
## Flows (Preview)
- <FlowName>: <what it does> (Flow ID: <uuid>)
```

---

## Updating a flow

If the flow's definition changes (new parameters, updated connections), re-run add-flow with the same flow ID:

```powershell
npx power-apps add-flow --flow-id <flow-id>
```

The command matches by `workflowEntityId` in `power.config.json` and updates in place. No manual cleanup required.

---

## Removing a flow

By data source name:

```powershell
npx power-apps remove-flow --flow-name <FlowName>
```

By flow ID:

```powershell
npx power-apps remove-flow --flow-id <flow-id>
```

Both commands remove the flow from `power.config.json` and regenerate the model services.

---

## Deploying

After adding flows and verifying locally with `npm run dev`:

```powershell
npm run build
npx power-apps push
```

---

## Limitations (preview)

| Limitation | Detail |
|---|---|
| PowerApps trigger only | Only manual instant flows with the PowerApps trigger are supported. Scheduled, automated, and other trigger types are not supported. |
| Solution-aware flows only | Non-solution flows are not listed. Add the flow to a solution first. |
| Maker access required | The maker must have access to the flow and all its underlying connections. |
| Dataverse permissions at runtime | End users need sufficient Dataverse permissions to invoke flows (App Opener security role or equivalent). |
| Manual refresh on flow changes | Re-run `add-flow` after any change to the flow definition. |
| npm CLI only | `pac code` does not have flow commands. Always use `npx power-apps`. |
