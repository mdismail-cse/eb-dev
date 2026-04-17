# Essential Blocks — Architecture Reference

> **TL;DR:** Mental model of the FREE plugin. Read when you need to know
> where things live, how the bootstrap flows, or which class does what.
> For pro, see `pro-architecture.md`. For shared controls, see
> `controls-api.md` + `controls-deep-dive.md`. For "where is X?", check
> `investigation-playbook.md` Recipe 6 instead.

Senior-dev mental model for the WPDevelopers/essential-blocks plugin. All
paths are relative to the free plugin root unless otherwise noted.

## 1. Top-level layout

| Path                        | Role                                                                                       |
|-----------------------------|--------------------------------------------------------------------------------------------|
| `essential-blocks.php`      | Plugin entry (30 lines). Defines `ESSENTIAL_BLOCKS_FILE`, requires autoload, bootstraps singleton `Plugin::get_instance()` via helper `wpdev_essential_blocks()` |
| `autoload.php`              | Minimal PSR-4 autoloader: `EssentialBlocks\Foo\Bar` → `includes/Foo/Bar.php`               |
| `includes/`                 | All PHP: core, blocks, API, admin, integrations, traits, utils                              |
| `src/`                      | All JS/React/SCSS source. Compiled to `dist/` and (older build) `assets/blocks/`            |
| `src/blocks/`               | 60+ block implementations — each a self-contained dir                                       |
| `src/controls/`             | **Git submodule.** Shared React controls (40+) used by both free and pro                    |
| `src/admin/`                | React apps for dashboard, quick-setup, pro-block-register                                   |
| `src/store/`                | Redux store (`@wordpress/data`) registered as `'essential-blocks'`                          |
| `src/global-styles/`        | Theme-level token system (colors, typography, spacing presets)                              |
| `src/modules/`              | Feature modules compiled separately (write-with-ai, ai-for-woo, templately-installer, …)    |
| `src/helpers/`              | Shared JS helpers used across blocks                                                        |
| `patterns/`                 | 11 JSON pattern files registered via `includes/Core/BlocksPatterns.php`                    |
| `templates/`                | Page templates (`essential-blocks-fullwidth-template.php`, blank template)                  |
| `views/`                    | PHP partials rendered by dynamic blocks (post-grid, carousel, forms, woo, instagram)        |
| `scripts/`                  | Release automation shell scripts (backup, validation, file-utils, notice-handler)           |
| `languages/`                | `.pot` file + translations                                                                  |
| `webpack.config.js`         | Build config with 40+ entry points                                                          |
| `Gruntfile.js`              | Legacy release tasks                                                                        |
| `package.json`              | npm/pnpm scripts                                                                            |
| `phpunit.xml.dist`          | PHPUnit config (tests dir may be sparse)                                                    |

Build output goes to `dist/` (and some legacy output to `assets/blocks/`).
Compiled bundles: `dist/admin/controls/controls.js` exposes `window.EBControls`
as the externals target.

## 2. PHP architecture

### Autoload

- Namespace base: `EssentialBlocks\`
- `EssentialBlocks\Core\Block` → `includes/Core/Block.php`
- `EssentialBlocks\Blocks\PostGrid` → `includes/Blocks/PostGrid.php`
- `EssentialBlocks\Traits\HasSingletone` → `includes/Traits/HasSingletone.php`
  (note the misspelling — it's `HasSingletone`, not `HasSingleton`, preserved for back-compat)

### Bootstrap sequence

```
essential-blocks.php
  → require autoload.php
  → wpdev_essential_blocks() → Plugin::get_instance()
     → Plugin::__construct()
        → define_constants()     (ESSENTIAL_BLOCKS_*: paths, urls, version, flags)
        → set_locale()           (load_plugin_textdomain)
        → instantiate services:
            Enqueue, Settings, Admin, StyleHandler, Scripts,
            FontLoader, Blocks (loader), Server (REST)
        → BlocksPatterns::init()
        → Integration init: Form, GoogleMap, Instagram, NFT, OpenVerse,
                            GlobalStyles, BlockUsage
        → add_action('init', register blocks + register post meta)
        → add_action('plugins_loaded', migrations/maintenance checks)
        → do_action('essential_blocks::init')   ← pro hooks in here
```

### Main classes to know

| Class                                        | File                                  | Role                                                                      |
|-----------------------------------------------|---------------------------------------|---------------------------------------------------------------------------|
| `EssentialBlocks\Plugin`                     | `includes/Plugin.php`                | Singleton entry. Defines all plugin constants, orchestrates services      |
| `EssentialBlocks\Core\Block` (abstract)       | `includes/Core/Block.php`            | Base for every block. `register_block_type()`, `load_frontend_*()`, `render_callback()` |
| `EssentialBlocks\Core\Blocks`                 | `includes/Core/Blocks.php`           | Block loader — reads `includes/blocks.php`, iterates, registers each      |
| `EssentialBlocks\Blocks\PostBlock`            | `includes/Blocks/PostBlock.php`      | Base for query-driven blocks (grid, carousel, meta, taxonomy, …)          |
| `EssentialBlocks\Admin\Admin`                 | `includes/Admin/Admin.php`           | Admin menu + AJAX handlers (`wp_ajax_*_eb_admin_options`, etc.)            |
| `EssentialBlocks\Utils\Settings`              | `includes/Utils/Settings.php`        | Wrapper around `get_option('eb_settings')`, fires `eb_after_save_*_settings` |
| `EssentialBlocks\Utils\Enqueue`               | `includes/Utils/Enqueue.php`         | Asset handle registration, editor vs frontend enqueue                     |
| `EssentialBlocks\Core\Scripts`                | `includes/Core/Scripts.php`          | Localizes editor assets via `EssentialBlocksLocalize`                     |
| `EssentialBlocks\Core\FontLoader`             | `includes/Core/FontLoader.php`       | Google Fonts + FontAwesome loader                                         |
| `EssentialBlocks\Modules\StyleHandler`        | `includes/Modules/StyleHandler.php`  | Generates dynamic per-post CSS files, caches to `wp-content/uploads/essential-blocks/` |
| `EssentialBlocks\API\Server`                  | `includes/API/Server.php`            | REST router — registers endpoints on `rest_api_init`                       |
| `EssentialBlocks\API\Base`                    | `includes/API/Base.php`              | REST endpoint base class with `get()`, `post()` helpers                   |
| `EssentialBlocks\API\PostBlock`               | `includes/API/PostBlock.php`         | Post query endpoints (grid/carousel)                                      |
| `EssentialBlocks\API\Product`                 | `includes/API/Product.php`           | WooCommerce product query endpoints                                       |
| `EssentialBlocks\Core\BlocksPatterns`         | `includes/Core/BlocksPatterns.php`   | Registers patterns from `patterns/*.json` on `admin_init`                 |
| `EssentialBlocks\Core\PageTemplates`          | `includes/Core/PageTemplates.php`    | Registers page templates from `templates/`                                |
| `EssentialBlocks\Integrations\Form`           | `includes/Integrations/Form.php`     | Form block submission + validation + email                                 |
| `EssentialBlocks\Utils\Helper`                | `includes/Utils/Helper.php`          | General utilities (responsive breakpoints, plugin checks, etc.)           |
| `EssentialBlocks\Utils\QueryHelper`           | `includes/Utils/QueryHelper.php`     | Post/product query builder used by post blocks and REST                   |

### Block registration flow

`includes/blocks.php` is the **block manifest** — a big PHP array where each
entry maps a snake_case key to metadata + an instantiated Block object:

```php
'post_grid' => array(
    'label'      => __('Post Grid', 'essential-blocks'),
    'value'      => 'post_grid',
    'visibility' => 'true',
    'status'     => 'popular',
    'category'   => 'content',
    'preferences'=> array('basic', 'advanced'),
    'object'     => PostGrid::get_instance(),
    'demo'       => ESSENTIAL_BLOCKS_SITE_URL . 'demo/post-grid/',
    'doc'        => ESSENTIAL_BLOCKS_SITE_URL . 'docs/post-grid/',
    'icon'       => ESSENTIAL_BLOCKS_ADMIN_URL . 'assets/blocks/post-grid/icon.svg',
),
```

`Blocks::register_blocks()` loops enabled entries and calls
`$block->register_block_type($slug)`, which in turn calls WP core
`register_block_type()` pointing at the compiled `block.json`.

**Visibility flag** lets the admin dashboard toggle blocks on/off. The setting
is read via `Settings::get('blocks')` and merged into the manifest.

### Free ↔ Pro interop

| Mechanism                               | Where                              | Pro uses it to…                                       |
|------------------------------------------|------------------------------------|--------------------------------------------------------|
| `ESSENTIAL_BLOCKS_IS_PRO_ACTIVE` const  | Defined in `Plugin::define_constants()` | Gate pro-only code in free                        |
| `do_action('essential_blocks::init')`   | End of `Plugin::__construct`       | Pro plugin bootstraps its own services                 |
| `apply_filters('essential_blocks_block_lists', $blocks)` | `Core/Blocks.php`       | Pro appends pro blocks to the manifest                  |
| `apply_filters('essential_blocks_block_path', $path)`    | `Core/Block.php`        | Pro can relocate block asset paths                      |
| `render_block` WP core filter            | `Plugin::filter_pro_blocks_frontend()` | Hides pro block output when pro is inactive         |

Pro blocks are registered by calling the same `Block` base class machinery
from the pro plugin — they're just autoloaded under a pro namespace.

### REST API

- Route base: `essential-blocks/v1/`
- Endpoints grouped in `includes/API/` (PostBlock, Product, Common, …)
- Permission callbacks default to `__return_true` for GET; POST typically
  verifies a nonce
- Always check `includes/API/Base.php` for the common wrappers

## 3. JavaScript / block architecture

### Typical block directory

```
src/blocks/<name>/
├── block.json
├── src/
│   ├── index.js          # Calls ebConditionalRegisterBlockType(metadata, { edit, save, attributes, ... })
│   ├── edit.js           # React editor component (biggest file for most blocks)
│   ├── save.js           # () => null for dynamic, JSX for static
│   ├── attributes.js     # Attributes schema (can be very large — post-grid ~19KB)
│   ├── frontend.js       # Frontend JS (vanilla) — sliders, AJAX, etc.
│   ├── style.scss        # Frontend styles
│   ├── editor.scss       # Editor-only styles
│   ├── constants/        # Option lists, preset definitions
│   ├── components/       # Internal React components (reusable within block)
│   └── icon.js / icon.svg
```

Some older blocks use `inspector.js` as a separate file for the sidebar
panels. Newer blocks inline InspectorControls in `edit.js`.

### Block entry point pattern

```javascript
import { ebConditionalRegisterBlockType } from "@essential-blocks/controls";
import metadata from "../block.json";
import Edit from "./edit";
import attributes from "./attributes";

ebConditionalRegisterBlockType(metadata, {
    keywords: [...],
    icon: Icon,
    attributes,
    edit: Edit,
    save: () => null,   // dynamic — server-rendered
});
```

`ebConditionalRegisterBlockType` is the controls-submodule wrapper that
handles pro/free visibility checks, category registration, and the
`eb_settings.blocks` enable/disable merge.

### Dynamic vs static

- **Dynamic:** `save: () => null` + `render_callback()` in PHP + a template
  in `views/`. Examples: post-grid, form, instagram-feed, all woo product blocks.
- **Static:** Custom `save()` returning JSX. Examples: button, icon,
  advanced-heading, notice. WP core saves the HTML inline to post content.

### Imports from controls

All blocks import controls via the alias:

```javascript
import {
    ResponsiveRangeController,
    ColorControl,
    TypographyDropdown,
    BackgroundControl,
    BorderShadowControl,
    ResponsiveDimensionsControl,
} from "@essential-blocks/controls";
```

At build time, webpack externalizes this alias to `window.EBControls`,
which is provided by the separately-built `dist/admin/controls/controls.js`
bundle. This keeps every block bundle small.

## 4. Controls submodule

Path: `src/controls/` (submodule of free) — independent git repo.

### Directory layout (inside `src/controls/src/`)

| Dir                   | What lives there                                                                 |
|-----------------------|----------------------------------------------------------------------------------|
| `controls/`           | 37 individual control components (see `controls-api.md` for the full list)       |
| `group-controls/`     | Grouped presets — `advanced-controls.js`, `layout-options.js`                    |
| `extensions/`         | Plugin-style extension system (animation effects, dynamic field extensions)      |
| `helpers/`            | CSS generation helpers — background, border, dimensions, typography, responsive  |
| `hoc/`                | Higher-order components — `withBlockContext`, AI wrappers                        |
| `hooks/`              | React hooks — `useBlockDefaults`, device type, attribute sync                    |
| `components/`         | Reusable UI (buttons, dropdowns, toggles)                                        |
| `icons/`              | Icon assets used across controls                                                  |
| `utils/`              | Small util functions                                                              |
| `index.js`            | Barrel export — everything available as `@essential-blocks/controls` re-exports |

### Group controls

These represent the "preset" system — one control that bundles multiple
settings (e.g., a typography preset that sets family + size + weight
+ line-height at once). They live under `group-controls/` and are typically
rendered inside an `<AdvancedControls>` wrapper in the block inspector.

## 5. Block lifecycle end-to-end (post-grid example)

PHP side:

1. `includes/blocks.php` — `'post_grid' => ['object' => PostGrid::get_instance(), …]`
2. `Core/Blocks::register_blocks()` loops enabled blocks → calls `$postGrid->register_block_type('post-grid')`
3. `Core/Block::register_block_type()` calls WP `register_block_type()` pointing at the compiled `block.json`
4. WP wires `render_callback` to `Blocks\PostGrid::render_callback($attributes, $content)`

JS side:

1. `src/blocks/post-grid/src/index.js` calls `ebConditionalRegisterBlockType(metadata, {edit, save: () => null, …})`
2. Edit component renders `InspectorControls` with controls from `@essential-blocks/controls`
3. Attributes flow through `setAttributes()` on change
4. Save returns null (dynamic), so only the block comment with attributes is written to post content

Frontend render:

1. WP calls `PostGrid::render_callback()` with serialized attributes
2. Method extracts `queryData`, calls `QueryHelper::get_posts()`
3. Loops posts and includes template from `views/post-grid.php`
4. Enqueues `essential-blocks-post-grid-frontend` script for pagination/filter/load-more

CSS:

1. Webpack compiles `style.scss` into a combined frontend CSS bundle
2. Dynamic per-attribute CSS generated by `Modules/StyleHandler.php` on save, cached as static file in uploads dir
3. Editor CSS comes from `editor.scss` only in editor context

## 6. Build system

See `build-and-dev.md` for full details. Essentials:

- `npm run start` — dev watch build via `@wordpress/scripts`
- `npm run build` — production build
- `npm run release` — scripted release + build
- `npm run pot` — regenerate `.pot`
- `npm run lint:js` / `lint:js:fix`

Webpack (`webpack.config.js`) defines 40+ entry points across:
- Admin controls library (`admin/controls/controls.js` → `window.EBControls`)
- Block editor bundle (`admin/editor/editor.js` — all block edits combined)
- Per-block frontend scripts (`blocks/<name>/frontend.js`)
- Admin dashboard (`admin/dashboard/admin.js`)
- Store (`admin/store/store.js`)
- Modules (separate per-module entries)
- Global styles (`admin/global-styles/global-styles.js`)

Aliases: `@essential-blocks/controls` → externalized to `window.EBControls`.

## 7. Admin, settings, modules

- Admin class: `includes/Admin/Admin.php` — registers `admin.php?page=eb-admin-blocks`, caps `manage_options`
- Dashboard UI is a React SPA in `src/admin/dashboard/` compiled to `dist/admin/dashboard/admin.js`
- Settings wrapper: `includes/Utils/Settings.php` — reads/writes `eb_settings` option
- AJAX surface:
  - `wp_ajax_save_eb_admin_options` — save general settings
  - `wp_ajax_get_eb_admin_options` — load current settings
  - `wp_ajax_reset_eb_admin_options` — reset to defaults
  - `wp_ajax_eb_save_quick_toolbar_blocks` — quick-toolbar config
  - `wp_ajax_hide_pattern_library` — pattern library dismiss
  - `wp_ajax_get_eb_admin_templates` — Templately integration
- Onboarding wizard: `includes/Admin/QuickSetup.php`
- Insights (opt-in anonymous tracking): `includes/Dependencies/Insights.php`
- Notice management via external dep `PriyoMukul\WPNotice`

## 8. Hooks and extensibility

**See `hooks-reference.md` for the complete list with line numbers.**

Quick recall — the 5 most important hooks:
- `essential_blocks::init` (`Plugin.php:168`) — pro boots here
- `essential_blocks_block_lists` (`Blocks.php:63`) — modify block registry
- `eb_frontend_styles/{$blockname}` (`Block.php:120`) — per-block CSS
- `eb_post_grid_query_results` (`PostGrid.php:129`) — modify query
- `eb_dynamic_tag_value` (`Form.php:492`) — resolve dynamic tags

## 9. Patterns, templates, views

- `patterns/` — 11 JSON files. Registered by `BlocksPatterns::init()` on `admin_init`. Can be disabled via `eb_settings['enablePatterns']`. Categories include `essential-blocks` plus per-block sub-categories.
- `templates/` — 2 page templates (`fullwidth`, `blank`) registered via `Core/PageTemplates.php`, hooked into `theme_page_templates` / `template_include`.
- `views/` — PHP partials required from dynamic block render callbacks. Organized by block/category: `views/post-grid.php`, `views/form-block.php`, `views/woocommerce/`, `views/common/`, `views/instagram-feed.php`, etc.

## 10. Testing

- `phpunit.xml.dist` is present but the tests dir is minimal — don't assume coverage
- ESLint config via `@wordpress/scripts` (`npm run lint:js`)
- No Playwright/e2e scaffolding in the repo

## 11. Must-know files (15 essentials)

| File | Why |
|---|---|
| `essential-blocks.php` | Plugin entry — 30 lines, just bootstraps singleton |
| `autoload.php` | PSR-4 autoloader |
| `includes/Plugin.php` | Main singleton, defines all constants, orchestrates services |
| `includes/blocks.php` | Block registry — full manifest array |
| `includes/Core/Block.php` | Abstract base for every block |
| `includes/Core/Blocks.php` | Block loader, applies enable/disable settings |
| `includes/Blocks/PostBlock.php` | Base for query-driven blocks (grid, carousel) |
| `includes/Blocks/PostGrid.php` | Biggest dynamic example — read this to understand the pattern |
| `includes/Modules/StyleHandler.php` | Dynamic CSS generation (see `css-pipeline.md`) |
| `includes/Admin/Admin.php` | Admin menu + AJAX handlers |
| `includes/Utils/Settings.php` | `eb_settings` option wrapper, fires save/reset hooks |
| `includes/Utils/QueryHelper.php` | All post queries route through this |
| `includes/Integrations/Form.php` | Form submission, validation, hooks |
| `src/controls/src/index.js` | Controls barrel export |
| `webpack.config.js` | Build entries — 40+ |

For lookup-by-purpose ("where is X defined?"), use
`investigation-playbook.md` Recipe 6.

## 12. Conventions & gotchas

**Naming:**
- JS slug kebab, PHP class Pascal, WP block name `essential-blocks/<kebab>`
- Attribute names camelCase (`columnNumber`, not `column_number`)
- Option keys `eb_*` snake_case
- Hook names `eb_*` snake_case

**Attribute storage:**
- Block comment in post content (standard Gutenberg)
- Responsive values use flat keys: `columnNumberDesktop` / `columnNumberTablet` / `columnNumberMobile`

**Singleton trait:**
- `HasSingletone` (misspelled, intentional) — every major class uses it
- Never `new ClassName()` — always `ClassName::get_instance()`

**Dynamic block checklist:**
- `save: () => null` in JS
- `render_callback()` method in PHP block class
- Template file in `views/`
- Frontend script handle registered in `register_scripts()` and listed in `$frontend_scripts`

**Controls value handling:**
- Responsive controls store three device-specific values, not a nested object
- Group controls may store sub-values as flat sibling attributes
- Always provide defaults in PHP `$default_attributes` — undefined values break inline CSS generation

**Styling tier-order:**
1. Webpack SCSS (static, shipped once) — `dist/admin/editor/frontend.css`
2. Dynamic inline CSS (generated from attributes) — `StyleHandler::generate_styles()`
3. Cached static CSS file in `wp-content/uploads/essential-blocks/` — per-post optimization

**QueryHelper is the only way:**
- All post queries go through `Utils/QueryHelper.php`
- Don't write raw `new WP_Query()` in block PHP — use `QueryHelper::get_posts($queryData)`

**Forms are special:**
- Form block integrates with contact plugins (WPForms, Fluent Forms)
- Submission validated via `eb_form_data_validation` filter
- Integrations register via `eb_form_block_integrations` action

**Pro gate:**
- Every pro block PHP class should check `ESSENTIAL_BLOCKS_IS_PRO_ACTIVE`
- The `render_block` filter in `Plugin::filter_pro_blocks_frontend()` is the backstop

**Submodule pointer:**
- `src/controls` is a git submodule with its own commits (~1,377 commits as of v5.8.1)
- When a free PR touches `src/controls`, the pointer moves — always check `git diff origin/master -- src/controls`

## Quick cross-references

→ See `investigation-playbook.md` Recipe 6 — "Where is X defined?"
(deduplicated; one source of truth for path lookups).
