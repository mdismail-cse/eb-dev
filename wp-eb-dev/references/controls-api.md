# Essential Blocks Controls — Reference

The `src/controls/` submodule is the shared React control library used by
both free and pro Essential Blocks. Blocks import from it via the alias
`@essential-blocks/controls`, which webpack externalizes to `window.EBControls`
at build time. The bundle is compiled to `dist/admin/controls/controls.js`.

- Submodule repo: separate from the main free plugin (its own git history —
  ~1,377 commits as of v5.8.1 / release-5.8.1 branch)
- Location relative to free: `src/controls/`
- Entry: `src/controls/src/index.js`
- Build: `src/controls/webpack.config.js`

## Top-level layout (`src/controls/src/`)

| Dir                  | Purpose                                                                    |
|----------------------|----------------------------------------------------------------------------|
| `controls/`          | 37 individual control components (leaf React components)                    |
| `group-controls/`    | Grouped/preset controls — `advanced-controls`, `layout-options`             |
| `extensions/`        | Plugin-style extension system (animation effects, dynamic field sources)    |
| `hoc/`               | Higher-order components — `withBlockContext`, AI wrappers                   |
| `hooks/`             | React hooks — `useBlockDefaults`, device type, attribute helpers            |
| `components/`        | Shared UI primitives (buttons, dropdowns, toggles, pickers)                 |
| `helpers/`           | CSS-generation helpers for attribute → inline style conversion              |
| `utils/`             | Small functional utilities                                                   |
| `icons/`             | Icon assets used throughout controls                                         |
| `shape-divider-svg/` | SVG presets for shape-divider control                                        |
| `extras/`            | Miscellaneous shared widgets                                                  |
| `index.js`           | Barrel export — everything blocks import goes through here                  |
| `backend.scss`       | Editor-only stylesheet for controls                                          |

## The 37 individual controls

Directory names (kebab-case) in `src/controls/src/controls/`. Most are
default-exported React components with prop shapes documented in each file's
top comment.

| Directory                              | Exports (typical)                     | Purpose                                            |
|----------------------------------------|----------------------------------------|----------------------------------------------------|
| `advanced-query-control`               | `AdvancedQueryControl`                | Rich query builder for post blocks                 |
| `animation-control`                    | `AnimationControls`                   | Entrance animations (selector, duration, delay)    |
| `background-control`                   | `BackgroundControl`                   | Color / gradient / image background picker         |
| `border-shadow-control`                | `BorderShadowControl`                 | Border style + shadow (hover states too)           |
| `button-group-control`                 | `ButtonGroupControl`                  | Toggle group (horizontal button row)               |
| `color-control`                        | `ColorControl`                        | Color picker with alpha + palette                  |
| `custom-query`                         | `CustomQuery`                         | Simpler query builder variant                      |
| `dimensions-control`                   | `DimensionsControl`                   | Padding/margin with linked/individual sides        |
| `dimensions-control-v2`                | `DimensionsControlV2`                 | Newer dimensions control rewrite                   |
| `dynamic-field`                        | `DynamicInputControl`, `DynamicFormFieldControl` | Text with dynamic tag support (post meta, etc.) |
| `flex-container`                       | `FlexContainerControl`                | Flexbox layout controls                            |
| `gradient-color-controller`            | `GradientColorControl`                | Linear/radial gradient builder                     |
| `icon-picker`                          | `EBIconPicker`                        | FontAwesome + custom SVG icon picker               |
| `image-avatar`                         | `ImageAvatar`                         | Image uploader (circular avatar preview)           |
| `liquid-glass-effect-control`          | `LiquidGlassEffectControl`            | Glassmorphism effect picker (newer, v5.8+)         |
| `pagination`                           | `Pagination`                          | Pagination styling controls                        |
| `popover-toggle`                       | `PopoverToggle`                       | Floating popover open/close UI                     |
| `pro-select-control`                   | `ProSelectControl`                    | Select with pro-only options marked                |
| `range-control`                        | `RangeControl`                        | Numeric slider input                               |
| `reset-control`                        | `ResetControl`                        | Reset-to-default button                            |
| `responsive-align-control`             | `ResponsiveAlignControl`              | Text/block alignment with responsive variants      |
| `responsive-range-control`             | `ResponsiveRangeController`           | Range with desktop/tablet/mobile values            |
| `responsive-select-control`            | `ResponsiveSelectController`          | Select dropdown with responsive variants           |
| `responsive-text-control`              | `ResponsiveTextController`            | Text input with responsive variants                |
| `shape-divider`                        | `ShapeDividerControl`                 | SVG shape divider picker                           |
| `sort-control`                         | `SortControl`                         | Sort option picker                                 |
| `sortable-control`                     | `SortableControl`                     | Drag-to-reorder list                               |
| `text-control`                         | `TextControl`                         | Plain text input (EB-styled)                       |
| `text-control-with-dropdown`           | `TextControlWithDropdown`             | Text + attached dropdown                           |
| `textarea-control`                     | `TextareaControl`                     | Multi-line text                                    |
| `toggle-button`                        | `ToggleButton`                        | On/off toggle switch                               |
| `typography-control`                   | `TypographyDropdown`                  | Font family, size, weight, line-height, etc.       |
| `typography-control-v2`                | `TypographyDropdownV2`                | Newer typography rewrite                            |
| `unit-control`                         | `UnitControl`                         | Input with unit selector (px/em/%/vw/...)          |
| `update-category-icon`                 | (utility)                             | Category icon updater                               |
| `withResButtons`                       | HOC                                   | Adds responsive device toggle buttons              |
| `woocommerce-query`                    | `WoocommerceQuery`                    | Query builder for products                          |

To see what each one actually exports, read the directory's `index.js`. The
pattern is always `export { default as <Name> } from './<component>'` or
default-export.

## Group controls

`src/controls/src/group-controls/`

- `components/advanced-controls.js` — `AdvancedControls` wrapper that
  nests multiple leaf controls under one collapsible panel
- `components/layout-options.js` — layout mode selector (grid / list / masonry / etc.)

Blocks use `AdvancedControls` to organize the inspector sidebar into sections
instead of long flat lists.

## HOCs (`src/controls/src/hoc/`)

| HOC                           | File                                      | Purpose                                                  |
|-------------------------------|-------------------------------------------|----------------------------------------------------------|
| `withBlockContext`            | `hoc/withBlockContext.js`                 | Provides current block context to wrapped components     |
| `withAiForImage`              | `hoc/ai-for-image/`                       | Wraps components with AI image generation capability     |
| `withAiForRewrite`            | `hoc/ai-for-rewrite/`                     | Wraps text components with AI rewrite                    |
| `withAiForRichtext`           | `hoc/ai-for-richtext/`                    | Wraps rich text components with AI generation            |

## Hooks (`src/controls/src/hooks/`)

| Hook                 | Purpose                                                                            |
|----------------------|------------------------------------------------------------------------------------|
| `useBlockDefaults`   | Returns the default attributes for the current block                               |
| `useDeviceType`      | Returns the currently selected responsive device (mobile/tablet/desktop)           |
| `useContextSelector` | Optimized context subscription (avoid re-renders on unrelated state changes)        |
| `attributes`         | Helper hooks for attribute read/write                                               |

## Helpers (`src/controls/src/helpers/`)

These are the CSS-generation functions. Every block inspector eventually
pipes attribute values through one of these to produce an inline style string.

| File                              | What it does                                                    |
|-----------------------------------|------------------------------------------------------------------|
| `backgroundHelpers.js`            | Normalize background attributes → `background:` CSS             |
| `borderShadowHelpers.js`          | Border + shadow → `border-*`, `box-shadow` CSS                  |
| `dimensionHelpers.js`             | Padding/margin → `padding: Npx Npx Npx Npx` (handles sides)     |
| `responsiveRangeHelpers.js`       | Pick right device value based on current breakpoint              |
| `typoHelpers.js`                  | Typography → `font-family`, `font-size`, `line-height`, etc.    |
| `shapeDividerHelpers.js`          | SVG shape divider CSS generation                                 |
| `AdvancedColorPicker.js`          | Color picker component (re-exported from helpers historically)  |
| `StyleComponent.js`               | CSS generation class wrapper                                     |
| `apiFetch.js`                     | Fetch wrapper for REST API calls                                 |
| `handlingPreviewBtnsHelpers.js`   | Preview device toggle handlers                                   |
| `hasVal.js`                       | "Is this value actually set?" check (handles 0, false, '')      |

## `ebConditionalRegisterBlockType`

This is the most important export of the whole submodule from a block
author's perspective. Almost every `src/blocks/<name>/src/index.js` uses it:

```javascript
import { ebConditionalRegisterBlockType } from "@essential-blocks/controls";

ebConditionalRegisterBlockType(metadata, {
    keywords,
    icon,
    attributes,
    edit,
    save,
});
```

It handles:
- Free/pro visibility gating (checks `ESSENTIAL_BLOCKS_IS_PRO_ACTIVE`)
- Category registration (creates `'essential-blocks'` inserter category if missing)
- Merge with `eb_settings.blocks` enable/disable state from admin dashboard
- Shim for `block.json` metadata on older Gutenberg versions

If a block isn't showing up in the editor, first suspect this function and
the `eb_settings['blocks'][slug]` visibility flag.

## Building the submodule

The free plugin's `package.json` has `prestart` and `prebuild` scripts that
run `npm link` / `pnpm link` to wire the submodule into `node_modules/` at
the alias `@essential-blocks/controls`. You don't usually run the submodule
build independently — `npm run build` in the free plugin root handles
everything, and the prebuild step ensures the link is fresh.

If the submodule dist is stale or missing, the editor will import undefined
and blocks will fail to register. Rebuilding from the free root typically
resolves this.

## Responsive value pattern

The responsive controls (`ResponsiveRangeController`, `ResponsiveDimensionsControl`,
`ResponsiveSelectController`, `ResponsiveTextController`) all store values as
**flat sibling attributes** with device suffixes, not as nested objects:

```javascript
// attributes.js
{
    columnNumberDesktop: { type: "number", default: 3 },
    columnNumberTablet: { type: "number", default: 2 },
    columnNumberMobile: { type: "number", default: 1 },
}
```

The helper `responsiveRangeHelpers.js` picks the right key based on the
current device type from `useDeviceType()`. When refactoring, preserve these
flat keys — changing to nested objects breaks old saved posts.

## Adding a new control — checklist

1. Create `src/controls/src/controls/<kebab-name>/` with `index.js` + component
2. Export from `src/controls/src/controls/<kebab-name>/index.js`
3. Add re-export to `src/controls/src/index.js`
4. If it has CSS, add SCSS next to the component and import in `backend.scss`
5. Rebuild (`npm run build` in free root — prebuild relinks submodule)
6. Import in a block's `edit.js` as `import { NewControl } from "@essential-blocks/controls"`
7. Update the submodule's own git history with a commit

Because the submodule has its own git history, the **submodule pointer**
inside the parent free repo must be committed separately after submodule
changes are merged. Always double-check `git diff -- src/controls` in the
parent when reviewing controls-related PRs.
