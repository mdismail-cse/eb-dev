# Essential Blocks — CSS Pipeline & StyleHandler

> **TL;DR:** Full attribute → CSS file → frontend pipeline trace.
> Read for ANY "CSS not applying", "stale styles", "block validation
> error", or `deprecated` migration question. Includes the 4-stage
> flow with file:line citations and 46-block deprecated migration
> pattern at the bottom.

How `setAttributes({bgColor: '#f00'})` in the editor becomes a CSS rule
on the frontend.

## Pipeline overview (4 stages)

```
Stage 1: JS editor                   Stage 2: Block save
─────────────────                    ────────────────────
Control → setAttributes()      →     StyleComponent.js minifies
         ↓                           all style objects → saves
    block's style.js                  to `blockMeta` attribute
    generates CSS strings             in post content
    per device (desktop/
    tablet/mobile)

Stage 3: PHP post-save               Stage 4: Frontend enqueue
─────────────────────                 ────────────────────────
StyleHandler::write_css_from_content()  wp_enqueue_scripts hook
  → CSSParser::eb_block_style_recursive()  → enqueue_frontend_assets()
  → CSSParser::blocks_to_style_array()      checks for CSS file
  → CSSParser::build_css()                   → wp_enqueue_style(
  → file_put_contents()                         'eb-block-style-{postId}',
    to uploads/eb-style/                         uploads/eb-style/...css
    eb-style-{postId}.min.css                 )
```

## Stage 1: JS generates CSS strings

Each block has a `style.js` file (e.g., `src/blocks/post-grid/src/style.js`)
that produces CSS strings from attributes. The pattern:

```javascript
// In style.js — uses helpers from @essential-blocks/controls
import { dimensionHelpers, typoHelpers, backgroundHelpers } from "@essential-blocks/controls";

const cssDesktop = `
  .eb-post-grid-${blockId} {
    ${backgroundHelpers.generateBgCSS(attributes)}
    ${typoHelpers.generateTypoCSS(titleTypo)}
    column-count: ${columnNumberDesktop};
  }
`;
const cssTablet = `...tablet overrides...`;
const cssMobile = `...mobile overrides...`;
```

Key helpers (`src/controls/src/helpers/`):

| Helper file                   | Generates CSS for                                |
|-------------------------------|--------------------------------------------------|
| `backgroundHelpers.js`        | `background:`, gradients, overlay                |
| `borderShadowHelpers.js`      | `border-*`, `box-shadow`, hover states            |
| `dimensionHelpers.js`         | `padding`, `margin` (handles per-side)            |
| `responsiveRangeHelpers.js`   | Picks desktop/tablet/mobile value                 |
| `typoHelpers.js`              | `font-family`, `font-size`, `line-height`, etc.   |
| `shapeDividerHelpers.js`      | SVG shape divider positioning                     |

## Stage 2: StyleComponent saves to blockMeta

**`src/controls/src/helpers/StyleComponent.js:54-65`**

`StyleComponent` is a React component that:
1. Collects all CSS strings (desktop + tablet + mobile) from the block
2. Minifies them via `softMinifyCssStrings()`
3. Wraps tablet CSS in `@media (max-width: ${tabletBreakpoint}px)` and
   mobile in `@media (max-width: ${mobileBreakpoint}px)`
4. Calls `setAttributes({ blockMeta: combinedCSS })` to persist
   the CSS as a block attribute

**`blockMeta` is the bridge** — it's a string attribute stored in the block
comment in post content. This is how JS-generated CSS reaches PHP.

Breakpoints default to tablet=1024, mobile=767 but are configurable via
`eb_settings['responsiveBreakpoints']`.

## Stage 3: PHP persists to disk

**Entry point:** `StyleHandler::write_css_from_content()` at
`includes/Modules/StyleHandler.php:391`

```php
public function write_css_from_content( $parsed_content, $post_id = false, $type = '' )
```

**Call chain:**
1. `write_css_from_content()` receives WP's parsed block tree
2. `CSSParser::eb_block_style_recursive()` (`includes/Utils/CSSParser.php:48`)
   walks block tree, extracts `blockMeta`, `commonStyles`, `customCss`
3. `CSSParser::blocks_to_style_array()` (`:109`) converts to keyed array
4. `CSSParser::build_css()` (`:159`) generates final CSS with media queries
5. `write_block_css()` (`:442`) writes to disk

**CSS file locations:**
```
wp-content/uploads/eb-style/
├── eb-style-{postId}.min.css       # Regular posts/pages
├── full-site-editor/
│   └── eb-style-{templateId}.min.css  # FSE templates
└── reusable-blocks/
    └── eb-reusable-{blockId}.min.css  # Reusable blocks
```

**Triggers:**
- `on_save_post()` (`:551`) — runs on post save (may be commented out in some versions)
- `generate_post_content()` (`:600`) — lazy generation on frontend if CSS missing
- `after_save_widget()` (`:529`) — widget save

## Stage 4: Frontend enqueue

**`enqueue_frontend_assets()`** at `StyleHandler.php:100`, hooked to `wp_enqueue_scripts`

1. Gets current post ID
2. Checks if `{uploads}/eb-style/eb-style-{id}.min.css` exists (`:118`)
3. If exists: `wp_enqueue_style('eb-block-style-{id}', $url)` (`:119`)
4. If missing: triggers `generate_post_content()` to build it on-the-fly
5. Also enqueues fixed frontend styles via `load_frontend_css_file()` (`:260`)
   for each block type present on the page

**Dependency filter:** `eb_generated_css_frontend_deps` at `:103` — lets
you add CSS deps before the generated file.

## Theme / page-builder integration

`StyleHandler::__construct()` (`includes/Modules/StyleHandler.php:71-92`)
hooks into popular theme/page-builder constructs to enqueue per-post EB
CSS where the rendered post ID isn't the queried object.

| Theme / Plugin   | Hook / Mechanism                                                       | Type   | Line       |
|------------------|------------------------------------------------------------------------|--------|------------|
| GeneratePress    | `generate_element_post_id`                                              | filter | `:78`      |
| Astra Addon      | `wp_enqueue_scripts` → `astra_advanced_hooks_css_generation()` (pri 15) | action | `:92`      |
| Templately       | `templately_printed_location`                                           | action | `:89`      |
| Blocksy          | inline plugin-presence check + `ct_content_block` query (NOT a filter) | n/a    | `:158-172` |

**Note (verified):** Blocksy support is NOT a filter hook — it's an
inline check inside `enqueue_frontend_assets()` for the active plugin
`blocksy-companion-pro/blocksy-companion.php`, which then queries
`ct_content_block` posts directly and enqueues their EB CSS files.

When investigating "EB styles missing inside a GP hook / Astra header /
Blocksy content block / Templately template", grep `StyleHandler.php`
for the integration name first — the issue is almost always a missing
CSS file at `uploads/eb-style/eb-style-{integrationPostId}.min.css` or
the integration not surfacing the right post ID.

## Cache invalidation

| Event                                     | What happens                                         | Code location                            |
|------------------------------------------|------------------------------------------------------|------------------------------------------|
| Post saved in editor                      | CSS file regenerated from fresh `blockMeta`           | `on_save_post()` `:551`                  |
| Responsive breakpoints changed in admin   | All CSS files deleted (full rebuild on next load)     | `remove_frontend_assets()` `:660`        |
| `eb_after_save_responsiveBreakpoints_settings` | Triggers `remove_frontend_assets()`              | `:76-77`                                  |
| CSS file missing on frontend              | Lazy-generated from post content                      | `generate_post_content()` `:600`         |
| Widget saved                              | Widget CSS regenerated                                 | `after_save_widget()` `:529`             |

**"Stale CSS" debugging checklist:**
1. Edit the block and save the post (forces CSS regeneration)
2. Check `wp-content/uploads/eb-style/` for the file (right post ID?)
3. Check breakpoints match — if they changed, all CSS was nuked
4. Check browser cache (hard refresh)
5. Check caching plugin (Litespeed, WP Super Cache, etc.)
6. Check if `blockMeta` attribute is populated (read the post content)
7. Check if `write_block_css()` at `:442` errors (PHP error log)

## Block deprecated migrations & CSS

When a block is migrated via `deprecated` → `migrate()`, the `blockMeta`
attribute may need updating. If the old saved post has stale `blockMeta`,
the CSS file will have outdated rules. The fix is to re-save the post.

46 blocks have `deprecated` arrays (see `deprecated` section below).

## The `deprecated` migration system

When a block's `save()` output changes (e.g., new wrapper div, renamed CSS
class, new attribute schema), WordPress needs migration logic. EB uses the
standard Gutenberg `deprecated` array.

### Which blocks have migrations

46 blocks have `deprecated.js` files. Key ones:
- `advanced-heading`, `accordion`, `button`, `countdown`, `flipbox`
- `image-gallery`, `popup`, `slider`, `team-member`, `wrapper`
- Most content blocks have at least one deprecated version

### Pattern

```javascript
// src/blocks/<name>/src/deprecated.js
const deprecated = [
    {
        attributes: { ...oldAttributes },  // schema at that version
        save: ({ attributes }) => {
            // JSX matching the OLD save output
        },
    },
    // Optional: with migrate()
    {
        attributes: omit({ ...oldAttributes }, ["removedProp"]),
        migrate: (oldAttributes) => ({
            ...oldAttributes,
            newProp: "default",  // add new attrs with defaults
        }),
        save: ({ attributes }) => { /* old save */ },
    },
];

export default deprecated;
```

### Registration

In `src/blocks/<name>/src/index.js`:

```javascript
import deprecated from "./deprecated";

ebConditionalRegisterBlockType(metadata, {
    attributes, edit, save,
    deprecated,  // passed as property
});
```

### How migration triggers

1. User opens post in editor
2. WP parses saved block HTML from post content
3. Current `save()` is tried — if HTML matches, no migration needed
4. If mismatch: WP iterates `deprecated` array top→bottom
5. Each entry's `save()` is tried against saved HTML
6. First match wins — if `migrate()` exists, old attributes are transformed
7. Block re-renders with migrated attributes

### Common `migrate()` patterns

**Pattern A — Rename an attribute:**
```javascript
migrate: (oldAttributes) => ({
    ...oldAttributes,
    newAttrName: oldAttributes.oldAttrName,
})
```

**Pattern B — Items array → InnerBlocks (returns TUPLE):**
```javascript
migrate: (attrs) => [
    omit(attrs, ["items"]),   // new attributes
    attrs.items.map(item =>    // new innerBlocks
        createBlock("essential-blocks/<child>", {...}, [
            createBlock("core/paragraph", { content: item.text })
        ])
    ),
]
```
The tuple-return signature is required when migration also produces inner
blocks. `lodash.omit` is available globally as `lodash` (no import).

**Pattern C — `block.json` `supports` config changed:**
The deprecated entry needs its own `supports: { align, anchor }` matching
the OLD config.

### Gotchas

- **No `isEligible()` used** — EB relies on schema matching only.
- **Order matters** — newest deprecated version goes at INDEX 0. WP iterates
  the array top-to-bottom; first match wins. Never modify or reorder
  existing entries.
- **`blockMeta` not in deprecated schemas** — old blocks may lack it;
  migration doesn't regenerate CSS. User must re-save the post.
- **`omit()` from lodash** — used to exclude new props from old schemas.
- **Static blocks only** — dynamic blocks (`save: () => null`) don't need
  deprecated because their saved HTML is just the block comment with the
  attribute JSON.
- **Save wrapper migrated from `useBlockProps.save()` → `<BlockProps.Save attributes={attributes}>`**
  in newer code. Old deprecated entries may still use the legacy wrapper.

### Authoring → defer to plugin's skill

For procedure (snapshot attributes, write the entry, register, verify),
defer to the plugin's `deprecate-block` skill at
`<plugin-checkout>/.claude/skills/deprecate-block/SKILL.md`. This skill
documents the WHAT/WHY; that one documents the HOW.
