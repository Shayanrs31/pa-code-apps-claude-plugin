---
mode: agent
tools: [codebase, terminalLastCommand, runCommands, readFile, writeFile, editFile]
description: Add Power Automate cloud flows (preview) to the current Power Apps code app
---

# Add Power Automate Flows (Preview)

You are adding a Power Automate cloud flow to the current code app.

Reference: https://learn.microsoft.com/en-us/power-apps/developer/code-apps/how-to/add-flows

> This feature is preview as of June 2026. npm CLI only - not available via pac code.

---

## Prerequisites

- `@microsoft/power-apps` version 1.1.1 or later
- Flow must use the PowerApps trigger (manual instant flow)
- Flow must belong to a solution

---

## Step 1: List available flows

```powershell
npx power-apps list-flows
```

To filter by name:

```powershell
npx power-apps list-flows --search <keyword>
```

Copy the Flow ID.

---

## Step 2: Add the flow

```powershell
npx power-apps add-flow --flow-id <flow-id>
```

Generated files:
- `src/services/{FlowName}Service.ts`
- `src/models/{FlowName}Model.ts`
- `schemas/logicflows/{FlowName}.Schema.json` (do not edit)

---

## Step 3: Build to verify

```powershell
npm run build
```

---

## Step 4: Wire a hook

```typescript
// src/hooks/use{FlowName}.ts
import { useState } from 'react';
import { {FlowName}Service } from '../services/{FlowName}Service';

export function use{FlowName}() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function trigger(input: Record<string, unknown>) {
    setLoading(true);
    setError(null);
    try {
      const result = await {FlowName}Service.Run(input);
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

---

## Updating a flow

```powershell
npx power-apps add-flow --flow-id <flow-id>
```

Re-running with the same flow ID picks up definition changes without manual cleanup.

---

## Removing a flow

```powershell
npx power-apps remove-flow --flow-name <FlowName>
```

---

## Limitations

- PowerApps trigger only
- Solution-aware flows only
- npm CLI only (`pac code` does not have flow commands)
- Re-run `add-flow` after any flow definition change
