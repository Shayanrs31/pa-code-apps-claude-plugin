# Mockup Standards

PA Plan renders an inline UI mockup as section 6 of every plan. This file defines what to show and how to show it.

---

## When to render mockups

Always render at least one mockup per plan. Render one mockup per distinct screen type (list, detail, form). Cap at three mockups per plan to keep context clean.

---

## How to render

Call `mcp__visualize__read_me` first (modules: `["mockup"]`). Then call `mcp__visualize__show_widget` with an HTML or SVG widget.

```typescript
// Always call read_me before the first show_widget
mcp__visualize__read_me({ modules: ["mockup"] });

// Then render
mcp__visualize__show_widget({
  title: "records_list_screen",
  loading_messages: ["Sketching the layout", "Drawing the table", "Adding status badges"],
  widget_code: `<div>...html mockup...</div>`
});
```

---

## Mockup style rules

- Use CSS variables for all colours. Never hardcode hex.
- Default to dark theme (dark sidebar, card background for content area)
- Match the app's font size and spacing conventions (monospace font, compact rows)
- Do not make mockups interactive beyond basic hover states. They are static previews.
- Include realistic-but-fictional data appropriate to the app's domain

---

## What to show per screen type

### List screen
Show:
- Page header with title and primary action button
- 3-4 metric/summary cards at the top
- Table with 4-6 columns including a status badge column
- Search input in the card header
- At least 4-5 data rows with realistic values
- Row hover state

### Detail or review screen
Show:
- Back button and page title with secondary action buttons
- Summary header card with key metadata fields in a grid
- Main comparison or data card
- Status badges per line
- Action buttons inline per row
- A footer total row

### Form or input screen
Show:
- Page header with title
- Form card with labelled fields (text inputs, dropdowns, date pickers)
- A confidence indicator or validation message where relevant
- Submit and Cancel buttons

### Dashboard or home screen
Show:
- Metric row at top (4 KPI cards)
- One or two chart/summary panels
- A recent items list or feed below

---

## Mockup labels

Label each mockup clearly in the plan text before calling `show_widget`:

```markdown
### Screen 1: Records list

The main entry point. Users see all records with status...

[mockup rendered below]
```

---

## Section 6 format in plan output

```markdown
## 6. UI Mockups

The screens below represent the proposed layout. Final styling follows the app's CSS variable theme.

### Screen 1: {Name}
{description sentence}
{show_widget call}

### Screen 2: {Name}
{description sentence}
{show_widget call}
```
