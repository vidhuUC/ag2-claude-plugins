---
name: figma-sync
description: >
  Sync AG2 brand tokens from Figma. Maintainer-only workflow that pulls
  latest design tokens from the AG2 Figma file and updates the plugin.
  Requires Figma MCP connection and file access.
disable-model-invocation: true
metadata:
  version: "0.1.0"
---

# Figma Sync — Maintainer Only

This skill syncs the AG2 brand guidelines plugin with the source Figma file.
It detects token changes, structural changes, and new pages, then updates
the plugin files after maintainer confirmation.

**Requirements:**
- Figma MCP connected (Claude Desktop has this built-in)
- Access to the AG2 Figma file (viewer or editor)

---

## Figma File Reference

- **File key**: `1fqtTB9astetM1bfkdEwN5`
- **URL**: https://www.figma.com/design/1fqtTB9astetM1bfkdEwN5/AG2---2026
- **Root (all pages)**: node `0:1`

If the user provides a Figma branch URL like
`https://figma.com/design/:fileKey/branch/:branchKey/:fileName`,
use the **branchKey** as the fileKey instead.

### Pages

| Page | Node ID | Description |
|------|---------|-------------|
| UI (canvas root) | `0:1` | Top-level canvas containing all pages |
| DS (Design System) | `7:1396` | Colors, typography, buttons, tokens |
| Product | `7:808` | Product UI screens |
| Landing | `7:813` | Landing page layouts (light + dark) |
| Vera | `114:1773` | Vera page with landing layout |
| Agent OS | `145:1641` | 14 AgentOS app icons (500x500 each) |
| Assets | `2:405` | Raw assets, screenshots, components |

### Key Component Nodes

| Component | Node ID |
|-----------|---------|
| Color palette | `4:9624` |
| Typography | `7:1397` |
| Buttons | `7:1547` |
| Agent icons | `21:117` |
| Landing Light | `7:955` |
| Landing Dark | `7:1065` |

---

## 3-Tier Sync Workflow

Run all three tiers in order. Report results to the user after each tier.

### Tier 1 — Token Check (values changed on existing nodes)

This catches the most common designer edits: tweaking a color, font size,
spacing value, or button style.

**Steps:**

1. Call `get_design_context` on **color node `4:9624`**
2. Extract all hex color values from the response
3. Compare each value against the Color Palette table in
   `skills/ag2-brand-guidelines/SKILL.md`:
   - `--color-primary` should be `#F3FF9B`
   - `--color-soft-yellow` should be `#FCFFE9`
   - `--color-green` should be `#9BFFA3`
   - `--color-blue` should be `#9BDDFF`
   - `--color-pink` should be `#D59BFF`
   - `--color-dark` should be `#1D1C1B`
4. Call `get_design_context` on **typography node `7:1397`**
5. Extract font families and sizes from the response
6. Compare against the Typography section:
   - Display font should be `Alpha Lyrae`
   - Body font should be `Geist`
   - H1 should be `64px`, H2 `56px`, H3 `48px`, etc.
7. Call `get_design_context` on **buttons node `7:1547`**
8. Compare button properties (padding, radius, colors, font)

**Report format:**
```
## Token Sync Report

| Token | SKILL.md Value | Figma Value | Status |
|-------|---------------|-------------|--------|
| --color-primary | #F3FF9B | #F3FF9B | Match |
| --color-blue | #9BDDFF | #7CC8FF | CHANGED |
| H1 size | 64px | 72px | CHANGED |
```

### Tier 2 — Structure Check (new/removed frames in existing pages)

This catches when a designer adds a new section, new agent icons, or
removes a frame.

**Steps:**

1. For each known page in the Pages table, call `get_metadata` on its node ID
2. Count direct child frames and compare to the last known state
3. Look for new frame names/IDs not previously recorded
4. Look for missing frame IDs that were previously known

**Key nodes to check:**
- Landing page `7:813` — should have Light + Dark variants
- Agent icons `21:117` — should have 36 agents
- Agent OS `145:1641` — should have 14 App icons
- Design System `7:1396` — should have Typo, Color, Buttons sections

**Report format:**
```
## Structure Sync Report

| Page | Known Frames | Current Frames | Status |
|------|-------------|----------------|--------|
| Landing (7:813) | 5 | 5 | No change |
| Agent icons (21:117) | 36 | 38 | 2 NEW frames found |
```

### Tier 3 — Page Discovery (new/removed/renamed pages)

This catches entirely new pages the designer created.

**Steps:**

1. Call `get_metadata` on root node `0:1` with file key `1fqtTB9astetM1bfkdEwN5`
2. Parse top-level sections/frames
3. Compare against the Pages table:
   - Known: DS `7:1396`, Product `7:808`, Landing `7:813`,
     Vera `114:1773`, Agent OS `145:1641`, Assets `2:405`
4. Flag any new sections not in the list
5. Flag any known sections that are missing (deleted)
6. Flag any name mismatches (renamed pages)

**Report format:**
```
## Page Discovery Report

| Node ID | Name in Plugin | Name in Figma | Status |
|---------|---------------|---------------|--------|
| 7:1396 | DS | DS | Match |
| 200:5000 | (not tracked) | Blog | NEW PAGE |
```

---

## After Detection: Updating the Plugin

**CRITICAL**: When changes are detected, do NOT silently update. Always:

### Step 1 — Report Changes

Show the maintainer a clear summary of what changed, organized by tier.

### Step 2 — Ask for Confirmation

Ask: "The designer has made these changes in Figma. Would you like me
to update the plugin to match?"

Present the specific edits:
- Token changes: "Update `--color-primary` from `#F3FF9B` to `#E8F08A`"
- Structure changes: "Add new agent 'Planner' (node `200:5678`) to agent list"
- Page changes: "Add new page 'Blog' (node `200:5000`) to Pages table"

### Step 3 — Apply Updates (only after maintainer confirms)

Edit the plugin files directly:

**For token changes:**
- Update the Color Palette / Typography table in `skills/ag2-brand-guidelines/SKILL.md`
- Update `:root` variables in `skills/ag2-brand-guidelines/references/css-template.md`
- Update any hardcoded values in `skills/ag2-brand-guidelines/references/components.md`

**For structure changes:**
- Add new frame node IDs to the relevant section
- Update counts (e.g., "36 agents" -> "38 agents")
- Pull `get_design_context` on new nodes to get their full specs

**For page changes:**
- Add new row to the Pages table with node ID, name, description
- Pull `get_metadata` on the new page to understand its structure

### Step 4 — Verify

Read back the modified sections and confirm the edits are correct.
Report: "Plugin updated. These files were changed: SKILL.md (lines X-Y),
css-template.md (line Z)."

---

## Reference Update Cascade

When a token changes, multiple files may need updating:

| Change Type | Files to Update |
|------------|----------------|
| Color hex value | SKILL.md, css-template.md, components.md |
| Font family | SKILL.md, css-template.md |
| Font size | SKILL.md, css-template.md |
| Button style | SKILL.md, components.md |
| Spacing/radius | SKILL.md, css-template.md |
| New page | Pages table in this file only |
| New component | This file + possibly components.md |
| Landing pattern | landing-page-pattern.md |

---

## Syncing AgentOS Icons

```
/figma-sync icons
```

Call `get_design_context` on node `145:1641`. Each child frame is an
icon. Use `get_screenshot` on individual icon node IDs to fetch visuals.

---

## Quick Sync Arguments

| Maintainer types | Action |
|-----------------|--------|
| `/figma-sync` | Full sync — all 3 tiers |
| `/figma-sync colors` | Tier 1 — color node only |
| `/figma-sync typography` | Tier 1 — typography node only |
| `/figma-sync buttons` | Tier 1 — buttons node only |
| `/figma-sync structure` | Tier 2 — all known pages |
| `/figma-sync pages` | Tier 3 — page discovery |
| `/figma-sync icons` | AgentOS icons sync |
