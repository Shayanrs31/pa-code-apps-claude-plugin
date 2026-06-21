---
name: add-datasource
description: Add a data source to the current code app. Asks what the app needs to do with data, then routes to the correct add-* skill. Use this when unsure which connector to pick.
user-invocable: true
allowed-tools: Read, Write, Bash, AskUserQuestion
model: sonnet
---

# Skill: add-datasource

Add a data source to the current code app.

This is a router skill. It asks what the app needs to do and routes to the correct `/add-*` skill.

---

## When invoked

The user wants to connect the app to a data source, but has not specified which connector to use.

---

## Step 1: Ask what the app needs to do

> "What does your app need to do with data? Describe it in plain language. For example: 'store and manage project records', 'read items from a SharePoint list', or 'send emails when a task is approved'."

Wait for their answer.

---

## Step 2: Route to the correct skill

Based on their answer, route to one of:

| If the app needs to... | Route to |
|---|---|
| Store and manage custom business records (tables, forms, CRUD) | `/add-dataverse` |
| Read or write SharePoint lists or document libraries | `/add-sharepoint` |
| Send or read Teams messages, create channels | `/add-teams` |
| Send emails, read inbox, manage calendar, manage contacts | `/add-office365` |
| Upload, download, or manage files and folders | `/add-onedrive` |
| Read or write Excel spreadsheet data | `/add-excel` |
| Track work items, bugs, pipelines, or sprints | `/add-azuredevops` |
| Call a custom REST API or Azure Function | `/add-connector` |

If the answer spans multiple connectors, route to each in sequence.

---

## Step 3: Confirm the route

Before routing, confirm with the user:

> "Based on what you described, I'd recommend adding {connector name}. This maps to the `/add-{connector}` skill. Does that sound right?"

If the user corrects you, adjust accordingly.

---

## No match

If the data source described does not match any of the connectors above, invoke `/add-connector` and let it handle the generic case.
