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
   - **Serah terima** — send a site handoff invitation to another account
     (`create_site_handoff`). Not included in **Tulis**; tick it only when the
     user wants the agent to hand sites over to clients.
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
| `get_plan_limits` | Baca | Free/Pro limits, the account's room for a new site, `variants` (`maxPerSite` 24, `maxPerUpload` 24); with `siteId`, that site's plan, pages used/left and `variantsUsed`/`variantsLeft` |
| `check_subdomain` | Baca | Is `<name>.wpage.id` free? normalised name, why not, up to 3 free alternatives |
| `export_site_bundle` | Baca | The site as a folder: `site.json` with ids and `variants[]`, `files` (`sections/<key>.astro` of the site's own variants, plus samples), `notCarried`, and `lock` |
| `create_site` | Tulis | A new DRAFT website from `manifest` (+ `variants` with each `source`, max 24, `subdomain`, `assets`); without `manifest.llmsTxt` the platform writes the site's llms.txt in the background (`llmsTxt: "generating"`) |
| `update_site` | Tulis | Change a site by diff (needs `baseRevision`); `variants` = only the files you changed or added |
| `add_page` | Tulis | Add one page (needs `baseRevision`) |
| `begin_asset_upload` | Tulis | An upload session for the user's own images |
| `publish_site` / `unpublish_site` | Publikasikan | Take a site live (optionally under a new subdomain) / back to draft |
| `get_preview_url` | Baca | Fresh signed preview links for one site: `previewUrl`, `previewPages`, `expiresAt`, and the public `url` when published |
| `create_site_handoff` | Serah terima | Invite another webto account (by email) to take the site over: dry run lists `blockers` and the summary; `confirm: true` emails the invitation |

Selling a template on the marketplace (template drafts, listings and their
screenshots, with `begin_listing_screenshot_upload`) is a different tool family:
see the `html-to-webto-template` skill, `references/mcp.md`.

There is no tool to delete a site, change the plan, set a custom domain,
touch payments, integrations (analytics, pixels), products or blog posts,
or accept or cancel a site handoff — those stay in the dashboard. Say so
instead of looking for a way.

## Handing a site over

When the user built a site for a client who bought it outright, the site can
move to the client's own account. `create_site_handoff` with `siteId` and the
client's `toEmail`:

- the dry run returns `blockers` (one sentence each: an active Pro
  auto-renewal, unfinished orders, an agency host, a domain still being
  processed, …) and the summary: `carries` (Pro term, AI CS add-on, domains,
  Marketplace licenses, buyer-data counts), `copies` (the user's own variants
  the client gets a copy of) and `cleared` (payment accounts, tracking,
  integrations, shipping credentials and other settings bound to the user);
- show the user that summary, including the buyer-data counts, and call
  again with `confirm: true` only after their explicit yes;
- the client gets an email link valid 7 days and must sign in with THAT
  email and accept in the dashboard. You never see the link and cannot
  accept for them. Until they accept, the site stays the user's.

Blockers are fixed by the user in the dashboard; then dry-run again. One
pending handoff per site: to change the recipient, the user cancels the
pending one in the dashboard first.

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

## Look at what you wrote

Every successful write (`create_site`, `update_site`, `add_page`) and every
`export_site_bundle` returns signed draft links:

- `previewUrl` — the home page, e.g. `https://kopi-senja.wpage.id/?_preview=1&_pt=…`;
- `previewPages` — `[{ title, slug, url }]`, one per page;
- `previewExpiresAt` — when the links stop working (12 hours);
- `url` — the public address, only once the site is published.

Open every page in a browser at 390, 768 and 1280 px wide BEFORE telling the
user the work is done, and fix what you see first. Links clicked inside the
preview stay in preview mode. A dry run of `update_site` / `add_page` wrote
nothing, so it carries `savedPreview` instead: the site as it is saved NOW,
without your change. When the links have expired, `get_preview_url` with the
site id gives fresh ones.

The `_pt` token in these links lets anyone holding them see the draft for 12
hours. Give the links to the user; do not paste them anywhere public.

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

A real write (`create_site`, `update_site`, `add_page`) returns `lock`; write
it verbatim beside `site.json`, replacing the old file. It is the complete lock
— every one of the user's own variants the site uses, `version` filled —
exactly what `export_site_bundle` would return at that moment, however few
variants the write sent:

```json
{ "lockVersion": 1, "kind": "site", "siteId": "…", "subdomain": "kopi-senja", "revision": "s1-…", "variants": {} }
```

`revision` is the `baseRevision` your next `update_site` / `add_page` must
send. It changes whenever the site changes — in the dashboard too — so a
stale one is refused (`revisionConflict: true`): export again, re-apply your
change to the fresh `site.json`, and retry. Never "force" around it; the
refusal is what stops you from overwriting the user's own edits.

`llms.txt` is the one site field outside the revision: the platform may
write it in the background a few seconds after `create_site` (or after an
`update_site` on a site that had none), and that must not stale the revision
you were just given. Every write result says where it comes from in
`llmsTxt`: `"bundle"` (your text), `"generating"` (the platform's, stored
within about a minute), `"cleared"`, `"unchanged"` or `"none"`. To change it,
send `llmsTxt` (see `site-json.md`); leaving the key out never touches it.

Beside the lock, a confirmed write returns `changed`: the variants it moved,
by key — `{ "hero-warung": { "from": 2, "to": 3 } }`. `from: null` means the
key is new to this site (on `create_site` every key is); `to: null` means no
section uses it any more; a key that did not move is absent, and `{}` means no
variant changed.

## Errors you will meet

| Message | Meaning / what to do |
|---|---|
| `FORBIDDEN` … batas N situs gratis | The account has no free site slot. The user can archive an unused site or upgrade in the dashboard; nothing you can do. |
| `… melebihi batas 5 per situs` / `15 per halaman` | Plan limit on pages / sections. Merge thin pages, trim sections, or ask the user to upgrade. |
| `subdomain: Subdomain sudah digunakan` | Offer the listed alternatives; let the user pick. |
| `variant "u:…" bukan milik Anda dan belum dibeli` | A `u:<id>` from someone else's work. Author your own variant (`u:@<key>`) instead. |
| `variant "u:@…" tidak ada di variants[] dan bukan variant milik situs ini` | A key the site does not have: add its entry and `source` to `variants`, or fix the key (after an export, keys are the file names in `sections/`). |
| `… melewati batas 24 per situs` | The site already holds many own variants. Reuse a key across pages instead of a new file; changing a file under an existing key is a new version, not a new variant. |
| `variants` has more than 24 items (a schema error on the call) | One upload carries at most 24 new/changed files, and a site holds at most 24 of its own variants. Reuse keys for designs that repeat. |
| `… tidak dikenal di situs ini — ekspor ulang` | An `id` that is not this site's — you edited a stale or foreign file. Export again. |
| `… akan DIHAPUS … kirim allowPageRemoval: true` | Your file drops a page. Put it back, or ask the user and then send the flag. |
| `NOT_FOUND Situs tidak ditemukan` | Wrong id, not this user's site, or a template draft (the other tool family). |
| `This token lacks the sites:… scope` | The token was created without that box ticked; the user makes a new one. |
| `Sudah ada serah terima tertunda untuk situs ini` | A handoff is already pending; the user cancels it in the dashboard before a new one. |
