# Essential Blocks — Release History & Changelogs

Concrete per-release changes extracted from `git log` between tags.
Use this to answer "what changed in version X", "when was Y introduced",
"which release broke Z", and "what's the upgrade path from A to B".

Merge commits and version-bump noise are excluded — only substantive
code changes are listed.

## Free plugin releases (v5.9.0 → v6.0.7)

### v6.0.7 (2026-04-02, from v6.0.6)

- Blocksy theme compatibility: refactored StyleHandler for Blocksy
  Content Post Type issue
- Social share: added text support
- Text block: added "no text" block support
- Added `blocks supports` (WP core block supports)
- Prevented `undefined` in saved block output
- Fixed image hotspot position issue
- Controls submodule updated

### v6.0.6 (2026-03-25, from v6.0.5)

- Fixed countdown `toLocaleString` issue (was `loLocateString` typo)
- WPML: added config for post meta block and call-to-action block
- Fixed loop builder, image comparison, countdown, Google Map issues
- Fixed filterable gallery issue

### v6.0.5 (2026-03-09, from v6.0.4)

- Added image description support
- Added translate support for countdown block digit
- Updated WPML XML config
- Fixed caption style 2 issue
- Controls submodule updated

### v6.0.4 (2026-02-26, from v6.0.3)

**Larger release — 20 changes:**
- FontAwesome frontend style added to ToggleContent block
- Fixed toggle text, editor active/hover color issues
- Fixed TOC ID issue + added configurable TOC prefix option
- Added image alt text control, replace image fix
- Added liquid glass for rounded only
- Fixed lightbox issue, icon issue, border issue of marquee
- Fixed focus issue, editor crash
- Controls submodule updated twice

### v6.0.3 (2026-02-10, from v6.0.2)

- Changelog update only (hotfix wrapper for v6.0.2)

### v6.0.2 (2026-02-10, from v6.0.1)

- February 2026 admin campaign notice
- Fixed image removed issue
- Added text color to pricing table feature
- Fixed align issue of feature list
- Updated admin design

### v6.0.1 (2026-01-26, from v5.9.2)

- Fixed author role block issue (user role permissions)
- Removed GSAP from free plugin
- Added block default responsive popup design
- Fixed heading line height, rich text, image uploader placement issues
- Controls submodule updated

### v5.9.2 (2026-01-15, from v5.9.1)

- Fixed timeline style/preset issues (5 timeline commits)
- Fixed team member deprecation issue
- Fixed submenu responsive issue
- Fixed editor issue (general)
- Added tag support
- **Added Hook to Modify Generated CSS Before File Write** (key: `eb_fixed_frontend_styles/{$blockname}`)
- Fixed social share console.log (removed)
- Fixed call-to-action z-index, radius issues

### v5.9.1 (2025-12-24, from v5.9.0)

- Fixed quick setup issue
- Fixed interactive animation issue
- Fixed reset issue (controls)
- Fixed adaptive height on frontend (slider)
- Controls submodule updated 3 times

### v5.9.0 (2025-12-17, from v5.8.2)

- Added icon width/height controls
- Fixed tooltip alignment and number issue
- Fixed advanced image alignment
- Fixed admin error
- Updated WP version requirement
- Fixed container width selector, content width
- Fixed advanced video overlay, product image JS
- Controls submodule updated

## Common themes across releases

**Most frequently patched areas (by release mention count):**
- Image blocks (gallery, comparison, advanced-image, hotspot) — 8/10 releases
- Countdown — 4/10 releases
- CSS/style issues — every release
- Controls submodule — 8/10 releases (updated nearly every version)
- WPML compatibility — 2 releases (v6.0.5-6.0.6)
- Toggle/accordion — 3/10 releases

**Patterns:**
- Patch releases (v6.0.2, v6.0.3) tend to be 1-5 fixes
- Minor releases (v5.9.0, v6.0.1) are 10-20 changes
- Controls submodule updates happen on almost every release
- Theme compatibility fixes (Blocksy, Kadence) appear as point releases

## How to query release diffs yourself

```bash
EB="<free path>"

# What changed between two specific releases:
git -C "$EB" log v6.0.5..v6.0.7 --oneline --no-merges

# Full code diff for a release:
git -C "$EB" diff v6.0.6..v6.0.7 --stat

# Which blocks were touched in a release:
git -C "$EB" diff --name-only v6.0.6..v6.0.7 | grep 'src/blocks/'

# Which PHP files changed:
git -C "$EB" diff --name-only v6.0.6..v6.0.7 | grep '\.php$'

# What the pro did in the same period:
EB_PRO="<pro path>"
git -C "$EB_PRO" log v2.7.7..v2.7.8 --oneline --no-merges
```

## Pro version alignment (quick lookup)

| Free   | Pro      | Date       | Free highlights                            |
|--------|----------|------------|---------------------------------------------|
| v6.0.7 | 2.7.9    | 2026-04-02 | Blocksy compat, WPML, image hotspot        |
| v6.0.6 | v2.7.8   | 2026-03-25 | Countdown locale, loop builder fix          |
| v6.0.5 | v2.7.5   | 2026-03-09 | Image description, WPML XML                |
| v6.0.4 | v2.7.4   | 2026-02-26 | TOC prefix, liquid glass, lightbox          |
| v6.0.1 | v2.7.3   | 2026-01-26 | GSAP removal, role fix, responsive popup    |
| v5.9.2 | v2.7.2   | 2026-01-15 | Timeline presets, CSS hook, tag support     |
| v5.9.0 | v2.7.1   | 2025-12-17 | Icon sizing, container width, video overlay |
| v5.8.0 | v2.6.1   | 2025-11-17 | Shape divider presets, flex container       |
| v5.7.0 | v2.5.0   | 2025-09-21 | Feature block additions                     |
| v5.5.0 | v2.2.0   | 2025-06-17 | Major feature set                            |
