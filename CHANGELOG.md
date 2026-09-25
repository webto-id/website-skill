# Changelog

## 0.1.1 — 2026-09-25

- The interview now covers every row of the business type's table before planning. A test run on a company site never asked about its directors, although the question was in the table; a row the user declines is fine, a row never asked is not.

## 0.1.0 — 2026-09-25

- First release. Interview-first website building for webto.id: core and per-business-type questions mapped to sections and fields, a no-invented-facts rule, confirmation before any write, publish only on the user's approval.
- `site.json` format with stable page/section ids; editing is export → change → `update_site` with `baseRevision` (id-preserving diff; blog posts are never touched).
- MCP tools (scopes `sites:read` / `sites:write` / `sites:publish`): `list_my_sites`, `get_plan_limits`, `check_subdomain`, `export_site_bundle`, `create_site`, `update_site`, `add_page`, `publish_site`, `unpublish_site`, `begin_asset_upload`.
- Local check: `variant-check site <folder>` (≥ 0.1.37).
