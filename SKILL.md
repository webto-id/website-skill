---
name: webto-website
description: Build and manage a small-business WEBSITE on webto.id for the user — interview them about their business, assemble a site.json from platform section variants, create it as a draft over the webto MCP server, publish it on their approval, and later edit it by id-preserving diffs. Use when someone asks to "buat website", "bikinin website untuk usaha saya", "make me a website on webto", "update/tambah halaman di website webto saya", or wants an AI agent to build or change their webto.id site. Not for selling templates on the marketplace (that is html-to-webto-template).
---

# webto-website

You build an ordinary person's website on webto.id — a warung, a clinic, a
contractor, a school, a freelancer — from what they tell you, and you keep it
up to date afterwards. The site is real: customers will read the prices,
phone numbers and opening hours you put on it. That shapes every rule below.

```
my-site/
  site.json          # the site: siteName, theme, businessProfile, siteSections, pages
  webto.lock.json    # written after create/update: siteId + revision (never edit)
  sections/*.astro   # ONLY if the user brought their own design (see step 4)
  assets/*.jpg       # the user's own logo and photos, referenced as "asset:<file>"
```

## Read first

- `references/interview.md` — what to ask, per kind of business, and which section/field each answer fills.
- `references/site-json.md` — the `site.json` format, ids, images, theme.
- `references/mcp.md` — connecting the `webto` MCP server and every tool's contract.
- `references/catalog.md` — every section type and platform variant, with the content fields each accepts (generated — trust it over memory).
- `examples/kopi-senja/site.json` — a complete small-business site that passes `variant-check site`. Its business, prices and numbers are fictional illustrations of the SHAPE; on a real site every such fact comes from the user.

## Non-negotiable rules

1. **Interview before you build.** Ask in small groups (3–6 questions at a time), never a 30-question form. Start with the core questions in `interview.md`, then the ones for their kind of business.
2. **Never invent facts.** Prices, names, addresses, phone and WhatsApp numbers, opening hours, legal details (NIB, izin, akreditasi), certifications, testimonials, client logos and statistics come ONLY from the user. If you don't know it, ask — or leave the field/section out. No "Rp 50.000" placeholders, no "Budi, Jakarta ⭐⭐⭐⭐⭐", no "500+ pelanggan puas". Marketing copy (headlines, descriptions) you may write, from what they told you.
3. **Confirm before writing.** Show the user a summary — pages, the sections on each, the key content, the subdomain — and wait for a yes before `create_site` with `confirm: true`.
4. **Platform variants by default.** Pick variants from `references/catalog.md`. Write WVF `.astro` files ONLY when the user brings their own design or HTML, and then follow the **html-to-webto-variant** skill for every file.
5. **Publishing is the user's call.** Publish only after they have looked at the preview link and approved the subdomain. If your token has no publish scope, tell them to press **Publikasikan** in the dashboard.
6. **Never ask for the token in the chat.** It lives in the `WEBTO_TOKEN` environment variable (setup in `references/mcp.md`).

## Workflow: a new site

1. **Check the connection and the plan.** Call `list_my_sites` and `get_plan_limits`. A Free account holds 3 websites, 5 pages per site (blog posts count), 15 sections per page. If what the user wants does not fit, say so plainly before interviewing — upgrading happens only in their dashboard.
2. **Interview** (`references/interview.md`): the core group first (name, what they do, where, how customers reach them), then their business type's group. Ask for their logo and photos; offer to use stock photos (`__IMG__:`) or illustrations (`__ILLU__:`) where they have none — and say which you used.
3. **Plan the structure** — pages and sections, mapped from the answers — and **show it**. Typical small business: Beranda (hero, features/services, testimonials if they gave any, faq, cta), plus 1–3 pages (Layanan/Menu, Tentang, Kontak). Fewer, fuller pages beat many thin ones.
4. **Write `site.json`** (`references/site-json.md`): real content, a theme that fits the business, `businessProfile` filled from the interview (it powers search results, the WhatsApp bubble and contact facts across the site).
5. **Validate locally**: `npx @webto-id/variant-check site ./my-site` — fix every ✖. It checks every section's content against the same rules as the server, one line per problem.
6. **Subdomain**: `check_subdomain` with the name the user wants; offer the suggestions if it is taken. Let them choose.
7. **Dry run**: `create_site` WITHOUT `confirm`. Read the whole report (`manifestErrors`, `warnings`, `licenses`); fix and repeat until `ok: true`.
8. **Confirm with the user**, then `create_site` with `confirm: true`. Save the returned `lock` as `webto.lock.json`. Give them the `previewUrl` (valid 12 hours; they can also open the site in their dashboard).
9. **Publish on approval**: `publish_site` dry run, then `confirm: true`. Give them the public `url`.

## Workflow: changing an existing site

1. **Export first, always**: `export_site_bundle` with the site id (from `list_my_sites` or `webto.lock.json`). The user may have edited the site in the dashboard since you last saw it. Write the returned `site.json`, `files`, and `lock` to the folder.
2. **Edit `site.json` in place.** Every page and section carries an `id` — KEEP IT on everything that stays. No id = new. A page or section you remove from the file is DELETED on the site.
3. **Adding one page?** `add_page` is lighter: send just that page (and `baseRevision`).
4. **Dry run** `update_site` with `baseRevision` = `lock.revision`. Show the user `summary` and `notes` — especially deletions: a deleted page takes its sections, its page settings, and leaves its form submissions without a page. Deleting a page needs `allowPageRemoval: true`, and you send it only after the user explicitly agreed to that deletion.
5. **Confirm**, then `confirm: true`, and save the new `lock`. If you get `revisionConflict`, the site changed since your export: export again and re-apply your change on top — never force.

Blog posts, products, custom domains, payments and integrations (analytics, pixels) are not part of `site.json` and no tool changes them; send the user to the dashboard for those.

## When the user brings a design

If they have an HTML page, a Figma export or a site they like and want theirs to look like it: convert the parts that platform variants cannot express into WVF files under `sections/`, declared in `site.json`'s `variants[]` and referenced as `u:@<key>` — exactly as the html-to-webto-variant skill describes. On a website these stay private to the user (no marketplace review). Everything else still comes from platform variants.

## Hard limits

| What | Free | Pro |
|---|---|---|
| Websites per account | 3 (archived ones don't count) | unlimited |
| Pages per site (blog posts included) | 5 | unlimited |
| Sections per page | 15 | 1000 |
| Content per section | 50 KB | 50 KB |
| Own images per bundle (`asset:`) | 20 files, 2 MB each, PNG/JPG/WebP | same |
| Custom WVF sections per upload | 12 | 12 |

A limit only blocks what you ADD past it: a site already over a limit (an older site, a lapsed Pro) can still be edited.
