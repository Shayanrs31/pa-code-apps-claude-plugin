---
name: add-onedrive
description: Add OneDrive for Business as a data source to the current code app. Use when the app needs to upload, download, or browse files and folders.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash
model: sonnet
---

# Skill: add-onedrive

Add OneDrive for Business as a data source for uploading, downloading, or managing files.

---

## Workflow

### Step 1: Get the connection ID

```powershell
pwsh -NoProfile -Command "pac connection list"
```

Find the OneDrive for Business connection ID. If missing: direct the user to create one at https://make.powerapps.com → Connections → OneDrive for Business.

### Step 2: Add the data source

```powershell
pwsh -NoProfile -Command "pac code add-data-source --action onedriveforbusiness --connectionReference <connection-id>"
```

### Step 3: Verify and build

Check for `src/generated/services/OneDriveService.ts`. Then:

```powershell
npm run build
```

### Step 4: Wire the hook

Common use cases:

**List files in a folder:**
```typescript
// src/hooks/useOneDriveFiles.ts
import { useState, useEffect } from 'react';
import { OneDriveService } from '../generated/services/OneDriveService';

export function useOneDriveFiles(folderPath: string) {
  const [files, setFiles] = useState<any[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    OneDriveService.ListFolder({ id: folderPath })
      .then(result => setFiles(result.value ?? []))
      .finally(() => setLoading(false));
  }, [folderPath]);

  return { files, loading };
}
```

**Get a download URL:**
```typescript
const url = await OneDriveService.GetFileContent({ id: fileId });
```

### Step 5: Update memory bank

Record the OneDrive connector in `memory-bank.md`.
