# Nexus

A HubSpot CMS theme built on top of the [HubSpot CMS Boilerplate](https://developers.hubspot.com/docs/cms/building-blocks/themes/hubspot-cms-boilerplate), extended with custom modules, templates, and global style controls for a marketing site (home, about, pricing, blog, contact, and landing pages).

## Requirements

- [HubSpot CLI](https://developers.hubspot.com/docs/cms/developer-reference/local-development-cli) (`hs`)
- A connected HubSpot account with CMS access (this project syncs to the **Nexus** folder in Design Manager)

Authenticate once with:

```bash
hs init
```

## Local development

This is a **local-files-first** workflow — changes are made here and pushed to HubSpot's Design Manager, not the other way around.

| Command | Purpose |
|---|---|
| `hs cms upload . Nexus` | One-time push of the current local state to Design Manager |
| `hs cms watch . Nexus` | Watch this folder and auto-upload every change as you save |
| `hs cms fetch Nexus/<path> <local-path> --overwrite` | Pull a specific remote file/folder back down (see note below) |

**Note on `fetch`:** fetching a plain folder appends its own name onto the destination you give (e.g. `hs cms fetch Nexus/sections .` writes to `./sections`). Fetching a `*.module` folder does **not** append — you must give the full destination path yourself (e.g. `hs cms fetch Nexus/modules/hero.module modules/hero.module`), or its contents land flattened into whatever destination you gave.

Preview the theme at:
`https://app.hubspot.com/theme-previewer/<account-id>/edit/Nexus-Local`

## Project structure

```
├── modules/          Custom, reusable CMS modules (see below)
├── templates/         Page templates (home, about, pricing, contact, blog, system pages)
│   ├── layouts/       Shared base layout
│   ├── partials/      Shared header/footer includes
│   └── system/        HubSpot system page templates (404, 500, password prompt, membership, etc.)
├── sections/           Drag-and-drop starter sections (hero, cards, pricing, CTA, multi-column/row content)
├── css/                Theme stylesheet, organized ITCSS-style:
│   ├── generic/        Resets/normalize
│   ├── tools/           Shared mixins/macros
│   ├── elements/        Bare HTML element styling (typography, forms, tables, buttons)
│   ├── objects/         Layout primitives (containers, grid/layout helpers)
│   ├── components/      Reusable UI components (header, default modules)
│   ├── utilities/       Helper/utility classes
│   └── templates/       Page-type-specific styles (system pages, blog)
├── js/                 Theme JavaScript
├── images/             Theme image assets, including module icons and template preview screenshots
├── theme.json           Theme metadata (label, preview path, responsive breakpoints)
├── fields.json           Global theme-level style settings (see below)
└── license.txt
```

## Global theme settings (`fields.json`)

Editable site-wide from the HubSpot theme editor, without touching individual modules:

- **Global colors & fonts** — primary/secondary brand tokens
- **Spacing** — vertical rhythm, max content width
- **Text** — H1–H6 and body/link typography
- **Buttons** — default and hover states (background, border, corner radius, spacing)
- **Forms** — title, labels, fields, submit button, and form container styling
- **Tables** — header/body/footer text & background, cell spacing/border
- **Website header & footer** — menu, dropdowns, background/text colors

## Modules

| Module | Description |
|---|---|
| Hero | Homepage hero with kicker/badge, heading, description, image, and two CTA buttons |
| Page Header | Reusable page-header pattern (breadcrumb + title + description) shared across About, Services, Solutions, Case Studies, Blog, and Resources pages |
| Card | Generic content card |
| Grid Col | Layout column for grid-based sections |
| Button | Standalone CTA button |
| Link | Styled text link |
| Announcement Bar | Site-wide top announcement/banner strip |
| Menu | Navigation menu, including dropdowns |
| Counters | Animated/static stat counters |
| Trust Ticker | Logo/trust-badge strip |
| Case Studies | Case study listing/teaser block |
| Testimonials | Customer testimonial block |
| Pricing Card | Pricing plan card |
| Image With Text | Split image + text content block |
| Social Follow | Social media follow links |
| Footer | Site footer |

Each module follows the standard HubSpot module structure (`meta.json`, `fields.json`, `module.html`, `module.css`, `module.js`, `_locales/`), with CONTENT fields for editable copy/media and STYLE fields (grouped by component, under a `styles` field group) for editor-side design control.

## Templates

- `home.html`, `about.html`, `pricing.html`, `contact.html`, `landing-page.html` — marketing pages
- `blog-index.html`, `blog-post.html` — blog listing and post
- `hubdb.html` — HubDB-driven page
- `qa-test.html` — internal QA/test template
- `system/` — required HubSpot system templates (error pages, search, membership, subscriptions)

## Conventions used in this project

- New style fields should follow the pattern documented in [`.claude/skills/hubspot-style-fields/SKILL.md`](.claude/skills/hubspot-style-fields/SKILL.md): style properties are grouped per visual component under the Style tab, the module's outermost wrapper group is always named `styles`, and borders use HubSpot's native `border` field type (width + style + color together), never a plain color field.
- `.vscode/settings.json` maps `.html`/`.css` files to HubL-aware language modes — install the HubSpot **HubL** VSCode extension so templating syntax inside `<style>` blocks isn't flagged as invalid CSS.

