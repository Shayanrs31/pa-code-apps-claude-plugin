---
mode: agent
tools: [codebase, readFile, writeFile]
description: Generate a Technical Design Document, User Guide, Handover Pack, or Architecture Overview for the current Power Apps code app
---

# Document Power Apps Code App

You are generating documentation for the current Power Apps code app.

---

## Step 1: Read memory bank

Read `memory-bank.md` in the project root. If it does not exist, ask the user to describe the app.

---

## Step 2: Confirm document type

Ask:

> "What type of document would you like?
> - Technical Design Document: for developers and architects
> - User Guide: for end users and business stakeholders
> - Handover Pack: for a team taking over the app
> - Architecture Overview: for IT leadership or clients"

Also ask: "Should I render this in chat, or save it as a `.md` file in the project?"

---

## Step 3: Generate

Read the relevant source files before writing. Do not invent details not present in `memory-bank.md` or the source files.

### Technical Design Document

Sections: Executive summary, Architecture, Data model, Backend API contract, Hook and service layer, Pages, Deployment, Known issues, Next steps.

### User Guide

Sections: Overview, How to access, Screen walkthroughs, Common tasks, Troubleshooting.

### Handover Pack

Sections: App summary, Tech stack, Folder structure, How to run locally, How to deploy, Data sources, Known issues, Open items.

### Architecture Overview

Sections: Solution summary, High-level architecture diagram (text-based), Key components, Security, Scalability notes, Next phase recommendations.

---

## Step 4: Save if requested

Write to `docs/{document-type}.md` in the project root. Confirm the file path before writing.

File naming:
- Technical Design Document: `docs/technical-design.md`
- User Guide: `docs/user-guide.md`
- Handover Pack: `docs/handover.md`
- Architecture Overview: `docs/architecture.md`

---

## Standards

- Markdown only. Never produce Word or PDF output.
- Use `#` for major sections, `##` for sub-sections.
- Use tables for structured data (data model, hook list, page list).
- Use fenced code blocks for commands and code snippets.
- Do not use em dashes. Use a colon or restructure the sentence.
- Do not use emojis.
