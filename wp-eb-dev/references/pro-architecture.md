# Essential Blocks Pro — Architecture Reference

Pro plugin repo: `https://github.com/WPDevelopers/essential-blocks-pro`
Bootstrap file: `essential-blocks-pro.php`
Namespace root: `EssentialBlocks\Pro\`

Pro does not stand alone — it **requires the free Essential Blocks plugin
to be active**. It hooks into the free plugin's `essential_blocks::init`
action, reuses the free plugin's `Core\Blocks` loader machinery, and
extends free blocks via its own `src/blocks-extends/` system.

## 1. Quick facts (as of origin/main 5b7dcb68e, 2026-04-02)

| Metric                                    | Value                                          |
|--------------------------------------------|------------------------------------------------|
| Total commits on `main`                    | 2,395                                          |
| Timespan                                    | 2023-02-27 → 2026-04-02 (~3 years)             |
| Contributors (top 10 shown)                 | Sumaiya Siddika (1009), Monir (622 combined), Jamil (634 combined), Rahat (118), Priyo (12) |
| Default branch                              | `main` (NOT `master`)                          |
| First commit                                | `482929ab` "Setup Commit" by Priyo Mukul, 2023-02-27 |
| First block                                 | `d9ea20e3` "add advanced search block for test build" by Jamil, 2023-03-05 |
| Latest release tag                          | `2.7.9` (or `v2.7.8` — naming varies)          |
| Standalone pro blocks                       | 22 (in `includes/Blocks/`)                      |
| Block extends (free-block enhancements)     | 7 (in `src/blocks-extends/`)                    |
| Integrations                                | 7 (AdvSearch, DataTable, DynamicFields, FancyChart, FormPro, LoopPagination, PostGridSearch) |
| Release tags                                | 45                                              |

Peak activity: 2023-05 (294 commits) during early buildout. Steady
~30-80/month since mid-2024.

## 2. Top-level layout

| Path                           | Role                                                                      |
|--------------------------------|---------------------------------------------------------------------------|
| `essential-blocks-pro.php`    | Plugin entry. Checks free plugin active, bootstraps on `essential_blocks::init` |
| `includes/Plugin.php`         | Main singleton, `EssentialBlocks\Pro\Plugin`                              |
| `includes/blocks.php`         | Pro block manifest (17 top-level entries) — returned array, merged into free's Blocks loader |
| `includes/Blocks/`            | 23 PHP block classes                                                       |
| `includes/Core/`              | Pro core (`Scripts`, `FrontendStyles`, `PostMeta`, `Maintenance`, `ConditionalMaintainance`, `DynamicTags/`) |
| `includes/Admin/`             | `Admin`, `FormResponseTable` — extends free's admin                       |
| `includes/Integrations/`      | Feature integrations — data table, charts, dynamic fields, form, search   |
| `includes/Utils/`             | Pro helpers — `FormBlockHandler`, `PostGrid`, `WooProductGrid`, `ConditionalDisplay`, `LiquidGlassRendererPro` |
| `includes/Deps/`              | Vendored deps — `WPDeveloper\Licensing` (EDD Software Licensing client)   |
| `src/blocks/`                 | 22 standalone pro block JS dirs                                            |
| `src/blocks-extends/`         | 7 "extends" — adds features to existing free blocks                       |
| `src/blocks-extends/` entries | `countdown`, `form`, `image-gallery`, `post-block`, `post-carousel`, `post-grid`, `woo-product-grid` |
| `src/controls/`               | **Pro-specific controls** (not the shared submodule)                       |
| `src/admin/`                  | Pro admin UI (dashboard enhancements, form responses)                      |
| `src/global-styles/`          | Pro global styles extensions                                                |
| `src/helpers/`                | Pro JS helpers                                                              |
| `src/templates/`              | Pattern/template data                                                        |
| `patterns/`                   | Pro-only block patterns                                                      |
| `views/`                      | Server-render templates for dynamic pro blocks                              |
| `languages/`                  | i18n                                                                         |
| `composer.json` + `vendor/`   | Composer-managed PHP deps (licensing SDK)                                   |
| `webpack.config.js`           | Build config                                                                 |

**Key difference from free:** Pro uses `composer` in addition to `pnpm`/`npm`,
because the licensing SDK (`WPDeveloper\Licensing`) is a PHP dep pulled via
Composer and committed under `includes/Deps/` / `vendor/`.

## 3. Bootstrap sequence

```
essential-blocks-pro.php
  → define( 'ESSENTIAL_BLOCKS_PRO_FILE' ), VERSION, REQUIRED_VERSION, DB table consts
  → require vendor/autoload.php   (composer autoload)
  → include wp-admin/includes/plugin.php (for is_plugin_active check)
  → if free plugin active → Maintenance::get_instance()
    else                  → ConditionalMaintainance::get_instance()
  → function wpdev_essential_blocks_pro()
     → if ( ! did_action( 'essential_blocks::init' ) ) return;
     → return \EssentialBlocks\Pro\Plugin::get_instance()
  → add_action( 'essential_blocks::init', 'wpdev_essential_blocks_pro', ...)
```

The critical check: `did_action('essential_blocks::init')`. Pro only boots
**after** the free plugin has fired its `essential_blocks::init` action at
the end of `free::Plugin::__construct()`. Without free → pro is inert.

### Plugin class init order

`EssentialBlocks\Pro\Plugin::__construct()` at `includes/Plugin.php`:

1. `define_constants()` — pro-specific constants
2. `set_locale()` — text domain `'essential-blocks-pro'`
3. Instantiate services:
   - `Enqueue` (from free), `Settings` (from free) — pro shares these
   - `Admin` (pro) — extends free admin menu
   - `Scripts` (pro) — enqueues pro editor assets
   - `FrontendStyles` (pro) — pro style generation
   - `PostMeta` (pro)
4. Register blocks:
   - `Blocks::get_instance()->register_blocks()` — reuses free loader with pro's manifest
5. Initialize integrations:
   - `FormPro`, `AdvSearch`, `DataTable`, `FancyChart`, `DynamicFields`, `PostGridSearch`
   - Dynamic tags: `AcfData`, `PostFields`, `SiteFields`, `HandleTagsResult`
   - Utils: `ConditionalDisplay`, `LiquidGlassRendererPro`
6. Fire `essential_blocks_pro::init` action — for third parties extending pro

## 4. Pro block manifest (`includes/blocks.php`)

Returns an array of block-slug → object. **It's a `return` array**, not a
mutating call — the free loader merges it in:

```php
// Excerpt from includes/blocks.php
return [
    'advanced_search'           => [ 'object' => AdvancedSearch::get_instance() ],
    'data_table'                => [ 'object' => DataTable::get_instance() ],
    'timeline_slider'           => [ 'object' => TimelineSlider::get_instance() ],
    'news_ticker'               => [ 'object' => NewsTicker::get_instance() ],
    'woo_product_carousel'      => [ 'object' => WooProductCarousel::get_instance() ],
    'form'                      => [ 'object' => ProForm::get_instance() ],
    'fancy_chart'               => [ 'object' => FancyChart::get_instance() ],
    'multicolumn_pricing_table' => [ 'object' => MultiColumnPricingTable::get_instance() ],
    'stacked_cards'             => [ 'object' => StackedCards::get_instance() ],
    'testimonial_slider'        => [ 'object' => TestimonialSlider::get_instance() ],
    'off_canvas'                => [ 'object' => OffCanvas::get_instance() ],
    'mega_menu'                 => [ 'object' => MegaMenu::get_instance() ],
    'mega_menu_item'            => [ 'object' => MegaMenuItem::get_instance() ],
    'loop_builder'              => [ 'object' => LoopBuilder::get_instance() ],
    'post_template'             => [ 'object' => PostTemplate::get_instance() ],
    'loop_pagination'           => [ 'object' => LoopPagination::get_instance() ],
    'form_phone_field'          => [ 'object' => PhoneField::get_instance() ],
];
```

**Note:** `'form'` in pro's manifest points to `ProForm::get_instance()` —
which replaces/extends the free Form block. Pro can override a free slug
by registering the same key.

## 5. Standalone pro blocks

| Registry key                     | PHP class                    | JS dir                        | Notes                                    |
|----------------------------------|------------------------------|-------------------------------|------------------------------------------|
| `advanced_search`                | `AdvancedSearch`             | `advanced-search/`            | Search block with filters + AJAX         |
| `data_table`                     | `DataTable`                  | `data-table/`                 | Sortable/paginated data table            |
| `timeline_slider`                | `TimelineSlider`             | `timeline-slider/`            | Timeline with slider navigation          |
| `news_ticker`                    | `NewsTicker`                 | `news-ticker/`                | Scrolling news ticker                    |
| `woo_product_carousel`           | `WooProductCarousel`         | `woo-product-carousel/`       | WooCommerce product carousel             |
| `form` (override)                | `ProForm`                    | —                              | Replaces free form with pro features     |
| `fancy_chart`                    | `FancyChart`                 | `fancy-chart/`                | Chart.js-based block                     |
| `multicolumn_pricing_table`      | `MultiColumnPricingTable`    | `multicolumn-pricing-table/`  | Multi-column pricing                     |
| `pricing_column`                 | `PricingColumn`              | `pricing-column/`             | Child of multicolumn pricing             |
| `pricing_cell`                   | `PricingCell`                | `pricing-cell/`               | Child of pricing column                  |
| `stacked_cards`                  | `StackedCards`               | `stacked-cards/`              | Stacked scroll cards                     |
| `testimonial_slider`             | `TestimonialSlider`          | `testimonial-slider/`         | Testimonial carousel                     |
| `off_canvas`                     | `OffCanvas`                  | `off-canvas/`                 | Off-canvas menu/drawer                   |
| `mega_menu`                      | `MegaMenu`                   | `mega-menu/`                  | Mega menu container (most-touched pro block) |
| `mega_menu_item`                 | `MegaMenuItem`               | `mega-menu-item/`             | Mega menu item                           |
| `loop_builder`                   | `LoopBuilder`                | `loop-builder/`               | Query loop builder (modern Gutenberg loop) |
| `post_template`                  | `PostTemplate`               | `post-template/`              | Child of loop_builder — template area    |
| `loop_pagination`                | `LoopPagination`             | `loop-pagination/`            | Pagination for loop_builder              |
| `form_phone_field`               | `PhoneField`                 | `form-phone-field/`           | Child of form — phone input              |
| `form_country_field`             | `CountryField`               | `form-country-field/`         | Child of form — country select           |
| `form_datetime_picker`           | `DateTimePicker`             | `form-datetime-picker/`       | Child of form — datetime                 |
| `form_recaptcha`                 | `Recaptcha`                  | `form-recaptcha/`             | reCAPTCHA integration                    |
| `form_multistep_wrapper`         | `MultistepWrapper`           | `form-multistep-wrapper/`     | Multi-step form container                |

Some pro blocks (`pricing_column`, `pricing_cell`, `post_template`) are
**child blocks** constrained via `parent` in block.json to their parents.

## 6. Block-extends system (`src/blocks-extends/`)

This is pro's most important cross-plugin mechanism. Instead of overriding
a free block entirely, pro **extends** it by adding inspector panels,
attributes, or frontend behavior to existing free blocks.

Extended free blocks:

- `countdown/` — pro adds advanced countdown formats
- `form/` — pro adds validation, integrations, field types (heavily used — 26+ commits)
- `image-gallery/` — pro adds filterable gallery, lightbox (27+ commits)
- `post-block/` — shared pro extension for all post blocks (grid, carousel, etc.)
- `post-carousel/` — pro-specific carousel features
- `post-grid/` — pro post grid enhancements (search, advanced filters)
- `woo-product-grid/` — pro woo grid enhancements

### How block-extends works

Each extend dir typically has:

```
src/blocks-extends/<name>/
├── index.js                 # Entry — hooks into the free block via wp.hooks / custom registration
├── frontend.js              # Additional frontend behavior (AJAX, interactions)
├── style.scss               # Additional styles
├── helper/
│   └── inspector.js         # Additional inspector panels
└── attributes.js            # Additional attributes
```

The pattern: use `wp.hooks.addFilter` (or equivalent) to inject into the
free block's inspector, or register additional behavior on the frontend
script that the free block already enqueues. Pro's PHP side may also
`add_filter('eb_frontend_styles/<name>', ...)` to enqueue additional CSS.

When reading an extend, trace both sides:
- JS: `src/blocks-extends/<name>/` — how it hooks into the free block's React tree
- PHP: `includes/Utils/PostGrid.php` / `FormBlockHandler.php` / etc. — server-side enhancements

## 7. Dynamic Tags system (pro-only)

Path: `includes/Core/DynamicTags/`

Pro introduces a full dynamic content system:

- `HandleTagsResult.php` — resolves `{{ tag_name }}` placeholders in content
- `Acf/AcfData.php` — resolves from ACF fields (Advanced Custom Fields)
- `Post/PostFields.php` — resolves from post fields (title, excerpt, meta)
- `Site/SiteFields.php` — resolves from site fields (name, tagline, URL)

The entry filter is `eb_dynamic_tag_value` (declared in free, listened
in pro). Pro's `HandleTagsResult` subscribes and runs resolution.

**Key files:**
- `includes/Integrations/DynamicFields.php` — REST routes + registration
- `includes/Core/DynamicTags/HandleTagsResult.php` (33+ commits — heavily iterated)

## 8. Form enhancements

Pro substantially extends the form block. The entry is `ProForm`
(replaces free `Form` in manifest) + `includes/Utils/FormBlockHandler.php`
(41+ commits — one of the hottest files in pro).

Pro adds:
- Form responses table (`ESSENTIAL_BLOCKS_FORM_ENTRIES_TABLE` — DB table created on activation)
- `includes/Admin/FormResponseTable.php` — WP_List_Table for admin
- Multi-step forms (`form_multistep_wrapper` block)
- Field types: phone, country, datetime, reCAPTCHA
- `eb_form_before_validation` + `eb_form_data_validation` listeners

Pro owns table: `{$wpdb->prefix}eb_form_entries`.

## 9. Licensing

Pro uses **EDD Software Licensing** for license key validation and auto-updates:

- Composer package: `deliciousbrains/wp-background-processing` and
  `WPDeveloper\Licensing\` (vendored in `includes/Deps/WPDeveloper/Licensing/`)
- License setting stored in pro options
- Filters used: `edd_sl_api_request_verify_ssl`, `edd_sl_plugin_updater_api_params`
- Plugin update check: `in_plugin_update_message-{$file}` action

When debugging a "pro not updating" issue, check:
1. License key valid and active
2. `ESSENTIAL_BLOCKS_PRO_VERSION` constant matches
3. EDD SL API endpoint reachable
4. Admin notices in `Admin::admin_notices()`

## 10. Custom hooks exposed by pro

Fired from pro PHP:

| Hook                                    | Type    | Purpose                                                 |
|-----------------------------------------|---------|---------------------------------------------------------|
| `essential_blocks_pro::init`            | action  | End of pro plugin bootstrap. Pro extensions hook here   |
| `eb_news_ticker_query_results`          | filter  | Modify news ticker queried posts                        |
| `eb_timeline_slider_query_results`      | filter  | Modify timeline slider queried posts                    |
| `eb_dynamic_tag_form_field_value`       | filter  | Resolve dynamic tag values inside form fields           |

Pro also **listens** to many free hooks:
- `essential_blocks::init` — pro's bootstrap entry
- `eb_dynamic_tag_value` — pro's dynamic tags handler
- `eb_form_before_validation` — pro's form validation
- `eb_form_block_integrations` — pro's integrations registration
- `eb_post_grid_query_results` — pro's post grid search/filter enhancements
- `essential_blocks_block_lists` — pro appends its blocks to free's manifest

## 11. File hotspots (top 20 source files on pro origin/main)

```
  86  essential-blocks-pro.php
  71  package.json
  44  includes/Core/Scripts.php
  41  includes/blocks.php
  41  includes/Utils/FormBlockHandler.php
  40  src/blocks/mega-menu/src/style.scss
  35  src/blocks/business-hours/src/inspector.js
  34  includes/Plugin.php
  33  includes/Core/DynamicTags/HandleTagsResult.php
  31  src/blocks/mega-menu/src/attributes.js
  30  src/blocks/business-hours/src/save.js
  29  src/blocks/mega-menu/src/inspector.js
  29  src/blocks/business-hours/src/edit.js
  29  includes/Integrations/AdvSearch.php
  28  views/timeline-slider.php
  27  src/blocks/mega-menu/src/edit.js
  27  src/blocks-extends/image-gallery/src/index.js
  26  src/blocks-extends/form/index.js
  26  src/blocks-extends/form/frontend.js
  26  includes/Utils/Helper.php
```

**Reading this:** mega-menu is the most-iterated standalone pro block.
`business-hours` shows up even though it's not in the 22 listed — it was
built, committed, maybe not yet in the canonical manifest (or it's
registered through a different path). Worth a grep if someone asks about it.

## 12. Pro branching conventions

Pro follows a **different** issue-ID convention than free:

- **4-digit branch names** like `1005-testimonial-slider`, `1104-security-issue-all-blocks`,
  `1082-bug-fix-post-grid`, `1110-reduce-google-font-load`
- Branch numbers start at ~1000 and increment
- Some branches use 5-digit numbers matching free's issue IDs
  (e.g., `77860-Optimize-assets-new`) when the work crosses both repos
- Release branches: `release-2.7.0`, `release-2.7.1`, …, `release-2.7.9`
- Dev branches: `sumaiya-dev` (seen in pro), individual devs

**Merging pattern:** pro merges directly into `main` (not `master`) via
PRs from release branches (e.g., `#120 from WPDevelopers/release-2.7.9`).

## 13. Pro ↔ Free alignment

Pro versions track free versions loosely:

| Free version | Pro version (typical) | Notes                                    |
|--------------|------------------------|------------------------------------------|
| v6.0.7       | v2.7.8 / 2.7.9        | April 2026                                |
| v6.0.4       | v2.7.4                | February 2026                             |
| v5.9.0       | v2.7.1                | December 2025                             |
| v5.8.0       | v2.6.1                | November 2025                             |
| v5.7.0       | v2.5.0                | September 2025                            |
| v5.5.0       | v2.2.0                | June 2025                                 |

Pro enforces `ESSENTIAL_BLOCKS_REQUIRED_VERSION` (free minimum). If free
is older than the required version, pro won't boot — it shows a notice via
`ConditionalMaintainance`.

## 14. Red flags specific to pro

When reviewing pro code:

- [ ] License key handling — never log keys, never expose via REST
- [ ] EDD SL API calls — verify SSL, don't disable verification
- [ ] Form entries table — `$wpdb->prepare()` mandatory for all queries
- [ ] Dynamic tags resolution — sanitize output (user-controlled input)
- [ ] ACF integration — check field exists before dereferencing (fatals in the wild)
- [ ] Pro block render in frontend — always rely on free's `filter_pro_blocks_frontend()` backstop

## 15. Quick cross-reference

| "Where is…"                                  | Free                                          | Pro                                              |
|-----------------------------------------------|-----------------------------------------------|--------------------------------------------------|
| Main plugin class                             | `includes/Plugin.php`                         | `includes/Plugin.php`                            |
| Block manifest                                | `includes/blocks.php`                         | `includes/blocks.php` (returns array)            |
| Block loader                                  | `includes/Core/Blocks.php`                    | Reuses free's loader                              |
| Block base class                              | `includes/Core/Block.php`                     | Extends free's `Block`                            |
| Admin class                                   | `includes/Admin/Admin.php`                    | `includes/Admin/Admin.php`                       |
| Settings                                      | `includes/Utils/Settings.php`                 | Reuses free's Settings                            |
| REST router                                   | `includes/API/Server.php`                     | Via integrations (`DynamicFields`, etc.)         |
| Init action                                   | `essential_blocks::init`                      | `essential_blocks_pro::init`                     |
| Post query helper                             | `includes/Utils/QueryHelper.php`              | `includes/Utils/PostGrid.php`, `WooProductGrid.php` |

## 16. Investigation starting points

If a user reports a pro bug, the fastest path to context:

1. Identify the block or integration: standalone pro block, block-extends,
   or an integration like data-table/fancy-chart/form
2. For standalone blocks: `includes/Blocks/<Name>.php` + `src/blocks/<kebab>/`
3. For block-extends (e.g., "form validation broken"):
   - JS: `src/blocks-extends/form/`
   - PHP: `includes/Utils/FormBlockHandler.php` + `includes/Integrations/FormPro.php`
4. For dynamic tags: `includes/Core/DynamicTags/HandleTagsResult.php`
5. For license/update issues: `includes/Deps/WPDeveloper/Licensing/`
6. Cross-reference with free: if pro extends a free block, check the free
   block's current implementation too — the bug may be in free
