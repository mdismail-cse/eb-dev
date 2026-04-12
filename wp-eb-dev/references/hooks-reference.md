# Essential Blocks — Hooks & Filters Reference

Quick lookup for every custom action/filter exposed by the Essential Blocks
free plugin. Use these when planning pro extensions, investigating why code
runs (or doesn't), or tracing how a value flows through the plugin.

Line numbers move — these are accurate as of `origin/master` ~v6.0.7
(fc51997e8, 2026-04-02). Use `grep -rn "do_action\|apply_filters" includes/`
to confirm current positions.

## Action hooks (do_action)

### Plugin lifecycle

| Hook                                    | Fired from                      | Line | When                                                    |
|-----------------------------------------|---------------------------------|------|---------------------------------------------------------|
| `essential_blocks::init`                | `includes/Plugin.php`           | 168  | End of `Plugin::__construct()` — pro hooks here         |

This is the primary extension point for the pro plugin. Pro listens to it to
register its own blocks, REST routes, and integrations.

### Settings lifecycle

All fired from `includes/Utils/Settings.php`:

| Hook                                    | Line | When                                                                    |
|-----------------------------------------|------|-------------------------------------------------------------------------|
| `eb_after_save_all_settings`            | 39   | After the full settings array is persisted                               |
| `eb_after_save_{$key}_settings`         | 59   | After an individual settings key is saved                               |
| `eb_after_reset_{$key}_settings`        | 83   | After an individual settings key is reset to default                    |

These are the place to invalidate caches or rebuild indices when admin
settings change.

### Frontend assets

| Hook                                    | Fired from                              | Use for                                     |
|-----------------------------------------|-----------------------------------------|---------------------------------------------|
| `eb_frontend_assets`                    | `includes/Modules/StyleHandler.php`     | React to frontend asset generation          |

### Form submission flow

All fired from `includes/Integrations/Form.php`:

| Hook                                    | When                                                                         |
|-----------------------------------------|------------------------------------------------------------------------------|
| `eb_form_block_integrations`            | Form bootstrap — third-party form plugins hook here to register              |
| `eb_form_submit_before_email`           | Before the notification email is sent — modify recipients/body               |
| `eb_form_submit_after_email`            | After the notification email is sent — side effects (logging, CRM, etc.)    |

### WordPress core hooks the plugin listens to

| Hook            | Listener                                      | Purpose                                                  |
|-----------------|-----------------------------------------------|----------------------------------------------------------|
| `init`          | `Plugin::init_hooks()`                        | Register blocks, register post meta                      |
| `plugins_loaded`| `Plugin::plugins_loaded()`                    | Run migrations / maintenance checks                      |
| `wp_loaded`     | `Plugin::wp_loaded()`                         | Late bootstrap                                            |
| `rest_api_init` | `API\Server::register_routes()`               | Register REST endpoints                                   |
| `admin_init`    | `Core\BlocksPatterns::init()`                 | Register block patterns                                   |
| `admin_menu`    | `Admin\Admin::admin_menu()`                   | Register admin page                                        |
| `render_block`  | `Plugin::filter_pro_blocks_frontend()`        | Strip pro block output when pro inactive                  |

## Filter hooks (apply_filters)

### Block registry

| Filter                                                       | Fired from                      | Line | Default value          | Used to…                                                   |
|--------------------------------------------------------------|---------------------------------|------|------------------------|------------------------------------------------------------|
| `essential_blocks_block_lists`                              | `includes/Core/Blocks.php`      | 63   | block manifest array  | Add/remove blocks from the registry (pro appends blocks)   |
| `essential_blocks_block_path`                                | `includes/Core/Block.php`       | 66   | block file path       | Override where a block's compiled assets live              |

### Per-block frontend assets

| Filter                                           | File:Line                          | Default value       | Used to…                                             |
|--------------------------------------------------|------------------------------------|---------------------|------------------------------------------------------|
| `eb_frontend_styles/{$blockname}`                | `Core/Block.php:120`               | `$frontend_styles`  | Add/remove frontend CSS handles for a specific block |
| `eb_frontend_scripts/{$blockname}`               | `Core/Block.php:121`               | `$frontend_scripts` | Add/remove frontend JS handles for a specific block  |
| `eb_fixed_frontend_styles/{$blockname}`          | `Modules/StyleHandler.php:297`     | CSS string          | Modify fixed CSS injected per-block                  |
| `eb_generated_css_frontend_deps`                 | `Modules/StyleHandler.php:103`     | deps array          | Add CSS file dependencies                            |

`{$blockname}` is the dashed block name without the `essential-blocks/`
prefix, e.g., `eb_frontend_styles/post-grid`. These are the cleanest way to
extend a block's asset list without editing the block class.

### Query result modification

| Filter                                | Fired from                                | Line | Default value    | Used to…                           |
|---------------------------------------|-------------------------------------------|------|------------------|------------------------------------|
| `eb_post_grid_query_results`          | `includes/Blocks/PostGrid.php`            | 129  | WP_Query posts   | Modify queried posts before render |
| `eb_post_carousel_query_results`      | `includes/Blocks/PostCarousel.php`        | 114  | WP_Query posts   | Same, for carousel                 |

Use these to filter posts by custom logic (hide expired, inject promoted
posts, reorder by weight) without monkey-patching the block class.

### Form block customization

All in `includes/Integrations/Form.php` except where noted:

| Filter                                | Line                | Default value              | Used to…                                                    |
|---------------------------------------|---------------------|----------------------------|-------------------------------------------------------------|
| `eb_form_before_validation`           | `Form.php:56`       | form fields array          | Modify submitted fields before validation                   |
| `eb_form_data_validation`             | `Form.php:485`      | validation result (bool)   | Custom validation rules                                      |
| `eb_form_submit_btn_attr`             | `Form.php:114`      | attributes array           | Customize submit button attributes (data-*, target, etc.)   |
| `eb_form_submit_btn_classes`          | `Form.php:119`      | classes string             | Modify submit button CSS classes                             |
| `eb_form_confirmation_div_attr`       | `Form.php:139`      | attributes array           | Customize confirmation message div attributes               |
| `eb_dynamic_tag_value`                | `Form.php:492` + `TableOfContents.php:237` | resolved value | Resolve dynamic tag values (post meta, custom fields)   |

### Conditional display

| Filter                                | Fired from                  | Default value    | Used to…                             |
|---------------------------------------|-----------------------------|-------------------|--------------------------------------|
| `eb_conditional_display_results`      | `includes/Core/Block.php`   | bool (visible?)  | Hide/show blocks based on custom logic |

### Niche / newer filters

| Filter                                  | Fired from                                  | Purpose                                 |
|------------------------------------------|---------------------------------------------|-----------------------------------------|
| `eb_liquid_glass_svg_rendered`          | `includes/Modules/LiquidGlassRenderer.php`  | Cache liquid-glass SVG output          |

## JavaScript hooks (wp.hooks)

Essential Blocks does not expose an extensive JS hook surface in the free
version. A few blocks register internal hooks via `@wordpress/hooks`, but
there's no published top-level JS extension API. The primary extension
channel for pro and third parties is the PHP filter system above.

When a block **does** use `wp.hooks`:
- Hook prefix convention: `eb.<hookName>`
- Callers are usually in `src/blocks/<name>/src/edit.js`
- Pro extensions add filters via `addFilter('eb.<hookName>', 'eb-pro/<slug>', cb)`

Grep to find current JS hooks:

```bash
grep -rn "applyFilters\|addFilter\|doAction\|addAction" "$EB/src/blocks/" --include="*.js" --include="*.jsx"
```

## How to trace where a hook is used

When you need to know "what listens to hook X":

```bash
EB="<free path>"
EB_PRO="<pro path>"

# Find who fires it (do_action / apply_filters):
grep -rn "do_action[\\(]\\? *['\"]eb_post_grid_query_results" "$EB"
grep -rn "apply_filters[\\(]\\? *['\"]eb_post_grid_query_results" "$EB"

# Find who subscribes to it (add_action / add_filter):
grep -rn "add_action[\\(]\\? *['\"]eb_post_grid_query_results" "$EB" "$EB_PRO"
grep -rn "add_filter[\\(]\\? *['\"]eb_post_grid_query_results" "$EB" "$EB_PRO"
```

For settings-related dynamic hook names like `eb_after_save_{$key}_settings`,
grep with a wildcard:

```bash
grep -rn "eb_after_save_.*_settings" "$EB" "$EB_PRO"
```

## Common extension patterns

**Pro plugin adding a block:**
```php
add_filter('essential_blocks_block_lists', function($blocks) {
    $blocks['my_pro_block'] = [
        'label'      => __('My Pro Block', 'eb-pro'),
        'value'      => 'my_pro_block',
        'visibility' => 'true',
        'object'     => \EssentialBlocks\Pro\Blocks\MyProBlock::get_instance(),
        // ...
    ];
    return $blocks;
});
```

**Adding a frontend asset to an existing block:**
```php
add_filter('eb_frontend_styles/post-grid', function($styles) {
    $styles[] = 'my-custom-post-grid-styles';
    return $styles;
});
```

**Modifying post-grid query results:**
```php
add_filter('eb_post_grid_query_results', function($posts, $attributes, $block) {
    // e.g., prepend a featured post
    array_unshift($posts, $featured_post);
    return $posts;
}, 10, 3);
```

**Reacting to form submission:**
```php
add_action('eb_form_submit_after_email', function($form_data, $form_attributes) {
    // e.g., sync to CRM
}, 10, 2);
```

## Gotchas

- **Hook names with dynamic segments** (`eb_after_save_{$key}_settings`,
  `eb_frontend_styles/{$blockname}`) — when grep'ing, use the static prefix
  or regex; listeners hard-code the full name.
- **Filters vs actions** — EB sometimes uses actions where a filter would be
  cleaner (especially around form flow). Check the signature in the file
  before assuming.
- **`$blockname` format** — hooks use the dashed name without the
  `essential-blocks/` prefix. `essential-blocks/post-grid` becomes `post-grid`
  in hook names.
- **Pro plugin hooks** — pro exposes its own `eb_pro_*` hooks for its own
  blocks. Those aren't listed here; check `essential-blocks-pro/includes/`.
