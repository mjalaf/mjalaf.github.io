# Project Specification — `mjalaf.github.io`

> Personal blog & portfolio of Martin Jalaf. Static site built with Astro 6, Tailwind CSS v4, and MDX, deployed to GitHub Pages.

---

## 1. Purpose & Scope

`mjalaf.github.io` is a fully static personal website that publishes technical
long-form content (blog posts, whitepapers), reusable command/code references
(snippets), and portfolio entries (projects). It has **no backend, no database,
and no authentication** — all pages are generated at build time and served as
static HTML/CSS/JS from GitHub Pages.

The site is bilingual (English + Spanish). Language is expressed through the
`english` / `spanish` frontmatter tag, **not** through URL prefixes or separate
locale routing.

### Goals
- Publish and organize technical content with fast, static delivery.
- Provide discoverability via tags, years, and categories.
- Keep authoring low-friction: author in Markdown/MDX with typed frontmatter.
- Zero server maintenance; deploy on push to `master`.

### Non-Goals
- User accounts, comments storage (comments are delegated to Disqus).
- Server-side rendering or dynamic API endpoints.
- Localized URL routing.

---

## 2. Tech Stack

| Concern | Choice |
| --- | --- |
| SSG framework | Astro `^6.3.1` (`astro`, `@astrojs/mdx`, `@astrojs/rss`, `@astrojs/sitemap`) |
| Styling | Tailwind CSS v4 via `@tailwindcss/vite` + `@tailwindcss/typography` |
| Images | `astro:assets` + `sharp` (build-time optimization) |
| Diagrams | Mermaid 11, loaded client-side from CDN on post/project pages |
| Comments | Disqus (blog post + whitepaper pages) |
| Analytics | Google Tag Manager (`GTM-M4LH8GS`), injected via `BaseHead.astro` |
| Hosting / CI | GitHub Pages via `.github/workflows/deploy.yml` on push to `master` |
| Node | `22.x` (CI) |
| Output mode | `static` (default; no `output` set in `astro.config.mjs`) |

Site URL: `https://mjalaf.github.io` (configured in `astro.config.mjs`).

---

## 3. Content Model

Content lives under `src/content/` and is validated by Zod schemas in
`src/content.config.ts`. Four collections are defined. All loaders use
`glob({ base, pattern: '**/*.{md,mdx}' })`, so the route slug is derived from
the filename.

### 3.1 `blog`
| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `title` | string | ✅ | |
| `description` | string | ✅ | |
| `pubDate` | Date | ✅ | `z.coerce.date()` — accepts ISO `YYYY-MM-DD` |
| `updatedDate` | Date | ❌ | |
| `heroImage` | image | ❌ | Resolved via `image()` helper |
| `tags` | string[] | ❌ | Drives tag clouds and tag pages |

### 3.2 `projects`
| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `title` | string | ✅ | |
| `description` | string | ✅ | |
| `github` | string (URL) | ❌ | Must be a valid URL |

### 3.3 `snippets`
| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `title` | string | ✅ | |
| `description` | string | ✅ | |
| `category` | string | ✅ | Groups snippets on the index; drives category icon |

### 3.4 `whitepapers`
| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `title` | string | ✅ | |
| `description` | string | ✅ | |
| `pubDate` | Date | ✅ | `z.coerce.date()` |
| `updatedDate` | Date | ❌ | |
| `heroImage` | image | ❌ | |
| `tags` | string[] | ❌ | |
| `author` | string | ❌ | Rendered in the byline |
| `version` | string | ❌ | Rendered as a `v{version}` chip |
| `pdfUrl` | string (URL) | ❌ | Renders a "Download PDF" button |

**Current content counts** (informational): blog `31`, projects `2`,
snippets `18`, whitepapers `5`.

---

## 4. Routing

Astro file-based routing under `src/pages/`. All routes render static HTML.
Dynamic routes use `getStaticPaths()` to enumerate values at build time; the
route param for detail pages is the content-collection `id` (filename-derived).

| Route | File | Purpose |
| --- | --- | --- |
| `/` | `index.astro` | Home: hero, tag cloud (by count), year cloud, recent posts |
| `/about` | `about.astro` | Bilingual bio + hardcoded FAQ |
| `/blog` | `blog/index.astro` | All blog posts, newest first, card grid |
| `/blog/<slug>/` | `blog/[...slug].astro` | Single post via `BlogPost.astro` layout |
| `/blog/tag/<tag>/` | `blog/tag/[tag].astro` | Posts filtered by tag |
| `/blog/year/<year>/` | `blog/year/[year].astro` | Posts filtered by calendar year |
| `/projects` | `projects/index.astro` | Project cards |
| `/projects/<slug>/` | `projects/[...slug].astro` | Project detail (Mermaid enabled) |
| `/snippets` | `snippets/index.astro` | Snippets grouped by `category` |
| `/snippets/<slug>/` | `snippets/[...slug].astro` | Snippet detail (copy-to-clipboard) |
| `/whitepapers` | `whitepapers/index.astro` | Whitepaper cards, newest first |
| `/whitepapers/<slug>/` | `whitepapers/[...slug].astro` | Whitepaper detail via `BlogPost.astro` |
| `/rss.xml` | `rss.xml.js` | RSS feed of the `blog` collection |
| `/sitemap-index.xml` | (generated) | Produced by `@astrojs/sitemap` |

---

## 5. Components & Layouts

Shared UI lives in `src/components/`; the single shared layout is
`src/layouts/BlogPost.astro`.

- **`BaseHead.astro`** — `<head>` for every page: charset/viewport, favicons,
  sitemap + RSS `<link>`, font preloads, canonical URL, primary meta,
  Open Graph + Twitter cards (falls back to a placeholder image), and the
  inline **Google Tag Manager** script. Imports `src/styles/global.css`.
- **`Header.astro`** — Sticky nav (`Home`, `Blog`, `Snippets`, `Projects`,
  `About`) with social icons and a `<details>`-based mobile menu. Uses
  `SITE_TITLE`. _(Note: `Whitepapers` is a route but verify it is present in the
  nav if desired.)_
- **`Footer.astro`** — Copyright + Astro credit; year from `new Date()`.
- **`FormattedDate.astro`** — Renders a `Date` as `en-us` `MMM D, YYYY` inside
  `<time datetime="...">`.
- **`HeaderLink.astro`** — Generic active-link helper.
- **`BlogPost.astro`** (layout) — Used by blog **and** whitepaper detail pages.
  Renders header (dates/title/description/tag chips), optional optimized hero
  image (`1020×510`), MDX body in `prose prose-invert`, a **Disqus** thread, and
  the **Mermaid** runtime (dynamically imported from CDN on `DOMContentLoaded`
  when `<pre data-language="mermaid">` blocks exist; theme `dark`).

`projects/[...slug].astro` and `snippets/[...slug].astro` inline their own page
chrome rather than using the shared layout. The snippet route injects per-page
CSS and a script that adds a "Copy" button to every `<pre>`.

---

## 6. Cross-Cutting Concerns

- **SEO**: canonical URLs, OG/Twitter cards, sitemap, and RSS are wired through
  `BaseHead.astro` and integrations.
- **Analytics**: GTM `GTM-M4LH8GS` (script in `BaseHead`, `<noscript>` iframe on
  pages that render `<Header />`).
- **Styling**: Tailwind v4 utility classes, dark-mode-first palette
  (`bg-zinc-900/60`, `text-zinc-300`, etc.); `@tailwindcss/typography` for prose.
  Global styles + Atkinson Hyperlegible font in `src/styles/global.css`.
- **Diagrams**: Mermaid fences (` ```mermaid `) render client-side on blog,
  whitepaper, and project detail pages only.
- **Comments**: Disqus embedded via the `BlogPost` layout.

---

## 7. Build, Run & Deploy

| Action | Command |
| --- | --- |
| Install | `npm install` (CI uses `npm ci`) |
| Dev server | `npm run dev` (port `4321`) |
| Production build | `npm run build` → `dist/` |
| Preview build | `npm run preview` |
| Astro passthrough | `npm run astro -- <cmd>` (e.g. `astro check`) |

**Deployment** — Push to `master` triggers `.github/workflows/deploy.yml`:
1. `actions/checkout@v4`
2. `actions/setup-node@v4` (Node 22)
3. `npm ci`
4. `npm run build`
5. `actions/upload-pages-artifact@v3` from `./dist`
6. `actions/deploy-pages@v4` publishes to the `github-pages` environment

No preview/staging environment exists.

---

## 8. Authoring Conventions

- **Frontmatter**: copy the shape from a recent neighbor in the same collection.
  `pubDate` is required for `blog` and `whitepapers` (ISO `YYYY-MM-DD`).
- **Tags**: reuse existing tags to avoid fragmenting clouds. Language is a tag
  (`english` / `spanish`). Examples: `architecture`, `microservices`,
  `cloud-native`, `azure`, `ai`.
- **Hero images**: place under `public/images/<PostName>/` and reference by
  absolute path, or under `src/assets/<PostName>/` and import for hashing +
  optimization. Layout sizes heroes to `1020×510`.
- **Snippet categories**: recognized icons on `/snippets` include
  `Linux 🐧`, `Docker 🐳`, `AKS ☸️`, `ARO 🎩`, `Azure APIM 🔀`, `KQL 🔍`,
  `Terraform 🏗️`, `Networking 🌐`; others fall back to `📄`. Add new categories
  to the `categoryIcons` map in `pages/snippets/index.astro`.
- **File naming**: match neighboring files in the same collection (kebab-case or
  PascalCase-with-hyphens). The filename becomes the URL slug.
- **Windows artifacts**: `*:Zone.Identifier` files should not be committed.

---

## 9. Constraints & Assumptions

- Static-only; no runtime server logic or secrets.
- Third-party runtime dependencies (Mermaid, Disqus, GTM) load from CDNs, so
  their failures won't surface during `astro build` — verify via local preview.
- Adding a new tag/year requires no registration; listing routes pick it up on
  the next build.
- `tsconfig.json` extends `astro/tsconfigs/strict` with `strictNullChecks`.

---

## 10. Directory Reference

```
mjalaf.github.io/
├── .github/workflows/deploy.yml   # GitHub Pages build & deploy
├── astro.config.mjs               # site URL, integrations, Vite/Tailwind plugin
├── tsconfig.json                  # extends astro strict config
├── public/                        # static assets served as-is (fonts, images)
├── src/
│   ├── consts.ts                  # SITE_TITLE, SITE_DESCRIPTION
│   ├── content.config.ts          # Zod schemas: blog, projects, snippets, whitepapers
│   ├── styles/global.css          # Tailwind import + font-face + body styles
│   ├── assets/                    # build-processed assets
│   ├── components/                # BaseHead, Header, Footer, FormattedDate, HeaderLink
│   ├── layouts/BlogPost.astro     # shared layout (blog + whitepapers)
│   ├── content/{blog,projects,snippets,whitepapers}/
│   └── pages/                     # file-based routing
└── README.md
```
