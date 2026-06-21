---
name: add-excel
description: Add Excel Online (Business) as a data source to the current code app. Use when the app needs to read or write data in a named Excel table stored in OneDrive or SharePoint.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash
model: sonnet
---

# Skill: add-excel

Add Excel Online (Business) as a data source for reading or writing spreadsheet data.

---

## Workflow

### Step 1: Get the connection ID

```powershell
pwsh -NoProfile -Command "pac connection list"
```

Find the Excel Online (Business) connection ID. If missing: direct the user to create one at https://make.powerapps.com → Connections → Excel Online (Business).

### Step 2: Discover the workbook

```powershell
pwsh -NoProfile -Command "pac code list-datasets -a excelonlinebusiness --connectionReference <connection-id>"
```

Ask the user which drive/site hosts the workbook.

### Step 3: Discover tables in the workbook

```powershell
pwsh -NoProfile -Command "pac code list-tables -a excelonlinebusiness --connectionReference <connection-id> --dataset '<drive-id>' --file '<path/to/workbook.xlsx>'"
```

Excel connector requires the data to be in a named Table (Insert → Table in Excel), not just a range. Ask the user to confirm the table name.

### Step 4: Add the data source

```powershell
pwsh -NoProfile -Command "pac code add-data-source --action excelonlinebusiness --connectionReference <connection-id> --dataset '<drive-id>' --table '<table-name>'"
```

### Step 5: Verify and build

Check for `src/generated/services/ExcelOnlineService.ts`. Then:

```powershell
npm run build
```

### Step 6: Wire the hook

```typescript
// src/hooks/useExcelTable.ts
import { useState, useEffect } from 'react';
import { ExcelOnlineService } from '../generated/services/ExcelOnlineService';

const DRIVE = '<drive-id>';
const FILE = '<path/to/workbook.xlsx>';
const TABLE = '<table-name>';

export function useExcelTable() {
  const [rows, setRows] = useState<any[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    ExcelOnlineService.GetItems({ source: DRIVE, drive: DRIVE, file: FILE, table: TABLE })
      .then(result => setRows(result.value ?? []))
      .finally(() => setLoading(false));
  }, []);

  return { rows, loading };
}
```

### Step 7: Update memory bank

Record the Excel connector and workbook location in `memory-bank.md`.
