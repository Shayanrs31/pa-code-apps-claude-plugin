---
name: add-connector
description: Add any Power Platform connector as a data source when no specific add-* skill exists for it. Handles connector discovery, pac code add-data-source, and hook wiring for custom or uncommon connectors.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Grep
model: sonnet
---

# Skill: add-connector

Add a generic Power Platform connector as a data source. Use this when no specific `/add-*` skill exists for the connector you need.

---

## Workflow

### Step 1: Identify the connector

Ask the user: "Which connector do you want to add? Give me the name as it appears in Power Apps (e.g., 'Azure Key Vault', 'HTTP with Azure AD', 'Custom connector name')."

### Step 2: Find the connection ID

```powershell
pwsh -NoProfile -Command "pac connection list"
```

Find the connection ID for the named connector.

If it does not exist, direct the user to https://make.powerapps.com → Connections → New connection and ask them to create it first.

### Step 3: Find the action name

```powershell
pwsh -NoProfile -Command "pac code list-datasets --help"
```

The `--action` flag value is the connector's internal name (not its display name). Common examples:
- SharePoint → `sharepointonline`
- Dataverse → `commondataservice`
- Teams → `teams`
- OneDrive → `onedriveforbusiness`
- Excel → `excelonlinebusiness`

For custom connectors or uncommon connectors, run:
```powershell
pwsh -NoProfile -Command "pac code add-data-source --help"
```

and ask the user to provide the action name from their connector's API reference.

### Step 4: Discover datasets (if applicable)

Some connectors require a dataset (site URL, drive, org) before listing tables:
```powershell
pwsh -NoProfile -Command "pac code list-datasets -a <action-name> --connectionReference <connection-id>"
```

If the command returns nothing or an error, the connector may not need a dataset. Proceed to Step 5 without `--dataset`.

### Step 5: Add the data source

```powershell
pwsh -NoProfile -Command "pac code add-data-source --action <action-name> --connectionReference <connection-id>"
```

Add `--dataset` and `--table` flags if the connector requires them.

### Step 6: Verify and build

Check for the generated service file in `src/generated/services/`. Then:

```powershell
npm run build
```

### Step 7: Wire the hook

Read the generated service file to understand available methods:
```bash
grep "^\s*async\|^\s*static" src/generated/services/*.ts
```

Write a hook in `src/hooks/` that calls the service methods. Never call them directly from components.

### Step 8: Update memory bank

Record the connector added in `memory-bank.md`.
