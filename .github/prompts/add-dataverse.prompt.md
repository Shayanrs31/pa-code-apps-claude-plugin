---
mode: agent
tools: [codebase, terminalLastCommand, runCommands, readFile, writeFile, editFile]
description: Add a Dataverse table as a data source to the current Power Apps code app
---

# Add Dataverse Data Source

You are adding a Dataverse table to the current Power Apps code app.

Follow `.github/instructions/known-issues.md` before running any pac command.

---

## Step 1: Check connections

```powershell
pwsh -NoProfile -Command "pac connection list"
```

Find the Dataverse connection ID. If none exists, direct the user to https://make.powerapps.com → Connections → New connection → Microsoft Dataverse.

---

## Step 2: List available tables

```powershell
pwsh -NoProfile -Command "pac code list-tables -a commondataservice --connectionReference <connection-id>"
```

Show the user the available tables. Ask which table(s) to connect.

---

## Step 3: Add the data source

```powershell
pwsh -NoProfile -Command "pac code add-data-source --action commondataservice --connectionReference <connection-id> --table <table-logical-name>"
```

Run once per table.

---

## Step 4: Verify generated files

Check that these were created:
- `src/generated/models/{TableName}Model.ts`
- `src/generated/services/{TableName}Service.ts`

Read the model file to identify exact column names and types. Never guess column names.

---

## Step 5: Fix MSCRM error if it occurs

If `npm run build` fails with a TypeScript error about MSCRM or Xrm, follow Issue 3 in `.github/instructions/known-issues.md`.

---

## Step 6: Build to verify

```powershell
npm run build
```

Fix any errors before proceeding. Do not deploy yet.

---

## Step 7: Wire the hook

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

---

## Step 8: Update memory bank

Record the Dataverse connector and table(s) added in `memory-bank.md`.

---

## Dataverse gotchas

- Picklist fields: read as `number`, write by setting the numeric option value
- Virtual fields: read-only, exclude from write payloads
- Lookups: set as `{ entityType: 'tablename', id: 'guid' }`
- Always include `$select` in retrieve calls
- Entity prefix (`cr{nnn}_`) varies per environment. Read it from the generated model file.
