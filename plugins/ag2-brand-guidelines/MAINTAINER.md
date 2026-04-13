# Maintainer Guide — AG2 Brand Guidelines Plugin

This guide is for **plugin maintainers only** — the people who update the
plugin when the design team changes tokens in Figma. Regular users do not
need this file or Figma access.

---

## Prerequisites

- Access to the AG2 Figma file (viewer or editor)
- Claude Desktop with Figma MCP connected (built-in, just authorize on first use)

## Figma File Reference

- **File key**: `1fqtTB9astetM1bfkdEwN5`
- **URL**: https://www.figma.com/design/1fqtTB9astetM1bfkdEwN5/AG2---2026

---

## Sync Workflow

The plugin includes a maintainer-only skill called `figma-sync`. It is
**not auto-invoked** — users cannot accidentally trigger it.

### Running a Sync

Open Claude Desktop (with this plugin installed) and type:

```
/figma-sync
```

This runs the full 3-tier sync:
1. **Tier 1 — Token Check**: Compares colors, fonts, buttons against Figma
2. **Tier 2 — Structure Check**: Detects new/removed frames in known pages
3. **Tier 3 — Page Discovery**: Detects new/renamed/deleted pages

You can also run partial syncs:

```
/figma-sync colors
/figma-sync typography
/figma-sync buttons
/figma-sync structure
/figma-sync pages
/figma-sync icons
```

### What Happens

1. Claude connects to Figma via MCP and pulls the latest values
2. Claude compares against the current SKILL.md tokens
3. Claude shows you a diff table (old vs new values)
4. **You confirm** before any changes are made
5. Claude updates SKILL.md + reference files (css-template, components, etc.)
6. You commit and push — users get the update on next plugin sync

**IMPORTANT**: Claude will never update the plugin silently. You always
review and confirm changes first.

---

## After Updating

After syncing and confirming changes:

1. **Commit and push** — users pull the latest on next plugin sync.

---

## Reference Update Cascade

When a token changes, multiple files may need updating:

| Change Type | Files to Update |
|-------------|----------------|
| Color hex value | SKILL.md, css-template.md, components.md |
| Font family | SKILL.md, css-template.md |
| Font size / scale | SKILL.md, css-template.md |
| Button spec | SKILL.md, components.md |
| New page added | figma-sync SKILL.md (Pages table) |
| New component | figma-sync SKILL.md, possibly components.md |
| Landing pattern | landing-page-pattern.md |

---

## Security Notes

- The `figma-sync` skill has `disable-model-invocation: true` — Claude
  will never auto-invoke it based on user conversation.
- Even if a user manually types `/figma-sync`, they need Figma MCP
  connected AND access to the AG2 Figma file — it fails safely.
- No Figma file keys or node IDs are in the user-facing SKILL.md.
