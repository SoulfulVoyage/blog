# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VuePress 1.x theme (Vue 2 + Stylus) for knowledge management and blogging. Dual package structure: the root `package.json` is the demo/blog site; `vdoing/package.json` is the independently publishable theme npm package (`vuepress-theme-vdoing`).

## Commands

```bash
# Dev server
npm run dev          # macOS/Linux (sets NODE_OPTIONS=--openssl-legacy-provider)
npm run dev:win      # Windows equivalent

# Production build
npm run build        # macOS/Linux
npm run build:win    # Windows equivalent

# Deploy to gh-pages
npm run deploy       # Build + force-push dist to gh-pages branch

# Publish theme to npm
npm run publish      # Publishes vdoing/ sub-package

# Batch edit markdown frontmatter
npm run editFm       # Reads config from utils/config.yml
```

Node >= 18 required. The `--openssl-legacy-provider` flag and `--max-old-space-size=4096` are set automatically via npm scripts. No tests or linter are configured.

## Architecture

### Theme entry points

- **`vdoing/index.js`** — Node-side theme API entry. Called by VuePress with `(options, ctx)`. Handles:
  - `setFrontmatter()` — auto-generates frontmatter for all .md files
  - Auto-generates structured sidebar when `sidebar: 'structuring'` is configured
  - Auto-creates category/tag/archive pages in `docs/@pages/`
  - Registers all markdown container plugins (note, tip, warning, danger, details, cardList, etc.)

- **`vdoing/enhanceApp.js`** — Client-side app enhancement. Fixes ISO8601 date formats, applies author info, mixes in the `posts` mixin globally.

### Key conventions

- **Numbered directory naming**: Directories and files use numeric prefixes for ordering (e.g., `01.前端/`, `02.客户端/`). The sidebar parser in `vdoing/node_utils/getSidebarData.js` splits on the first dot to extract order and title.
- **`_posts/` directory**: Blog-style "fragmented" articles. These get `sidebar: auto` and are categorized under a configurable `categoryText` (default: "随笔").
- **Auto frontmatter**: `vdoing/node_utils/setFrontmatter.js` automatically adds `title`, `date`, `permalink`, `categories`, and `tags` to markdown files missing these fields. Uses `gray-matter` for parsing and `json2yaml` for writing.

### Theme structure (`vdoing/`)

```
layouts/         — Layout.vue (main), 404.vue
components/      — 28 Vue SFCs (Navbar, Sidebar, Page, Home, etc.)
global-components/ — CodeGroup, CodeBlock, Badge
mixins/          — posts.js (computed: $filterPosts, $sortPosts, $groupPosts, $categoriesAndTags), titleBadge.js
styles/          — Stylus stylesheets. palette.styl (CSS variables), config.styl (breakpoints/dimensions), code.styl/code-theme.styl (syntax highlighting)
util/            — Client-side utilities (path resolution, date formatting, URL encoding)
node_utils/      — Build-time Node.js utilities (setFrontmatter, getSidebarData, handlePage, readFileList)
```

### Site configuration (`docs/.vuepress/`)

- **`config.ts`** — Main VuePress config. Theme config, nav, sidebar (`'structuring'` for auto-generation), plugins.
- **`config/`** — Sub-modules: `baiduCode.ts`, `htmlModules.ts`, `sidebar.js` (manual sidebar reference, not currently used).
- **`styles/`** — Override styles (index.styl, palette.styl).
- **`plugins/love-me/`** — Custom plugin (like button).

## CI/CD

- **GitHub Actions** (`ci.yml`): On push to master — yarn install, build, push `docs/.vuepress/dist` to `gh-pages` using `GITHUB_TOKEN`.
- **Baidu Push** (`baiduPush.yml`): On push and daily at 23:00 UTC — submits URLs to Baidu for search indexing.
- **Vercel**: Configured via `vercel.json` with 1-year immutable cache on `/assets/js/` and `/assets/css/`.
