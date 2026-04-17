# Essential Blocks — Common Bug Patterns

> **TL;DR:** 10 recurring bug categories with symptoms, root causes,
> and fix patterns. Read at the START of any bug investigation —
> 2-minute scan can short-circuit hours of digging. Pairs with
> `investigation-playbook.md` Recipe 1.

Extracted from ~777 fix commits across 6 years.

## Top bug areas by frequency (from commit messages)

```
152  image (gallery, comparison, advanced-image, hotspot, avatar)
103  style/CSS (inline, responsive, hover, editor vs frontend)
 87  icon (picker, size, alignment, color)
 76  grid (post-grid, product-grid, layout)
 73  button (dual-button, CTA, link)
 67  slider (parallax, adaptive height, transition)
 62  form (validation, submission, fields, reCAPTCHA)
 61  CSS (specificity, missing, stale, responsive)
 59  accordion (toggle, nested, deprecation)
 53  frontend (render, script, interaction)
 49  color (control, gradient, picker, default)
 45  gallery (filterable, lightbox, masonry)
 42  height (adaptive, responsive, container)
 40  editor (crash, console error, save)
 33  responsive (breakpoint, tablet, mobile)
 32  toggle (content, switch, click)
 30  hover (state, transition, color)
 29  heading (line-height, separator, gradient)
 29  width (container, content, responsive)
```

## Recurring patterns

### 1. "CSS not applying on frontend" (~100+ instances)

**Symptoms:** Styling works in editor but not on frontend, or appears
after a hard refresh.

**Root causes (most common first):**
1. `blockMeta` attribute empty — StyleComponent didn't save
2. CSS file in `uploads/eb-style/` stale — cache not invalidated
3. Caching plugin (LiteSpeed, WP Super Cache) serving old CSS
4. Responsive breakpoint mismatch — admin changed breakpoints after blocks saved
5. Block not on the page anymore but CSS still targeting old blockId

**Fix pattern:** Re-save the post (triggers `write_css_from_content`).
If persistent, check `includes/Modules/StyleHandler.php:442` —
`write_block_css()` — for file-write errors.

**Where to look:**
- `includes/Modules/StyleHandler.php` — CSS file generation
- `includes/Utils/CSSParser.php` — attribute → CSS parsing
- `src/controls/src/helpers/StyleComponent.js` — JS-side CSS collection
- See `css-pipeline.md` for the full flow

### 2. "Block validation error — unexpected content" (~50+ instances)

**Symptoms:** "This block contains unexpected or invalid content" when
opening a previously saved post.

**Root causes:**
1. `save()` function output changed without adding a `deprecated` entry
2. Attribute added/removed without migration
3. CSS class name changed in `save()` JSX
4. HTML structure changed (new wrapper div, changed tag)

**Fix pattern:** Add a `deprecated` entry in `src/blocks/<name>/src/deprecated.js`
with the OLD `save()` and optionally `migrate()` to map old attrs → new.

**Where to look:**
- `src/blocks/<name>/src/save.js` — current save output
- `src/blocks/<name>/src/deprecated.js` — existing migrations
- `src/blocks/<name>/src/attributes.js` — current schema
- See `css-pipeline.md` "deprecated" section for the pattern

### 3. "Responsive value not persisting" (~33 instances)

**Symptoms:** Setting a value for tablet/mobile reverts to desktop on
reload, or mobile value shows desktop's value.

**Root causes:**
1. Attribute key mismatch — JS uses `columnNumberTablet` but attributes.js
   defines `columnTablet`
2. `useDeviceType()` returning wrong device
3. `ResponsiveRangeController` not wired to the right attribute prefix
4. Default values only set for desktop, not tablet/mobile

**Fix pattern:** Verify all three attribute keys exist in `attributes.js`:
```
<prefix>Desktop, <prefix>Tablet, <prefix>Mobile
```

**Where to look:**
- `src/blocks/<name>/src/attributes.js` — all three keys exist?
- `src/controls/src/helpers/responsiveRangeHelpers.js` — helper logic
- `src/controls/src/hooks/` — `useDeviceType` implementation

### 4. "Hover/transition broken" (~30 instances)

**Symptoms:** Hover effect doesn't trigger, or transition stutters.

**Root causes:**
1. Missing CSS `transition` property on the default state
2. Hover styles generated but placed outside the responsive media query
3. CSS specificity fight — theme CSS overrides hover
4. Missing hover attribute in `attributes.js` (e.g., `hvrBgColor` exists
   but not persisted)

**Fix pattern:** Check that `style.js` generates both default and
`:hover` selectors, and that transition is on the base selector.

### 5. "Icon size/color/alignment wrong" (~87 instances)

**Symptoms:** Icon renders at wrong size, color doesn't apply, or
alignment is off.

**Root causes:**
1. SVG `viewBox` vs `width/height` conflict
2. FontAwesome icon uses `<i>` tag; custom SVG uses `<svg>` — CSS targets
   one but not the other
3. Icon wrapper missing flex alignment CSS
4. Pro icon changes not reflected because controls submodule stale

**Fix pattern:** Check if the block uses `EBIconPicker` from controls.
Verify CSS targets both `<i>` and `<svg>` variants.

### 6. "Editor crashes / console errors" (~40 instances)

**Symptoms:** Gutenberg editor shows white screen, or console shows
React errors when block is selected.

**Root causes:**
1. Undefined attribute accessed without default (e.g., `attributes.queryData.postType` when queryData is undefined)
2. Missing `?.` optional chaining on nested objects
3. Control component receiving `undefined` instead of expected object
4. `@essential-blocks/controls` bundle stale (submodule not linked)

**Fix pattern:** Add nullish coalescing (`??`) or optional chaining (`?.`)
at the crash site. For stale controls, rebuild.

### 7. "Form submission fails" (~62 instances)

**Symptoms:** Form doesn't submit, shows error, or email never arrives.

**Root causes:**
1. Nonce expired (long page cache + form on cached page)
2. reCAPTCHA v3 token expired/missing (pro)
3. Email blocked by hosting provider's `wp_mail()` restrictions
4. Validation filter (`eb_form_data_validation`) returns false unexpectedly
5. Integration (Mailchimp, etc.) throws unhandled exception

**Where to look:**
- `includes/Integrations/Form.php` — free submission handler
- `includes/Utils/FormBlockHandler.php` (pro) — pro form handler
- `src/blocks/form/src/frontend.js` — client-side AJAX submit

### 8. "Deprecation / migration issue" (~20 instances)

**Symptoms:** Block shows "Attempt block recovery" button; or block
renders with old layout after update.

**Root causes:**
1. `deprecated` array missing for a `save()` change
2. `deprecated` entry's attributes schema doesn't match what's in post content
3. Multiple deprecated entries — wrong order (newest must be first)
4. `migrate()` returning wrong shape (missing a key)

**Fix pattern:** Read the post content (`wp_posts.post_content`) to see
the exact saved attributes + HTML. Compare against each `deprecated`
entry's `save()`. The first one whose HTML matches wins.

### 9. "Theme conflict" (~15 instances across recent releases)

**Symptoms:** EB block renders incorrectly only with specific theme.

**Known problematic themes:**
- **Blocksy** — Content Post Type issue with StyleHandler (fixed v6.0.7)
- **Kadence** — CSS specificity fights
- **Twenty Twenty-X** — Button color override

**Fix pattern:** Usually a CSS specificity issue. The fix is more specific
selectors in EB's CSS or a `!important` (last resort). Check
`StyleHandler.php` for theme-specific compatibility code.

### 10. "Dynamic block shows nothing" (~30 instances)

**Symptoms:** Block appears in editor but renders empty on frontend.

**Root causes:**
1. `render_callback()` returns empty string when query returns 0 results
2. Pro block renders but pro is inactive (`filter_pro_blocks_frontend` gate)
3. View template file missing (`views/<name>.php` not found)
4. Post/product query returns empty due to wrong query parameters

**Where to look:**
- `includes/Blocks/<Name>.php::render_callback()` — check return value
- `views/<name>.php` — template exists?
- `includes/Utils/QueryHelper.php` — query parameters correct?

## Bug diagnosis quick-start

```bash
EB="<free path>"

# Has this block had fixes recently?
git -C "$EB" log --since='3 months ago' --oneline -- src/blocks/<name>/ | grep -iE 'fix'

# What kinds of fixes?
git -C "$EB" log --since='6 months ago' --oneline --no-merges -- src/blocks/<name>/ includes/Blocks/<Name>.php

# Has this area been patched and re-patched (volatile)?
git -C "$EB" log --oneline -- src/blocks/<name>/src/edit.js | wc -l

# Was this bug fixed before (regression)?
git -C "$EB" log --all -S'buggy_function_or_value' --oneline
```

## Blocks most likely to have bugs (volatility × complexity)

Based on commit-count churn + number of fix-commits + JS file size:

| Block          | Churn | Fix freq | Why risky                                                  |
|----------------|-------|----------|------------------------------------------------------------|
| form           | 218   | high     | Complex: validation, integrations, pro override, 7+ fields |
| image-gallery  | 93    | high     | edit.js is 61-commit hotspot; filterable + lightbox (pro)   |
| slider         | 180   | high     | Frontend JS heavy; adaptive height; parallax variant        |
| post-grid      | 138   | medium   | Dynamic; complex query; AJAX pagination; pro extends        |
| accordion      | 170   | medium   | Nested child blocks; deprecation issues; toggle conflicts   |
| countdown      | 61    | medium   | Locale/i18n issues (`toLocaleString`); timer edge cases     |
| toggle-content | 76    | medium   | Switch click handler; frontend JS; 129+ edit-file commits  |
| lottie-animation | 38  | medium   | External JSON player; frontend.js hotspot (32 commits)     |
