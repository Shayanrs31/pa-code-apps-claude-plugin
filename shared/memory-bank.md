# Memory Bank - Template

Copy this template into your project root as `memory-bank.md` when you start a new code app. PA Code creates it automatically at the end of the scaffold step.

---

```markdown
# {App Name} - Memory Bank

## Project

- **Path:** {absolute path to project folder}
- **App name:** {display name}
- **Environment:** {environment name} ({environment-id})
- **App URL:** https://apps.powerapps.com/play/e/{env-id}/app/{app-id}
- **Version:** v0.1.0

## Completed Steps

- [ ] Prerequisites validated
- [ ] Scaffold (npx degit)
- [ ] Initialize (pac code init)
- [ ] Baseline deploy
- [ ] Data sources added
- [ ] App implemented
- [ ] Final deploy

## Data Sources

List each connector added via `pac code add-data-source`:

- {Connector name}: {what table or dataset was connected}

## Pages

List each page in `src/pages/`:

- {PageName}.tsx: {one-line description}

## Hooks

List each hook in `src/hooks/`:

- {hookName}.ts: {what data it manages}

## Mock Services

If using `src/services/mock.ts` before real connectors are wired:

- {ServiceName}.{method}(): {what it returns}

When real connectors land, mock calls in hooks are replaced with generated service calls.

## Backend API Notes

If the app consumes a custom backend API (not a standard connector), record the key details here:

- Base URL: {base URL}
- Key endpoints:
  - `GET /items` - returns list of records
  - `GET /items/{id}` - returns single record

Freeze this contract before building real connectors.

## Known Issues Encountered

Note any project-specific issues here (not covered by `shared/known-issues.md`):

- {issue}: {fix applied}

## Next Steps

- {what to do next}
- {connector to add}
- {feature to implement}

## Auth and Roles

If role-based access is planned:

- Roles: {list roles}
- Claims: {how roles are passed to the app}
- Status: {planned / in progress / done}
```

---

## Instructions for agents

- Create this file at the end of the scaffold step
- Update it at the end of every significant step. Do not wait until the end of the session.
- If a session ends unexpectedly, the next session must read this file first to resume from the correct step
- Check off completed steps using `[x]` as they are done
- Never delete this file. It is the source of truth for project state across sessions.
