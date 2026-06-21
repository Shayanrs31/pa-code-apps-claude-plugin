---
name: list-connections
description: List all available Power Platform connections in the current environment. Use before adding a data source to find the connection ID needed for pac code add-data-source.
user-invocable: true
allowed-tools: Bash
model: sonnet
---

# Skill: list-connections

List all available Power Platform connections for the current environment.

---

## When invoked

The user wants to see what connections are available before adding a data source, or needs to find a connection ID for a specific connector.

---

## Workflow

### Step 1: List connections

```powershell
pwsh -NoProfile -Command "pac connection list"
```

The output shows all connections in the active environment, including:
- Connection ID
- Display name
- Connector type
- Status (Connected / Error)

### Step 2: Filter and display

Present the results in a clean table:

| Connection ID | Name | Connector | Status |
|---|---|---|---|
| conn-xyz | My SharePoint | SharePoint Online | Connected |
| conn-abc | Office 365 | Office 365 Outlook | Connected |

### Step 3: Advise next steps

Tell the user:
- Which connections can be used with `/add-*` skills
- If a needed connector is missing, they need to create the connection first at https://make.powerapps.com → Connections → New connection
- The connection ID is needed when running `pac code add-data-source --connectionReference <id>`

---

## If no connections exist

```
No connections found in this environment.

To create a connection:
1. Go to https://make.powerapps.com
2. Navigate to Connections → New connection
3. Search for your connector and follow the auth flow
4. Come back and run /list-connections again to get the connection ID
```
