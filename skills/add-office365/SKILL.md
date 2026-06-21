---
name: add-office365
description: Add Office 365 Outlook as a data source to the current code app. Use when the app needs to send emails, read inbox, or manage calendar.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash
model: sonnet
---

# Skill: add-office365

Add Office 365 Outlook as a data source for sending emails, reading inbox, or managing calendar.

---

## Workflow

### Step 1: Get the connection ID

```powershell
pwsh -NoProfile -Command "pac connection list"
```

Find the Office 365 Outlook connection ID. If missing: direct the user to create one at https://make.powerapps.com → Connections → Office 365 Outlook.

### Step 2: Add the data source

```powershell
pwsh -NoProfile -Command "pac code add-data-source --action office365 --connectionReference <connection-id>"
```

### Step 3: Verify and build

Check for `src/generated/services/Office365Service.ts`. Then:

```powershell
npm run build
```

### Step 4: Wire the hook

Common use cases:

**Send an email:**
```typescript
// src/hooks/useEmail.ts
import { Office365Service } from '../generated/services/Office365Service';

export function useEmail() {
  async function send(to: string, subject: string, body: string) {
    await Office365Service.SendEmail({
      emailMessage: {
        To: to,
        Subject: subject,
        Body: body,
      }
    });
  }
  return { send };
}
```

**Read inbox:**
```typescript
const messages = await Office365Service.GetEmails({ folderPath: 'Inbox', top: 20 });
```

### Step 5: Update memory bank

Record the Office 365 connector in `memory-bank.md`.
