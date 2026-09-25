---
name: webto-website
description: Build and manage a small-business WEBSITE on webto.id for the user — interview them about their business, design a site of its own (every section authored as a WVF variant, like a template), create it as a draft over the webto MCP server, publish it on their approval, and later edit it by id-preserving diffs. Use when someone asks to "buat website", "bikinin website untuk usaha saya", "make me a website on webto", "update/tambah halaman di website webto saya", or wants an AI agent to build or change their webto.id site. Not for selling templates on the marketplace (that is html-to-webto-template). REQUIRES the html-to-webto-variant skill installed alongside it.
---

# webto-website

You build an ordinary person's website on webto.id — a warung, a clinic, a
contractor, a school, a freelancer — from what they tell you, and you keep it
up to date afterwards. Two things are true at once:

- **The site is real.** Customers will read the prices, phone numbers and
  opening hours you put on it, so no fact is ever invented.
- **The site is designed for them.** You author every section yourself as a
  WVF variant — the same format and the same care as a marketplace template —
  so a warung does not look like a law firm. Assembling platform catalog
  sections is not the job.

```
my-site/
  site.json          # the site: siteName, theme, businessProfile, siteSections, pages, variants[]
  sections/*.astro   # one WVF file per distinct section design (key = file basename)
  webto.lock.json    # written after create/update: siteId + revision (never edit)
  assets/*.jpg       # the user's own logo and photos, referenced as "asset:<file>"
```

## Read first

- The **html-to-webto-variant** skill — every `.astro` here follows it: the WVF subset (`wvf.md`), design rules (`design.md`), Tailwind subset (`tailwind.md`), base fields per section type (`schema.md`). It must be installed alongside this one; if it is not, stop and tell the user to install it.
- `references/interview.md` — what to ask, per kind of business, and which section/field each answer fills.
- `references/site-json.md` — the `site.json` format, ids, images, theme, `variants[]`.
- `references/mcp.md` — connecting the `webto` MCP server and every tool's contract.
- `references/catalog.md` — every section type with its base fields (generated). Use it for the `content` shape of each type; its platform variants are for the few exceptions in rule 4.
- `examples/warung-senja/` — a complete small site with five authored sections (navbar, hero, menu, cta, footer) that passes `variant-check site`. Its business, prices and numbers are fictional; on a real site every such fact comes from the user.

## Non-negotiable rules

1. **Interview before you build.** Ask in small groups (3–6 questions at a time), never a 30-question form. Start with the core questions in `interview.md`, then the ones for their kind of business. Cover EVERY row of that business type's table before you plan: a row the user declines is fine, a row you never asked is not (a company site without the question about its directors, a clinic without its doctors' schedule, reads as unfinished to the owner).
2. **Never invent facts.** Prices, names, addresses, phone and WhatsApp numbers, opening hours, legal details (NIB, izin, akreditasi), certifications, testimonials, client logos and statistics come ONLY from the user. If you don't know it, ask — or leave the field/section out. No "Rp 50.000" placeholders, no "Budi, Jakarta ⭐⭐⭐⭐⭐", no "500+ pelanggan puas". Marketing copy (headlines, descriptions) you may write, from what they told you.
3. **Confirm before writing.** Show the user a summary — pages, the sections on each, the key content, the visual direction, the subdomain — and wait for a yes before `create_site` with `confirm: true`.
4. **Author a WVF variant for every section, chrome included** (navbar, footer, and a banner if there is one), following the html-to-webto-variant skill. A platform variant is used ONLY where WVF cannot do the job:
   - **an embedded, interactive Google map** — `<iframe>` is not allowed in WVF, so use the platform `map` section (it draws the map from `address`); a WVF section may still show the address with a "Buka di Google Maps" link;
   - **a blog post body** (`post`) — written by the platform, never by you.

   Everything else is authorable: forms (`<FormFields />`, `form` type), video (`video()`), products and blog lists (the platform fills `products`/`posts` in database mode and renders pagination around your section), sliders/tabs/countdowns (a `<script is:inline>`; on the user's own site it runs right away and is checked in the background — keep it to the section's own interface, or it gets switched off).
5. **Publishing is the user's call.** Publish only after they have looked at the preview link and approved the subdomain. If your token has no publish scope, tell them to press **Publikasikan** in the dashboard.
6. **Never ask for the token in the chat.** It lives in the `WEBTO_TOKEN` environment variable (setup in `references/mcp.md`).

## Workflow: a new site

1. **Check the connection and the plan.** Call `list_my_sites` and `get_plan_limits`. A Free account holds 3 websites, 5 pages per site (blog posts count), 15 sections per page; every site holds at most **24 of its own variants**, and one upload carries at most **12**. If what the user wants does not fit, say so plainly before interviewing — upgrading happens only in their dashboard.
2. **Interview** (`references/interview.md`): the core group first, then their business type's group, then the look: what the business should feel like (hangat, rapi, mewah, ceria, tegas), brand colors, a logo, photos they own, a site they like. Offer stock photos (`__IMG__:`) or illustrations (`__ILLU__:`) where they have none — and say which you used.
3. **Plan the structure** — pages and sections, mapped from the answers. Typical small business: Beranda (hero, services/menu, testimonials if they gave any, faq, cta), plus 1–3 pages (Layanan/Menu, Tentang, Kontak). Fewer, fuller pages beat many thin ones.
4. **Set the visual direction before writing any file.** From the business and the user's taste, decide ONE design language and hold it across every section:
   - the **theme** first: palette (`colors` with real contrast, `darkColors`), fonts from the platform list (`site-json.md`), `radius`, `spacing` — every section then uses the tokens, never hardcoded colors (variant skill `design.md`, `tailwind.md`);
   - the recurring moves: how headings are set, the button shape, the card and image treatment, the section rhythm (which sections sit on `bg-secondary`, which on `bg-background`), one or two motifs (a tilted photo, a dotted menu leader, a hand-drawn underline) — not a new idea per section;
   - the chrome follows the same language: the navbar and footer are designed, not generic.

   Put the direction in two or three lines in your summary for the user.
5. **Author the sections** — one `sections/<key>.astro` per DISTINCT design, following the variant skill. **Reuse a key across pages** (the inner pages' hero, the cta, the footer are the same design with different `content`): it keeps the site coherent and under the 12-per-upload / 24-per-site limits. Declare each key in `site.json`'s `variants[]` (`key`, `sectionType`, `name` = the key in English Title Case, `description`, `mood`) and reference it as `"variant": "u:@<key>"`. Keys are lowercase with hyphens (`hero-warung`, `menu-warung`).
6. **Write `site.json`** (`references/site-json.md`): real content in every section, the theme from step 4, `businessProfile` filled from the interview.
7. **Validate locally**, and fix every ✖:
   - the whole site: `npx @webto-id/variant-check site ./my-site` — manifest, every `.astro`, and every section's content against the server's rules (non-strict for a site: warnings are advice, errors block);
   - each file with its own content and a preview: `npx @webto-id/variant-check sections/<key>.astro --type <sectionType> --content <content.json> --out preview-<key>.html`, where `content.json` holds that section's `content` from `site.json`. Run it from the site folder: the preview then wears the site's own `theme` (colors, fonts, radius — it says `theme: site.json` when it does), `--theme dark` shows its `darkColors`, `asset:` images come from `assets/`, and `__IMG__:`/`__ILLU__:` show stock stand-ins. Judge contrast only on a preview that says `theme: site.json`;
   - look at every preview. With a browser, check 390, 768 and 1280 px wide, light and dark: no horizontal scroll, headings that wrap well, buttons and text that read on their surface, images that fill their frames. Wait a second before judging: sections with `data-wv-effect` fade in. Without a browser, say so to the user and name this check as not done.
8. **Subdomain**: `check_subdomain` with the name the user wants; offer the suggestions if it is taken. Let them choose.
9. **Dry run**: `create_site` WITHOUT `confirm`. Read the whole report (`manifestErrors`, per-variant `lint`, `warnings`, `licenses`); fix and repeat until `ok: true`.
10. **Confirm with the user**, then `create_site` with `confirm: true`. Save the returned `lock` as `webto.lock.json`. Give them the `previewUrl` (valid 12 hours; they can also open the site in their dashboard).
11. **Publish on approval**: `publish_site` dry run, then `confirm: true`. Give them the public `url`.

More than 12 designs? Create the site with the 12 that matter most (chrome, hero, the main content), then add the rest with `update_site` — up to 24 per site in total.

## Workflow: changing an existing site

1. **Export first, always**: `export_site_bundle` with the site id (from `list_my_sites` or `webto.lock.json`). It returns `site.json` with ids, the site's own `sections/*.astro` (and samples) in `files`, and `lock`. Write them all to the folder. The user may have edited the site in the dashboard since you last saw it.
2. **Edit in place.** Every page and section carries an `id` — KEEP IT on everything that stays. No id = new. A page or section you remove from `site.json` is DELETED on the site.
3. **Section code changes**: edit the `.astro`, and send in `variants` ONLY the entries whose file changed or is new (with `source` = the file's content). A changed file under an existing key becomes a new version of that variant — not a new variant — so it does not count against the 24.
4. **Adding one page?** `add_page` is lighter: send just that page (and `baseRevision`, plus `variants` for any new design it uses).
5. **Validate** as in step 7 above, then **dry run** `update_site` with `baseRevision` = `lock.revision`. Show the user `summary` and `notes` — especially deletions: a deleted page takes its sections, its page settings, and leaves its form submissions without a page. Deleting a page needs `allowPageRemoval: true`, and you send it only after the user explicitly agreed to that deletion.
6. **Confirm**, then `confirm: true`, and save the new `lock`. If you get `revisionConflict`, the site changed since your export: export again and re-apply your change on top — never force.

Blog posts, products, custom domains, payments and integrations (analytics, pixels) are not part of `site.json` and no tool changes them; send the user to the dashboard for those.

## Hard limits

| What | Free | Pro |
|---|---|---|
| Websites per account | 3 (archived ones don't count) | unlimited |
| Pages per site (blog posts included) | 5 | unlimited |
| Sections per page | 15 | 1000 |
| Content per section | 50 KB | 50 KB |
| Own WVF variants per site | 24 | 24 |
| Own WVF variants per upload | 12 | 12 |
| One `.astro` source | 128 KB | 128 KB |
| Own images per bundle (`asset:`) | 20 files, 2 MB each, PNG/JPG/WebP | same |

A limit only blocks what you ADD past it: a site already over a limit (an older site, a lapsed Pro) can still be edited.
