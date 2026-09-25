# site.json

One file describes the whole website. It is the same site definition the
platform's template importer reads, validated by the same schema, with two
differences: no `listing` (a website is not merchandise), and — after an
export — an `id` on every page and section.

```jsonc
{
  "siteName": "Warung Senja",
  "language": "id",                 // 30 languages; drives fixed UI labels ("Alamat", "Baca selengkapnya")
  "siteType": "website",            // "website" | "landing" | "personal" — exactly these
  "aiDescription": "Warung makan…",   // 1-3 sentences; feeds the site's AI context (llms.txt, blog)
  "businessProfile": { … },         // strongly recommended, see below
  "theme": { … },                   // optional; omitted keys take platform defaults
  "siteSections": [ … ],            // chrome: exactly one navbar (header), optional banner, footer
  "pages": [ … ],                   // one page must have slug "" (the homepage)
  "variants": [ … ]                 // one entry per sections/<key>.astro — every section you design
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
  { "type": "navbar", "position": "header", "variant": "u:@navbar-warung", "content": { "siteName": "Warung Senja", "ctaText": "Pesan", "ctaUrl": "https://wa.me/6281234567890" } },
  { "type": "footer", "position": "footer", "variant": "u:@footer-warung", "content": { "text": "© 2026 Warung Senja" } }
],
"pages": [
  {
    "title": "Beranda",
    "slug": "",                     // "" = homepage; others are slugified ("Menu Kami" → "menu-kami")
    "seoTitle": "Warung Senja — masakan rumahan di Jl. Kaliurang, Yogyakarta",
    "seoDescription": "…",
    "showInNavbar": true,           // default true
    "showInFooter": false,          // default false
    "sections": [
      { "type": "hero", "variant": "u:@hero-warung", "content": { "headline": "…", "ctaText": "Lihat menu", "ctaUrl": "/menu", "images": [{ "url": "asset:warung.jpg", "alt": "Bagian depan Warung Senja" }] } },
      { "type": "cta", "variant": "u:@cta-warung", "content": { "heading": "…", "buttonText": "Chat WhatsApp", "buttonUrl": "https://wa.me/6281234567890" } }
    ]
  }
]
```

- `type`: from `catalog.md`. `variant`: `u:@<key>` for your own section (the rule), or a platform variant name from `catalog.md` for the exceptions (`map`, `post`); omitting it gives the type's first platform variant. `content` must match the type's base fields (plus the extension fields your variant declares).
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

## Your own sections: `variants[]`

Every section you design is one file, `sections/<key>.astro`, declared once:

```jsonc
"variants": [
  { "key": "cta-warung", "sectionType": "cta", "name": "Cta Warung",
    "description": "Rounded primary band, centred heading, one light pill button.",   // English, what it looks like
    "mood": ["warm", "bold"] }
]
```

- Reference it from any section of that `type` as `"variant": "u:@<key>"`. **One key, many sections**: the cta on Beranda and on Menu, the hero of every inner page — same file, different `content`. The site stays coherent and within the limits (12 new files per upload, 24 own variants per site).
- The key is lowercase with hyphens and equals the file's basename. `name` is the key in English Title Case.
- On a website these variants are private: never listed, no marketplace approval. They are checked automatically in the background after each save; a script that is clearly harmful is switched off (the section still renders without it) and the user is emailed — so keep scripts to the section's own interface (variant skill `wvf.md` §4 "Review rule"). After an export, `variants[]` lists the site's own variants and `sections/` holds their files.
- On `update_site`, send in `variants` ONLY the entries whose file you changed or added, each with its `source`. A `u:@<key>` whose entry you leave out means "the variant this site already has under that key"; a key the site does not have is an error. A changed file under an existing key becomes a new VERSION of that variant, not a new variant.

## Check it

```bash
npx @webto-id/variant-check site ./my-site                          # the whole folder
npx @webto-id/variant-check sections/cta-warung.astro --type cta   --content cta-content.json --out preview-cta.html                 # one file, rendered with real content
```

One line per problem — manifest, every `.astro`, and every section's content,
e.g. `pages[0].sections[4] (FAQ / default): items: minimal 1 item, ada 0`.
The `create_site` / `update_site` dry run then re-checks everything
server-side, including what only the server knows (which variants you own or
license, your plan's limits, the subdomain).
