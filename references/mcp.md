# The webto MCP server

Everything this skill does to a live account goes through the `webto` MCP
server with the user's personal token. Without it you can still write and
validate `site.json` locally, but you cannot create or change a site — say so
and stop there.

## Setup (the user does this once)

1. webto.id dashboard → **Pengaturan → Keamanan → Token akses (agen AI)** →
   **Buat token**. Under **Website**, tick what the agent may do:
   - **Baca** — list sites, limits, subdomain check, export (always needed);
   - **Tulis** — create draft sites, update them, add pages, upload images;
   - **Publikasikan** — take a site live and back offline. Leave it unticked if
     the user wants to press Publish themselves; the skill then hands over.
2. Copy the token (shown once) into the environment as `WEBTO_TOKEN`
   (`setx WEBTO_TOKEN "…"` on Windows, `export WEBTO_TOKEN="…"` on
   macOS/Linux; open a new terminal after `setx`).
3. Add to the project's `.mcp.json`:

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

The token never goes in a file and never in the chat. If the user pastes one
into the conversation anyway, tell them to revoke it in the same Settings card
and create a new one.

## Tools

| Tool | Scope | Does |
|---|---|---|
| `list_my_sites` | Baca | The user's websites (not template drafts): id, name, subdomain, status, public url, page/post counts, the site's limits, and whether a new site fits the account |
| `get_plan_limits` | Baca | Free/Pro limits, the account's room for a new site; with `siteId`, that site's plan and pages used/left |
| `check_subdomain` | Baca | Is `<name>.wpage.id` free? normalised name, why not, up to 3 free alternatives |
| `export_site_bundle` | Baca | The site as a folder: `site.json` with ids, `files` (the user's own WVF sections), `notCarried`, and `lock` |
| `create_site` | Tulis | A new DRAFT website from `manifest` (+ `variants`, `subdomain`, `assets`) |
| `update_site` | Tulis | Change a site by diff (needs `baseRevision`) |
| `add_page` | Tulis | Add one page (needs `baseRevision`) |
| `begin_asset_upload` | Tulis | An upload session for the user's own images |
| `publish_site` / `unpublish_site` | Publikasikan | Take a site live (optionally under a new subdomain) / back to draft |

There is no tool to delete a site, change the plan, set a custom domain,
touch payments, integrations (analytics, pixels), products or blog posts —
those stay in the dashboard. Say so instead of looking for a way.

## Every write: dry run, then confirm

All write tools DRY-RUN unless you pass `confirm: true`. The dry run validates
everything and writes nothing. Read the whole report:

- `ok` — clean or not;
- `manifestErrors` — each addressed by path (`pages[1].sections[3] (FAQ / default): …`);
- `warnings` — worth reading, not blocking;
- `summary` (updates) — what would change, one line each;
- `notes` (updates) — what a deletion or a slug change leaves behind, and every license slot the write would use;
- `licenses` (create) — the same for license slots.

Fix, dry-run again until `ok: true`, show the user what will happen, then call
once more with `confirm: true`. Limits: 30 dry runs and 10 writes per 10
minutes per user — `variant-check site` first, so you arrive with a clean file.

## Images

```bash
# 1. begin_asset_upload → { uploadSessionId, url }
curl -X POST "$URL" -H "Authorization: Bearer $WEBTO_TOKEN" \
  -F "uploadSessionId=$SESSION" -F "file=@assets/kedai.jpg"
# 2. repeat per file, then pass to create_site / update_site:
#    "assets": { "uploadSessionId": "…" },
#    "assetManifest": [{ "filename": "kedai.jpg", "size": 812345, "type": "image/jpeg" }]
```

Never put image bytes (base64) into a tool call.

## webto.lock.json

A real write returns `lock`; write it verbatim beside `site.json`:

```json
{ "lockVersion": 1, "kind": "site", "siteId": "…", "subdomain": "kopi-senja", "revision": "s1-…", "variants": {} }
```

`revision` is the `baseRevision` your next `update_site` / `add_page` must
send. It changes whenever the site changes — in the dashboard too — so a
stale one is refused (`revisionConflict: true`): export again, re-apply your
change to the fresh `site.json`, and retry. Never "force" around it; the
refusal is what stops you from overwriting the user's own edits.

## Errors you will meet

| Message | Meaning / what to do |
|---|---|
| `FORBIDDEN` … batas N situs gratis | The account has no free site slot. The user can archive an unused site or upgrade in the dashboard; nothing you can do. |
| `… melebihi batas 5 per situs` / `15 per halaman` | Plan limit on pages / sections. Merge thin pages, trim sections, or ask the user to upgrade. |
| `subdomain: Subdomain sudah digunakan` | Offer the listed alternatives; let the user pick. |
| `variant "u:…" bukan milik Anda dan belum dibeli` | Use a platform variant from the catalog instead. |
| `… tidak dikenal di situs ini — ekspor ulang` | An `id` that is not this site's — you edited a stale or foreign file. Export again. |
| `… akan DIHAPUS … kirim allowPageRemoval: true` | Your file drops a page. Put it back, or ask the user and then send the flag. |
| `NOT_FOUND Situs tidak ditemukan` | Wrong id, not this user's site, or a template draft (the other tool family). |
| `This token lacks the sites:… scope` | The token was created without that box ticked; the user makes a new one. |
