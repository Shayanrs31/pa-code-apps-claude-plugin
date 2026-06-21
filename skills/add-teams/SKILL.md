---
name: add-teams
description: Add Microsoft Teams as a data source to the current code app. Use when the app needs to send messages, read channels, or list teams.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash
model: sonnet
---

# Skill: add-teams

Add Microsoft Teams as a data source for sending messages, creating channels, or reading Teams data.

---

## Workflow

### Step 1: Get the connection ID

```powershell
pwsh -NoProfile -Command "pac connection list"
```

Find the Microsoft Teams connection ID. If missing: direct the user to create one at https://make.powerapps.com → Connections → Microsoft Teams.

### Step 2: Add the data source

```powershell
pwsh -NoProfile -Command "pac code add-data-source --action teams --connectionReference <connection-id>"
```

### Step 3: Verify and build

Check for `src/generated/services/TeamsService.ts`. Then:

```powershell
npm run build
```

### Step 4: Wire the hook

Common use cases:

**Send a message to a channel:**
```typescript
// src/hooks/useTeamsNotify.ts
import { TeamsService } from '../generated/services/TeamsService';

export function useTeamsNotify() {
  async function sendMessage(teamId: string, channelId: string, message: string) {
    await TeamsService.SendMessage({ groupId: teamId, channelId, body: message });
  }
  return { sendMessage };
}
```

**List teams and channels:**
```typescript
import { TeamsService } from '../generated/services/TeamsService';
const teams = await TeamsService.GetTeams();
const channels = await TeamsService.GetChannels({ groupId: teamId });
```

### Step 5: Update memory bank

Record the Teams connector in `memory-bank.md`.
