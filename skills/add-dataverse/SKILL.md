---
name: add-dataverse
description: Add a Dataverse table as a data source to the current code app. Covers connection discovery, pac code add-data-source, MSCRM type fix, and hook wiring.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Grep
model: sonnet
---

# Skill: add-dataverse

Add a Dataverse table as a data source.

---

## Before you start

Read `shared/known-issues.md` before running any `pac` command.

---

## Workflow

### Step 1: Check connections

```powershell
pwsh -NoProfile -Command "pac connection list"
```

Find the Dataverse connection ID (connector type: `commondataservice` or `Dataverse`).

If no Dataverse connection exists: direct the user to https://make.powerapps.com → Connections → New connection → Microsoft Dataverse.

### Step 2: List available tables

```powershell
pwsh -NoProfile -Command "pac code list-tables -a commondataservice --connectionReference <connection-id>"
```

Show the user the available tables. Ask which table(s) to connect.

### Step 3: Add the data source

```powershell
pwsh -NoProfile -Command "pac code add-data-source --action commondataservice --connectionReference <connection-id> --table <table-logical-name>"
```

Run once per table.

### Step 4: Verify the generated files

After adding, check that these were created:
- `src/generated/models/{TableName}Model.ts`
- `src/generated/services/{TableName}Service.ts`

Read the model file to identify the exact column names and types. Never guess column names.

### Step 5: Fix MSCRM error (if it occurs)

If `npm run build` fails with a TypeScript error about MSCRM or Xrm types, follow the fix in `shared/known-issues.md` Issue 3.

### Step 6: Build to verify

```powershell
npm run build
```

Do not deploy yet. Fix any errors before proceeding.

### Step 7: Wire the hook

Show the user how to call the generated service from a hook:

```typescript
// src/hooks/use{TableName}.ts
import { useState, useEffect } from 'react';
import { {TableName}Service } from '../generated/services/{TableName}Service';
import type { {TableName}Model } from '../generated/models/{TableName}Model';

export function use{TableName}() {
  const [records, setRecords] = useState<{TableName}Model[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    {TableName}Service.retrieveMultipleRecords(
      '{table-logical-name}',
      '?$top=250&$select=col1,col2,col3'
    )
      .then(result => setRecords(result.value ?? []))
      .finally(() => setLoading(false));
  }, []);

  return { records, loading };
}
```

### Step 8: Update memory bank

Record the Dataverse connector and table(s) added in `memory-bank.md`.

---

## Dataverse gotchas (from shared/development-standards.md)

- Picklist fields: read as `number`, write by setting the numeric option value
- Virtual fields: read-only, exclude from write payloads
- Lookups: set as `{ entityType: 'tablename', id: 'guid' }`
- Always include `$select` in retrieve calls
- The entity prefix (`cr{nnn}_`) varies per environment. Read it from the generated model file.
