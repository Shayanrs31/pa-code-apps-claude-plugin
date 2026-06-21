---
name: add-sharepoint
description: Add SharePoint Online as a data source to the current code app. Use when the app needs to read or write SharePoint lists or document libraries.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash
model: sonnet
---

# Skill: add-sharepoint

Add a SharePoint Online list or document library as a data source.

---

## Workflow

### Step 1: Get the connection ID

```powershell
pwsh -NoProfile -Command "pac connection list"
```

Find the SharePoint Online connection ID. If missing: direct the user to create one at https://make.powerapps.com → Connections → SharePoint.

### Step 2: Discover available sites

```powershell
pwsh -NoProfile -Command "pac code list-datasets -a sharepointonline --connectionReference <connection-id>"
```

Show the user the available sites. Ask which site to use.

### Step 3: Discover available lists

```powershell
pwsh -NoProfile -Command "pac code list-tables -a sharepointonline --connectionReference <connection-id> --dataset '<site-url>'"
```

Show the lists and document libraries. Ask which list(s) to connect.

### Step 4: Add the data source

```powershell
pwsh -NoProfile -Command "pac code add-data-source --action sharepointonline --connectionReference <connection-id> --dataset '<site-url>' --table '<list-name>'"
```

Run once per list.

### Step 5: Verify and build

Check for `src/generated/services/SharePointOnlineService.ts`. Then:

```powershell
npm run build
```

### Step 6: Wire the hook

```typescript
// src/hooks/use{ListName}.ts
import { useState, useEffect } from 'react';
import { SharePointOnlineService } from '../generated/services/SharePointOnlineService';

const SITE = 'https://yourorg.sharepoint.com/sites/YourSite';
const LIST = 'Your List Name';

export function use{ListName}() {
  const [items, setItems] = useState<any[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    SharePointOnlineService.GetItems({ dataset: SITE, table: LIST })
      .then(result => setItems(result.value ?? []))
      .finally(() => setLoading(false));
  }, []);

  return { items, loading };
}
```

### Step 7: Update memory bank

Record the SharePoint connector, site URL, and list(s) added in `memory-bank.md`.
