---
name: document
description: Generate documentation for the current code app. Produces a Technical Design Document, User Guide, Handover Pack, or Architecture Overview as Markdown, rendered in chat or saved to a file.
user-invocable: true
allowed-tools: Read, Write, Glob, Grep
model: sonnet
---

# Skill: document

Generate documentation for the current code app.

---

## When invoked

The user wants to produce a document: a technical design doc, user guide, handover pack, or architecture overview. They may or may not have specified the type.

---

## Workflow

### Step 1: Read memory bank

Read `memory-bank.md` in the project root for current app state. If it does not exist, ask the user to describe the app.

### Step 2: Confirm document type and audience

If not already specified, ask:

> "What type of document would you like?
> - Technical Design Document: for developers and architects
> - User Guide: for end users and business stakeholders
> - Handover Pack: for a team taking over the app
> - Architecture Overview: for IT leadership or clients"

Also ask: "Should I render this in chat, or save it as a `.md` file in the project?"

### Step 3: Hand off to PA Document

Invoke PA Document with:
- Document type
- Output format (in-chat or save to file)
- Current app state from `memory-bank.md`

PA Document will read the source files as needed and produce the document.

### Step 4: File save confirmation

If the user wants a saved file, PA Document will write to `docs/{document-type}.md` in the project root. It will confirm the file path before writing.

---

## Available document types

See `agents/pa-document/AGENT.md` for full section structure of each document type.

| Type | Audience | Key content |
|---|---|---|
| Technical Design Document | Developers, architects | Architecture, API contract, data model, hooks, deploy steps |
| User Guide | End users | Screen walkthroughs, how-to tasks, troubleshooting |
| Handover Pack | Maintenance developers | Tech stack, how to run locally, deploy process, known issues |
| Architecture Overview | IT leadership, clients | High-level diagram, component descriptions, security, next phase |
