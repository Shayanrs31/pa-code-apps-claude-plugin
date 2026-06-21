---
name: create-code-app
description: Create a new Power Apps code app from scratch. Invokes PA Plan for requirements and mockups, then PA Code for scaffolding, implementation, and deployment.
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, EnterPlanMode, ExitPlanMode, AskUserQuestion, mcp__visualize__read_me, mcp__visualize__show_widget
model: sonnet
---

# Skill: create-code-app

Create a new Power Apps code app from scratch.

---

## When invoked

The user wants to build a new code app. They may have described an idea, or they may have invoked this skill with no context.

---

## Workflow

This skill invokes two agents in sequence:

1. **PA Plan** - gathers requirements, designs the plan, renders UI mockups, creates `memory-bank.md`
2. **PA Code** - scaffolds, initialises, implements, and deploys

### Step 1: Hand off to PA Plan

Invoke PA Plan. Pass any context the user has already provided (app name, purpose, data sources mentioned) so the agent can skip redundant questions.

PA Plan will:
- Ask what the user wants to build (if not already known)
- Ask about data requirements and map to connectors
- Ask about UI and screen requirements
- Enter plan mode and present a plan with UI mockups as section 6
- Exit plan mode after user approval
- Write `memory-bank.md`

Do not proceed to Step 2 until the user has explicitly approved the plan.

### Step 2: Hand off to PA Code

Invoke PA Code. Pass:
- Project folder path (or let PA Code ask the user)
- Environment ID from the plan
- List of data sources to add (from the plan)

PA Code will run the full scaffold → init → build → deploy workflow.

---

## Prerequisites check (done by PA Code, but surfaced here)

The user needs:
- Node.js v22+
- pac CLI installed (Power Platform CLI)
- A Power Platform environment with maker permissions
- (Optional) Git

If any prerequisite is missing, surface the error immediately before doing anything else.

---

## Output

On completion, provide:

```
{App Name} is live.

App: {name} {version}
Environment: {environment-name} ({environment-id})
URL: https://apps.powerapps.com/play/e/{env-id}/app/{app-id}
Project: {project-folder}

What was built:
- {screen 1}: {description}
- {screen 2}: {description}
- Data: {connectors used, or "mocked in src/services/mock.ts"}

To redeploy:
  npm run build
  pac code push (run from the project folder in PowerShell)

What you can add next:
- /add-dataverse - store and manage custom business data
- /add-sharepoint - work with SharePoint lists or documents
- /add-connector - connect to any other service
- /document - generate a technical design document or user guide
```
