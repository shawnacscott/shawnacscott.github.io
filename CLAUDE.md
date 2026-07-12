# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The marketing/content website for Hearthfire Clay, a nonprofit community ceramics studio in Gilroy, CA. It's a static Jekyll site — no application backend, no JS framework, no test suite. Class registration, payments, and scheduling are handled by an external service (Kilnfire), embedded via script tags.

## Commands

Build/serve with Bundler + Jekyll (Ruby):

```bash
bundle install          # install gems (jekyll ~> 4.3, jekyll-sitemap, jekyll-environment-variables)
bundle exec jekyll serve    # local dev server with live rebuild, http://localhost:4000
bundle exec jekyll build    # production build into _site/
```

There is no lint, test, or CI command configured in this repo.

## Architecture

**Content model.** Pages are mostly flat Markdown files at the repo root (`index.md`, `about.md`, `classes.md`, `contact.md`, `donate.md`, `team.md`, `volunteer.md`, `policies.md`), each with front matter selecting a layout. Two Jekyll collections hold repeatable content:
- `_classes/` — one file per class/workshop (front matter: `title`, `weight`, `price`, `min_age`, `registration_url`, `intro_image`, `active`). Subfolder `_classes/workshops/` is just organizational; both plain and nested files are in the same `classes` collection (see `collections.classes` in `_config.yml`).
- `_team/` — one file per team member, `team` collection.

Both collections get `layout` auto-assigned via `defaults` in `_config.yml`, so individual class/team files don't need `layout:` in front matter.

**Class listing/sorting convention.** Anywhere classes are listed (home page, `_layouts/classes.html`, mobile menu), the same Liquid pattern is repeated: split into classes with a `weight` (sorted by it) and classes without one (natural order), then concat — weighted classes always sort first. A class is hidden from listings when `active: false` is set in its front matter (checked in `_includes/class-card.html` and the menu injection in `_includes/main-menu.html`); the class's own page still builds and is reachable by direct URL even when inactive.

**Layouts** (`_layouts/`) are thin wrappers that all extend `default.html`, which owns the `<head>`, header, footer, and global includes. Layout front matter can set a `bodyClass` and page-specific defaults (e.g. `_layouts/class.html` sets `default_intro_image` as a fallback when a class has none).

**Includes** (`_includes/`) hold reusable partials: `header.html`, `footer.html`, `main-menu.html` / `main-menu-mobile.html` (desktop/mobile nav, driven by `_data/menus.yml`), `class-card.html` (class summary card used on the home page and classes listing), `social.html`, `seo-schema.html`, plus third-party embed snippets (`google-analytics.html`, `google-tag-manager-*.html`, `kilnfire-conversion-tracking.html`, `givebutter.html`).

**Data files** (`_data/`) drive site-wide config outside of page content: `menus.yml` (nav structure, including the `inject_classes` flag that injects the live class list into a menu item), `contact.yml`, `seo.yml`, `social.json`, `policies.json` (Google Docs embeds for legal pages), `features.json`.

**Styles.** SCSS lives in `_sass/`, compiled from `assets/css/style.scss` (Jekyll's Sass processor, `compressed` output per `_config.yml`). Structure: `_sass/bootstrap/` is a largely unmodified Bootstrap 5.3 vendor copy (most component partials are commented out in `style.scss` — only what's needed is imported); `_sass/components/` holds this site's actual component styles (one partial per component, e.g. `_header.scss`, `_classes.scss`, `_intro.scss`); `_sass/pages/` holds page-specific overrides keyed to each layout's `bodyClass`; `_sass/libraries/hamburgers/` is a vendored mobile-menu-icon library. `_bootstrap-variables.scss` at the top level overrides Bootstrap's default variables (brand colors, fonts, footer colors) before Bootstrap itself loads — this is the place to change the site's color palette or typography.

**Third-party integrations**, all wired in via includes in `_layouts/default.html`: Google Tag Manager/Analytics, Givebutter (donation widget, embedded on the home page as `<givebutter-widget>`), and Kilnfire (class registration/e-commerce, embedded per-class via `_layouts/class.html`'s `<div class="kilnfire-upcoming-classes" data-org-id="..." data-template-id="...">` and `classembed.js`, plus conversion tracking).

**`jekyll-serif-theme/`** is a near-empty leftover directory (only IDE metadata) — not an active theme source, safe to ignore.

## Content editing notes

- Class/workshop body content is authored as raw HTML wrapped in a single `<div class="... prose ...">` block inside the Markdown file, not as Markdown prose — match that pattern when editing class descriptions.
- `min_age` and `price` in class front matter drive both the price/age line rendered on the class page (`_layouts/class.html`) and the summary shown in `class-card.html`; keep both in sync when changing a class's pricing or age range.
- `llms.txt` at the repo root is a hand-maintained summary of core site pages for LLM crawlers — update it when adding/removing/renaming top-level pages.
