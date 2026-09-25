# Changelog

## 0.2.1 — 2026-09-26

- **A site's own scripts are checked in the background.** They run on the user's site right away; a script that is clearly harmful is switched off (the section still renders without it) and the user is emailed. Keep scripts to the section's own interface (the variant skill's `wvf.md` §4 "Review rule"). Replaces "runs without review".

## 0.2.0 — 2026-09-25

- **Every section is designed, not assembled.** The agent authors a WVF variant for every section, the navbar and footer included, following the html-to-webto-variant skill — which must now be installed alongside. Platform variants remain only where WVF cannot do the job: an embedded interactive map (`map`) and a blog post body (`post`). Forms, video, database-driven product and blog lists, sliders and countdowns are all authorable.
- **A design step before any file.** From the interview (feel, brand colours, logo, a site the user likes) the agent fixes one visual direction — the theme first, then the recurring moves (headings, buttons, cards, section rhythm, one or two motifs) — and shows it in the confirmation summary.
- **Limits taught and checked.** A site holds at most 24 of its own variants and one upload carries at most 12; `get_plan_limits` reports both (`variants.maxPerSite`, `variants.maxPerUpload`, and per site `variantsUsed` / `variantsLeft`). Reuse one key across pages (inner-page hero, cta, footer) instead of a new file per page.
- **Validation per site and per file:** `variant-check site` for the folder, then `variant-check sections/<key>.astro --type … --content … --out preview.html` per file, looked at 390/768/1280 px in light and dark. Run from the site folder, the preview wears the site's own theme (`colors` / `darkColors`, fonts, radius), shows `asset:` images from `assets/` and stock stand-ins for `__IMG__:` — a demo palette hid exactly the contrast problems a real theme shows. Needs `@webto-id/variant-check` ≥ 0.1.37.
- **Editing sends only what changed.** An export carries the site's own `sections/*.astro`; `update_site` takes in `variants` only the files that changed or are new — an unsent `u:@<key>` means the variant the site already has. A changed file under an existing key is a new version, not a new variant.
- New example `examples/warung-senja/` (five authored sections, one key reused across two pages, the platform map on the contact page), clean under `variant-check site`. It replaces `kopi-senja`.

## 0.1.1 — 2026-09-25

- The interview now covers every row of the business type's table before planning. A test run on a company site never asked about its directors, although the question was in the table; a row the user declines is fine, a row never asked is not.

## 0.1.0 — 2026-09-25

- First release. Interview-first website building for webto.id: core and per-business-type questions mapped to sections and fields, a no-invented-facts rule, confirmation before any write, publish only on the user's approval.
- `site.json` format with stable page/section ids; editing is export → change → `update_site` with `baseRevision` (id-preserving diff; blog posts are never touched).
- MCP tools (scopes `sites:read` / `sites:write` / `sites:publish`): `list_my_sites`, `get_plan_limits`, `check_subdomain`, `export_site_bundle`, `create_site`, `update_site`, `add_page`, `publish_site`, `unpublish_site`, `begin_asset_upload`.
- Local check: `variant-check site <folder>` (≥ 0.1.37).
