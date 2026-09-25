# site.json

One file describes the whole website. It is the same site definition the
platform's template importer reads, validated by the same schema, with two
differences: no `listing` (a website is not merchandise), and — after an
export — an `id` on every page and section.

```jsonc
{
  "siteName": "Kopi Senja",
  "language": "id",                 // 30 languages; drives fixed UI labels ("Alamat", "Baca selengkapnya")
  "siteType": "website",            // "website" | "landing" | "personal" — exactly these
  "aiDescription": "Kedai kopi…",   // 1-3 sentences; feeds the site's AI context (llms.txt, blog)
  "businessProfile": { … },         // strongly recommended, see below
  "theme": { … },                   // optional; omitted keys take platform defaults
  "siteSections": [ … ],            // chrome: exactly one navbar (header), optional banner, footer
  "pages": [ … ],                   // one page must have slug "" (the homepage)
  "variants": [ … ]                 // ONLY for your own WVF sections (sections/<key>.astro)
}
```

The subdomain is NOT in site.json: pass it to `create_site` as `subdomain`
(after `check_subdomain`), or publish under a new one with `publish_site`.
`variant-check site` tolerates a top-level `subdomain` key, so you may keep it
there as a note.

## businessProfile

Fill it from the interview; every field is optional, none may be invented.

```json
{
  "legalName": "CV Kopi Senja Nusantara",
  "phone": "+6281234567890",
  "email": "halo@kopisenja.id",
  "businessType": "FoodEstablishment",
  "address": { "street": "Jl. Braga No. 12", "city": "Bandung", "region": "Jawa Barat", "postalCode": "40111", "country": "ID" },
  "openingHours": [{ "days": ["Mo", "Tu", "We", "Th", "Fr"], "open": "08:00", "close": "22:00" }, { "days": ["Sa", "Su"], "open": "09:00", "close": "23:00" }],
  "socialLinks": ["https://instagram.com/kopisenja"]
}
```

`businessType` is a closed list: Organization, SoftwareApplication,
LocalBusiness, Restaurant, Store, MedicalBusiness, ProfessionalService,
HealthAndBeautyBusiness, AutomotiveBusiness, HomeAndConstructionBusiness,
FoodEstablishment, LodgingBusiness, SportsActivityLocation,
EducationalOrganization. It drives the site's structured data (Google rich
results). `phone` also switches on the floating WhatsApp button when the site
is created.

## theme

All keys optional. Values are closed sets — anything else is an error.

| key | values |
|---|---|
| `colorMode` | `light`, `dark`, `system` |
| `colors`, `darkColors` | `primary`, `secondary`, `background`, `foreground`, `accent` — hex (`#9A5B00`) or rgb/hsl |
| `fonts` | `heading`, `body`: one of the platform's 93 Google Fonts (Inter, Poppins, Playfair Display, Plus Jakarta Sans, DM Sans, Lora, …; `variant-check site` prints the full list on a wrong name); `headingSize` 24–96, `bodySize` 12–24 |
| `radius` | 0–24 (px) |
| `layout` | `fullscreen`, `boxed` |
| `maxWidth` | `3xl` … `7xl` |
| `spacing` | `compact`, `normal`, `spacious` |
| `textGradient` | `none`, `subtle`, `vibrant` |
| `pageTransitions` | `none`, `fade` |

Pick a palette that fits the business and has contrast: `foreground` must read
on `background`, and text on `primary` must read too. On an update, OMIT
`theme` to keep the site's theme; a theme you send replaces it whole.

## siteSections and pages

```jsonc
"siteSections": [
  { "type": "navbar", "position": "header", "content": { "siteName": "Kopi Senja", "ctaText": "Pesan", "ctaUrl": "https://wa.me/6281234567890" } },
  { "type": "footer", "position": "footer", "content": { "text": "© 2026 Kopi Senja" } }
],
"pages": [
  {
    "title": "Beranda",
    "slug": "",                     // "" = homepage; others are slugified ("Menu Kami" → "menu-kami")
    "seoTitle": "Kopi Senja — kedai kopi di Braga, Bandung",
    "seoDescription": "…",
    "showInNavbar": true,           // default true
    "showInFooter": false,          // default false
    "sections": [
      { "type": "hero", "variant": "split", "content": { "headline": "…", "ctaText": "Lihat menu", "ctaUrl": "/menu", "images": [{ "url": "asset:kedai.jpg", "alt": "Bagian depan Kopi Senja" }] } },
      { "type": "faq", "content": { "heading": "Pertanyaan", "items": [{ "question": "…", "answer": "…" }] } }
    ]
  }
]
```

- `type` and `variant`: from `catalog.md`; omit `variant` for the type's first. `content` must match the type's base fields plus the variant's extra fields.
- The navbar lists the site's pages automatically (those with `showInNavbar`) — do not restate them in `content.links`, which is for EXTRA destinations only.
- Links between pages: `/menu`, `/kontak`; to a section on the same page: `#contact`, `#pricing` (every section gets an anchor from its type automatically).
- Optional per section: `sectionStyles` (background/dividers), `isVisible: false` (kept but hidden). Per page: `themeOverride` — only the fields that page differs on, e.g. `{ "hideNavbar": true }` for a landing page.

## ids (after an export)

`export_site_bundle` returns site.json with `id` on every page, section and
chrome section. They are how `update_site` knows what to change:

| In the file you send | On the site |
|---|---|
| entry with a known `id` | updated in place (only what changed); its form submissions, stats and pixels stay attached |
| entry without `id` | created |
| stored `id` missing from the file | deleted (pages only with `allowPageRemoval: true`) |
| `id` the site does not know | error — never silently "new" |

Never copy ids between sites, never invent them, never renumber them.

## Images

In any image field (`images[].url`, `logoUrl`, `features[].image`, …):

- `asset:<file>` — the user's own photo or logo, uploaded with `begin_asset_upload` (PNG/JPG/WebP, ≤ 2 MB each, ≤ 20 files). Use this for anything that is *theirs*: logo, storefront, team, products.
- `__IMG__:<english query>` — a stock photo the platform picks from Unsplash at creation (`__IMG__:cozy coffee shop interior warm light`). Only for mood pictures, never to pretend to be their product or their people.
- `__ILLU__:<query>` — a platform illustration.
- A full `https://…` URL the user gave you and has the right to use.

Tell the user which images are stock, so they can replace them with their own.

## Your own sections (only with a user's design)

`variants: [{ key, sectionType, name, description, mood }]` plus
`sections/<key>.astro`, referenced from a page as `"variant": "u:@<key>"`.
Follow the html-to-webto-variant skill for the file. On a website these
variants stay private (drafts, no marketplace review); editing the file and
running `update_site` gives the variant a new version.

## Check it

```bash
npx @webto-id/variant-check site ./my-site
```

One line per problem — manifest, every `.astro`, and every section's content,
e.g. `pages[0].sections[4] (FAQ / default): items: minimal 1 item, ada 0`.
The `create_site` / `update_site` dry run then re-checks everything
server-side, including what only the server knows (which variants you own or
license, your plan's limits, the subdomain).
