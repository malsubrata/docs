# CLAUDE.md

Guidance for Claude Code (and other AI assistants) working in this repository.

## What this repo is

This is the **Mintlify documentation site** for **TeraWallet** (plugin slug `woo-wallet`,
formerly WooWallet), a digital-wallet plugin for WooCommerce by StandaloneTech.

This repo contains **two distinct things**:

1. **The documentation site** — the `.mdx` pages, `docs.json`, `logo/`, `images/`, etc.
   This is what gets published.
2. **`woo-wallet-src/`** — a checkout of the **free core** plugin source. The **source of truth**
   for the free docs. **Read-only**; never edit plugin code here. Has its own `CLAUDE.md`.
3. **`woo-wallet-pro-src/`** — a checkout of the **TeraWallet Pro** add-on source. The source of
   truth for the `pro/` docs. Also **read-only**, also has its own `CLAUDE.md` (very detailed —
   read it before writing Pro docs).

Documented versions: core **1.6.4** (`woo-wallet-src/woo-wallet.php`), Pro **1.0.6**
(`woo-wallet-pro-src/woo-wallet-pro.php`). Pro requires the core plugin to run.

## Local development

This is a [Mintlify](https://mintlify.com) project configured by `docs.json`.

```bash
npm i -g mint      # install the Mintlify CLI (one-time)
mint dev           # preview locally at http://localhost:3000
mint broken-links  # check for broken internal links before publishing
```

Publishing is handled by Mintlify's GitHub integration: pushing to the default branch deploys.

## Repository layout

```
docs.json                  ← site config: theme, colors, nav tabs/groups, SEO metatags
index.mdx                  ← homepage
getting-started/           ← key-features, installation, configuration, pro
user-guide/                ← wallet-dashboard, partial-payments
admin-guide/               ← settings, transactions
cashback/                  ← rules-logic
developer-guide/           ← architecture, hooks, rest-api
faq/                       ← troubleshooting
releases/                  ← changelog (Release Notes)
pro/                       ← TeraWallet Pro tab (overview, install, features, settings, hooks)
logo/  images/             ← brand assets and screenshots
GEMINI.md                  ← legacy AI-context file; overlaps with this one (keep consistent)
woo-wallet-src/            ← READ-ONLY free plugin source (source of truth for free docs)
woo-wallet-pro-src/        ← READ-ONLY Pro plugin source (source of truth for pro/ docs)
```

Navigation is defined in `docs.json` under `navigation.tabs`. **Every new page must be both
created as an `.mdx` file and registered in `docs.json`**, or it won't appear in the sidebar.

## Authoring conventions

- **Frontmatter**: every page needs `title` and a unique, descriptive `description` (the
  `description` is the SEO meta description — make it specific, not boilerplate).
- **Mintlify components** in use: `<Card>` / `<CardGroup>`, `<Note>`, `<Tip>`, `<Info>`,
  `<Warning>`, and `<Update>` (used for release entries in `releases/changelog.mdx`).
- **Currency examples**: use a neutral `$` in examples for consistent rendering. Do not paste
  locale-specific symbols (the original docs used `₹`, which has been standardized to `$`).
- **Internal links** are root-relative without the extension, e.g. `/developer-guide/rest-api`.
- Flag new features with their version, e.g. *(new in 1.6.4)*, and link to the release notes.

## The golden rules (accuracy)

The docs must track the plugin source. Before documenting anything, verify it against
`woo-wallet-src/`:

- **REST API**: the canonical core namespace is **`terawallet/v1`** (NOT `wc/v3/wallet`, which is a
  deprecated back-compat layer). Verify routes/args in
  `woo-wallet-src/includes/api/v1/**/class-*.php`. Pro adds `terawallet/v1/coupons`, withdrawal
  webhooks under `terawallet/v1`, and the importer under `woo-wallet-pro/v1/import` — verify these
  in `woo-wallet-pro-src/modules/**`.
- **Pro docs** (`pro/`): the source of truth is `woo-wallet-pro-src/` (read its `CLAUDE.md` and
  `README.md` first). Pro is **1.0.6** and requires the core plugin.
- **Settings labels**: verify in `woo-wallet-src/includes/class-woo-wallet-settings.php`.
- **Hooks/filters**: grep the actual name in `woo-wallet-src/` before documenting it — names in
  marketing copy or memory are not authoritative. Many filters live in
  `includes/helper/woo-wallet-util.php`, `includes/class-woo-wallet-wallet.php`, and `templates/`.
- **Version & requirements**: from the header in `woo-wallet-src/woo-wallet.php`.
- **Verbatim changelog**: `woo-wallet-src/changelog.txt` (mirrored in `readme.txt`).

If marketing/changelog text and the code disagree, the **code wins** — document what the code does.

## Release-update checklist (when the plugin version bumps)

1. Read the new version's entry in `woo-wallet-src/changelog.txt`.
2. Add an `<Update>` block at the top of `releases/changelog.mdx` (label = version, description = date).
3. Update affected feature pages; verify every new setting/endpoint/hook name against the source.
4. Update the version reference in this file and on the homepage Quick Links if applicable.
5. Run `mint broken-links` and `mint dev`; confirm new pages render and appear in nav.
6. SEO pass: each touched page has a unique `description`; images have `alt` text.

## SEO notes

`docs.json` carries the site `description` and `seo.metatags` (OpenGraph + Twitter card pointing
at `/images/hero.png`). To enable analytics, add an `integrations` block to `docs.json` with your
provider's key, e.g.:

```json
"integrations": { "ga4": { "measurementId": "G-XXXXXXXXXX" } }
```

(Left out intentionally — no live key is committed. Add yours when ready.)
