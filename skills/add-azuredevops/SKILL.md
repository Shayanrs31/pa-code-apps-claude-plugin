---
name: add-azuredevops
description: Add Azure DevOps as a data source to the current code app. Use when the app needs to read work items, builds, pipelines, or sprints.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash
model: sonnet
---

# Skill: add-azuredevops

Add Azure DevOps as a data source for reading work items, builds, or pipelines.

---

## Workflow

### Step 1: Get the connection ID

```powershell
pwsh -NoProfile -Command "pac connection list"
```

Find the Azure DevOps connection ID. If missing: direct the user to create one at https://make.powerapps.com → Connections → Azure DevOps.

### Step 2: Discover organisations

```powershell
pwsh -NoProfile -Command "pac code list-datasets -a visualstudioteamservices --connectionReference <connection-id>"
```

Ask the user which organisation to connect.

### Step 3: Discover projects

```powershell
pwsh -NoProfile -Command "pac code list-tables -a visualstudioteamservices --connectionReference <connection-id> --dataset '<org>'"
```

Ask the user which project and data (work items, builds, etc.) to connect.

### Step 4: Add the data source

```powershell
pwsh -NoProfile -Command "pac code add-data-source --action visualstudioteamservices --connectionReference <connection-id> --dataset '<org>' --table '<project>'"
```

### Step 5: Verify and build

Check for `src/generated/services/AzureDevOpsService.ts`. Then:

```powershell
npm run build
```

### Step 6: Wire the hook

```typescript
// src/hooks/useWorkItems.ts
import { useState, useEffect } from 'react';
import { AzureDevOpsService } from '../generated/services/AzureDevOpsService';

const ORG = '<your-org>';
const PROJECT = '<your-project>';

export function useWorkItems(type: string = 'Task') {
  const [items, setItems] = useState<any[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    AzureDevOpsService.GetWorkItemsByWiql({
      organization: ORG,
      project: PROJECT,
      query: `SELECT [System.Id], [System.Title], [System.State] FROM WorkItems WHERE [System.WorkItemType] = '${type}' ORDER BY [System.Id] DESC`,
    })
      .then(result => setItems(result.workItems ?? []))
      .finally(() => setLoading(false));
  }, [type]);

  return { items, loading };
}
```

### Step 7: Update memory bank

Record the Azure DevOps connector, org, and project in `memory-bank.md`.
