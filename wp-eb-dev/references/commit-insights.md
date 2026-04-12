# Essential Blocks — Commit History Insights

Data-driven view of 6 years of development. Use this to answer "is this
area stable or volatile?", "how has X evolved?", "who should I ask about
Y?", and "are we touching a hot file?".

Source: `git log origin/master` as of fc51997e8 (2026-04-02).

## Headline numbers

| Metric                                       | Value                              |
|-----------------------------------------------|------------------------------------|
| Total commits on master                       | 7,992                              |
| Total commits incl. merges                    | 8,388                              |
| Unique contributors                            | 22                                  |
| Version tags                                   | 166+                                |
| Timespan                                       | 2020-04-05 → 2026-04-02 (~6 years) |
| Submodule (controls) commits                   | 1,377 (on release-5.8.1)            |
| Peak commit month                              | 2023-01 (253 commits)               |
| Earliest commit                                | `a7a461a3d` "Update gitignore" by Tasnim Alam, 2020-04-05 |
| First block committed                          | `fcff6d20b` "Add button block for testing" by Tasnim Alam, 2020-04-06 |
| First call-to-action block                     | `b7190c2ae` "Add call to action block" by Tasnim Alam, 2020-04-06 |
| Latest tagged release                          | v6.0.7 (2026-04-02)                 |

## Commit type breakdown (approximate, from message-verb grep)

| Type              | Count    | Pattern grep                       |
|-------------------|----------|------------------------------------|
| Merges            | ~1,940   | `^merge`                           |
| Updates           | ~1,316   | `update|upgrade`                   |
| Fixes             | ~777     | `fix|bug|resolve`                  |
| Adds / features   | ~682     | `add|feat|new`                     |
| Release markers   | ~859     | `release|version|v\d`              |

(Categories overlap — a commit titled "Add fix for X" counts in both.)

## Monthly activity heatmap (commits per month)

Peak periods, top 10:

```
2023-01  ████████████████████████████  253
2023-10  ███████████████████████████   249
2023-08  ██████████████████████████    242
2021-09  █████████████████████████     239
2024-05  ██████████████████████        203
2021-12  █████████████████████         197
2024-10  █████████████████████         196
2023-12  ████████████████████          192
2024-01  ████████████████████          187
2023-11  ███████████████████           182
```

**Story the data tells:**

- **2020-04 → 2020-12**: Foundation phase. Sparse commits (8-98/month),
  block skeletons added by Tasnim Alam.
- **2021**: Team expands, controls submodule created (June 2021), feature
  blocks added rapidly. Sep+Dec peaks.
- **2022**: Sustained ~150-180 commits/month, heavy feature work.
- **2023**: Peak year. Three months above 200 commits. This is when the
  current architecture (blocks/, controls/, dashboard) solidified.
- **2024**: Continued velocity — new modules (AI integrations, Templately).
- **2025**: Gradual slowdown into stabilization. Quality-of-life fixes,
  tech debt.
- **2026 Q1**: 6.0.x series — measured pace, ~50-100 commits/month.

## Top contributors

(Combining author-name variants for people whose commits appear under
multiple spellings. See `git-workflow.md` for the breakdown.)

| Rank | Contributor                                              | Commits ~ | Specialty                            |
|------|----------------------------------------------------------|-----------|--------------------------------------|
| 1    | Jamil Uddin ("MD Jamil Uddin" + "Jamil Uddin" + "Jamil") | ~2,864    | Architecture, post blocks, admin     |
| 2    | Sumaiya Siddika                                          | 1,990     | Frontend blocks, editor UIs          |
| 3    | Monir (4 name variants)                                   | ~1,627    | Controls, admin, integrations        |
| 4    | hz-tyfoon / haunzala                                      | ~783      | Controls submodule founder           |
| 5    | Tasnim Alam                                               | ~471      | Foundation blocks (2020)             |
| 6    | Rahat Hossain                                             | 412       | Various                               |
| 7    | Priyo Mukul (3 spellings)                                 | ~104      | Admin notices, insights              |
| 8    | Rupok                                                     | 13        | Occasional                            |
| 9    | Sapan Mozammel                                            | 8         | Occasional                            |
| 10   | M Asif Rahman                                             | 3         | Occasional                            |

Top 4 account for ~85% of all commits. Blame/ownership is concentrated.

## Commits per block (from commit message mentions)

Most-mentioned blocks in commit messages (rough ranking):

```
form              ████████████████████████  218
button            ████████████████████████  217
slider            ████████████████████      180
accordion         ███████████████████       170
post-grid         ███████████████           138
infobox           ████████████              106
image-gallery     ██████████                 93
team-member       █████████                  84
toggle-content    ████████                   76
social            ████████                   71
flipbox           ███████                    64
post-carousel     ███████                    61
countdown         ███████                    61
testimonial       ██████                     58
advanced-heading  ██████                     55
pricing-table     █████                      50
feature-list      ████                       39
woo-product-grid  ████                       37
number-counter    ████                       32
```

This isn't authoritative (commit messages don't always mention the block
by name), but it's a reasonable "where does the team spend time" signal.

## File hotspots (top 30, source code only)

Excludes `dist/`, `node_modules/`, legacy vendor bundles:

```
 253  package.json
 232  includes/Plugin.php
 230  essential-blocks.php
 178  includes/Admin/Admin.php
 146  includes/Core/Scripts.php
 132  includes/blocks.php
 112  webpack.config.js
  84  includes/class-essential-blocks.php  (legacy, pre-refactor)
  69  includes/essential-admin.php          (legacy)
  63  includes/Utils/Helper.php
  61  src/blocks/image-gallery/src/edit.js
  57  src/blocks/image-gallery/src/inspector.js
  51  includes/Integrations/Form.php
  44  includes/frontend-loader.php          (legacy)
  43  src/blocks/timeline/src/inspector.js
  42  includes/Modules/StyleHandler.php
  41  src/blocks/timeline/src/style.scss
  40  includes/class-essential-blocks-enqueues.php  (legacy)
  40  includes/Core/Block.php
  38  src/blocks/lottie-animation/src/edit.js
  37  src/blocks/image-gallery/src/style.scss
  37  includes/Core/Maintenance.php
  36  src/blocks/image-gallery/src/save.js
  36  src/blocks/timeline/src/style.js
  34  src/blocks/slider/src/inspector.js
  33  src/blocks/form/src/edit.js
  32  src/blocks/lottie-animation/src/frontend.js
  31  src/blocks/lottie-animation/src/inspector.js
  31  src/blocks/accordion/src/edit.js
  31  includes/API/Product.php
```

### Reading the hotspots

**"Legacy" files** (marked above) — `class-essential-blocks.php`,
`essential-admin.php`, `class-essential-blocks-enqueues.php`,
`frontend-loader.php` — pre-date the current `EssentialBlocks\` namespace
and PSR-4 layout. They may have been moved/split into `includes/Core/` and
`includes/Admin/`. If git log points to them for old changes, use
`git log --follow` to track the rename.

**Infrastructure churn**: `Plugin.php` + `essential-blocks.php` +
`package.json` + `webpack.config.js` are touched on almost every release
(version bumps, constant updates, build config). High commit count ≠ high
defect risk — it's routine plumbing.

**Real volatility**: `Admin.php`, `Scripts.php`, `blocks.php`, `Form.php`,
`StyleHandler.php`, and the `image-gallery` + `timeline` + `lottie` +
`slider` + `form` + `accordion` block edit/inspector files. These are the
places where behavior actually changes. Read their recent history before
making non-trivial modifications.

## Evolution timeline

| Year    | Milestone / Phase                                                                   |
|---------|--------------------------------------------------------------------------------------|
| 2020    | Foundation. Tasnim Alam scaffolds ~10 blocks. No controls submodule yet.             |
| 2021 Q2 | Controls submodule extracted (Hanzala's `Initial commit`, 2021-06-07).              |
| 2021 Q3-Q4 | Team expands. ~240 commits/month. Early feature blocks stabilize.                  |
| 2022    | Migration to `EssentialBlocks\` namespace + PSR-4 autoload. Legacy files deprecated.  |
| 2022-2023 | Admin dashboard React app built out (`src/admin/dashboard/`).                       |
| 2023    | Peak velocity. Major additions: patterns, templates, advanced controls, responsive system. |
| 2023-2024 | AI modules introduced (`src/modules/write-with-ai/`, `ai-for-woo/`).               |
| 2024    | WooCommerce block suite expanded. Global styles system matures.                      |
| 2024-2025 | Form block heavy iteration — integrations, validation hooks, child field blocks.    |
| 2025    | 5.x releases focus on stability and theme compatibility (Blocksy, Kadence, etc.).  |
| 2026    | 6.0.x series — liquid glass effects, image hotspot, WPML support.                   |

Recent release themes (from commit messages in release branches):

- **v6.0.6-6.0.7** (2026-03 to 2026-04): Blocksy theme compatibility, WPML
  config for call-to-action + post meta, countdown locale, editor crash fix,
  image hotspot
- **v6.0.0-6.0.5**: Liquid glass effects, pricing table fixes, accordion
  issues, interactive animation, advanced video
- **v5.8.x-5.9.x**: Theme tests, shape divider presets, flex container

## Commit message quirks to know

- **Typos in commit bodies are common** — "loLocateString" (should be
  `toLocaleString`) slipped into a recent title. Don't assume commit
  subjects reflect final identifiers.
- **Mix of passive and active voice** — "fixed", "fix", "Fixed", "fixing" all
  appear. Grep case-insensitive.
- **No conventional commits** — don't try to parse `feat:` / `fix:` /
  `refactor:` as authoritative.
- **Revert patterns** — look for `Revert "Merge branch '...'"` to find work
  that was rolled back. Example: `Revert "Merge branch '80438-image-hotspot'
  into release-6.0.7"` — the feature was backed out mid-release.
- **Descriptive branch merges** — Merge commits often preserve the branch
  name, which is the best link back to the originating issue.

## How to query history for investigation

```bash
EB="<free path>"

# Commits touching a file, newest first:
git -C "$EB" log --oneline -- includes/Modules/StyleHandler.php | head -20

# Everything in the last year touching a block:
git -C "$EB" log --since='1 year ago' --oneline -- src/blocks/post-grid/

# What changed between two releases:
git -C "$EB" log v5.9.0..v6.0.0 --oneline | wc -l
git -C "$EB" log v5.9.0..v6.0.0 --format='%h %s' -- src/blocks/

# Who has touched a file the most:
git -C "$EB" log --format='%an' -- includes/Plugin.php | sort | uniq -c | sort -rn

# Find when a string first appeared in the codebase:
git -C "$EB" log -S'LiquidGlassEffectControl' --reverse --oneline

# Find when a string was deleted:
git -C "$EB" log -S'function old_name_here' --diff-filter=D --oneline

# Show the full diff of a specific commit:
git -C "$EB" show <hash>

# Search commit messages:
git -C "$EB" log --grep='80756' --oneline
git -C "$EB" log --grep='wpml' --oneline -i
```

## What this history does NOT tell you

- **Code quality trajectory** — commit count is churn, not quality.
- **Bug density** — a "fix" commit may close 1 bug or 10.
- **Test coverage** — there's minimal test history.
- **Customer-facing feature releases** — the commit log mixes refactors
  with user-visible features; check `readme.txt` changelog sections for
  what users see.
- **Pro plugin activity** — this is the free repo only. The pro plugin has
  its own git history.

## Quick answers to common "how old / how stable" questions

| Question                                                    | Answer                                              |
|-------------------------------------------------------------|-----------------------------------------------------|
| When did the controls submodule start?                      | 2021-06-07 (`919bf36`, Hanzala)                     |
| When did the `EssentialBlocks\` namespace arrive?            | ~2022 refactor (legacy `class-*.php` files phased out) |
| When did the first form block land?                         | Check `git log -- includes/Blocks/Form.php --reverse` |
| When did patterns get added?                                 | `git log -- includes/Core/BlocksPatterns.php --reverse` |
| Is `StyleHandler.php` a new module or old?                   | 42 commits over multiple years — steadily evolved    |
| Has post-grid ever been rewritten?                           | No full rewrite; continuously iterated (138 commits) |
