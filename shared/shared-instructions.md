# Shared Instructions

All agents in this plugin must follow these rules before doing anything else.

---

## 0. MCP tools

The `microsoft-learn` MCP server is available when configured in `.vscode/mcp.json`. Use it to look up official documentation before writing any SDK, connector, or pac CLI code.

See `shared/mcp-tools.md` for setup, tool names, and key documentation pages.

---

## 1. Memory bank first

Before any task, check for `memory-bank.md` in the project root. If it exists, read it and skip steps already marked complete. Update it at the end of every significant step. Do not wait until the end of the session.

See `shared/memory-bank.md` for the full template.

---

## 2. Windows CLI compatibility

The `pac` CLI is a Windows executable. It is NOT on the bash PATH.

- Always run `pac` commands via PowerShell: `pwsh -NoProfile -Command "pac ..."`
- Never run `pac` in a Bash tool block
- Never use `-e` as a short flag for `--environment`. The pac CLI 2.4.1+ uses the long form only.
- `pac --version` is not valid. Use `pac help` to confirm the CLI is installed.

---

## 3. Connector-first rule

Before writing any custom fetch, axios, or raw HTTP call in a code app, check whether a Power Platform connector exists for the data source.

- If a connector exists: use `pac code add-data-source` and call the generated service
- If no connector exists: use an HTTP connector to reach the backend API

Never bypass the connector layer with direct fetch calls in production code.

---

## 4. Safety guardrails

- Never run `pac code push` (deploy) without explicit user confirmation in that turn
- Never edit files under `src/generated/`. They are regenerated on each `pac code add-data-source` and changes will be lost.
- Never hardcode user identity. Always use `getContext()` via `useCurrentUser`.
- Never run `pac auth clear` unless the user has explicitly asked to reset their auth profile
- Check `shared/known-issues.md` before running any `pac code` command

---

## 5. Template scaffolding

When creating a new code app:

- Use `npx degit microsoft/PowerAppsCodeApps/templates/vite {folder} --force` and not `git clone`
- Use `--force` to overwrite if the directory already has files
- Verify `package.json` exists after scaffold before proceeding to `pac code init`

---

## 6. pac code init

```powershell
pwsh -NoProfile -Command "pac code init --displayName '{app-name}' --environment <environment-id>"
```

After init, read `power.config.json` and verify `environmentId` matches the selected environment. Correct it if mismatched.

---

## 7. OData safety

All Dataverse OData queries must include `$top`. Never issue unbounded queries:

```typescript
// Bad
DataverseService.getAll()

// Good
DataverseService.retrieveMultipleRecords('entity', '?$top=250&$select=col1,col2')
```

---

## 8. Build before deploy

Always run `npm run build` and confirm it passes before running `pac code push`. Never deploy a broken build.
