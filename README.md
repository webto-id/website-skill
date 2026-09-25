# webto-website

An AI skill (Claude Code, Cursor, and other agents that read `SKILL.md`) that builds and manages a small-business **website on [webto.id](https://webto.id)** for you: it interviews you about your business, assembles a `site.json` from webto's platform section variants, creates the site as a draft over the webto MCP server, publishes it on `<subdomain>.wpage.id` when you approve, and later edits it without disturbing what you changed in the dashboard.

It never invents facts: prices, names, addresses, opening hours, legal details, testimonials and statistics come only from you.

## Install

```bash
# Claude Code (project-level)
git clone https://github.com/webto-id/website-skill .claude/skills/webto-website

# Claude Code (global): same command into ~/.claude/skills/
```

## Connect the webto MCP server

1. In the webto dashboard: **Settings → Security → Token akses (agen AI)** → create a token and tick **Website: Baca / Tulis** (and **Publikasikan** if the agent may publish). It is shown once.
2. Put it in your environment as `WEBTO_TOKEN`, and add to the project's `.mcp.json`:

```json
{
  "mcpServers": {
    "webto": {
      "type": "http",
      "url": "https://api.webto.id/mcp",
      "headers": { "Authorization": "Bearer ${WEBTO_TOKEN}" }
    }
  }
}
```

Never paste the token into the chat. Revoke it in the same settings page when you are done.

## Use

> Buatkan website untuk usaha saya di webto.

> Tambahkan halaman "Menu" di website webto saya, dan ganti jam buka jadi 08.00–21.00.

Local check before anything is sent (needs `@webto-id/variant-check` ≥ 0.1.37):

```bash
npx @webto-id/variant-check site ./my-site
```

## Contents

| Path | What it is |
|---|---|
| `SKILL.md` | The rules and both workflows: a new site, and changing an existing one |
| `references/interview.md` | What to ask per kind of business, and which section/field each answer fills |
| `references/site-json.md` | The `site.json` format: ids, images, theme, business profile |
| `references/mcp.md` | Every MCP tool, scopes, dry-run → confirm, limits, the lock file |
| `references/catalog.md` | Every section type and platform variant with its content fields (generated) |
| `examples/kopi-senja/site.json` | A complete small-business site that passes `variant-check site` (fictional facts) |

## License

MIT
