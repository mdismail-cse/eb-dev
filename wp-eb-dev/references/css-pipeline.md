# Essential Blocks — CSS Pipeline & StyleHandler

How `setAttributes({bgColor: '#f00'})` in the editor becomes a CSS rule
on the frontend. This is the single most important flow for debugging
"CSS not applying", "styles missing on frontend", or "stale styles".

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

### Gotchas

- **No `isEligible()` used** — EB relies on schema matching only
- **`blockMeta` not in deprecated schemas** — old blocks may lack it;
  migration doesn't regenerate CSS (user must re-save)
- **`omit()` from lodash** — used to exclude new props from old schemas
- **Order matters** — newest deprecated version first
- **Static blocks only** — dynamic blocks (`save: () => null`) don't need
  deprecated because their saved HTML is just the block comment
