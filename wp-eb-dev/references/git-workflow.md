# Essential Blocks — Git Workflow, Branching & Release Process

Conventions extracted from 7,992 commits on `origin/master`
(2020-04-05 → 2026-04-02). Use this when navigating PRs, matching issue
IDs to branches, or understanding who owns what.

## Branch naming

### Issue branches

The dominant pattern is `{5-digit-issue-id}-{short-description}`, e.g.:

- `80756-wpml-support`
- `80438-image-hotspot`
- `80674-editor-crash-issue`
- `79799-accordion-issue`
- `79679-adv-heading-issue`
- `79376-interactive-animation-issue`

The issue ID is a project-management ticket number (FBS/Startise PM system
at `https://projects.startise.com`). When a user mentions a 5-digit number
like `80756`, that's almost certainly an issue ID — map it to a branch with:

```bash
git -C "$EB" branch -a | grep "80756"
git -C "$EB" log --all --oneline | grep -iE "\b80756\b"
```

246+ unique issue branches have been merged into `master`.

### Developer branches

Individual contributors push work to namespaced branches that merge to
`latest` or directly to `master`:

- `jamil-dev` (23 merges)
- `monir-dev` / `monir-dev-v4` (32 total merges)
- `hanzala-dev` (15 merges)
- `sumaiya-dev-4.0` / `sumaiya-dev` (14 merges)

### Feature branches

Larger features get descriptive branches with no issue ID:

- `579-presets`
- `shape-divider`
- `nft-gallery`
- `form-block-jamil`
- `eb-pro-jamil`
- `improve-hanzala`

### Release branches

Every release has a `release-X.Y.Z` branch (e.g., `release-6.0.7`,
`release-5.8.1`). These are cut from `master`, get last-mile fixes, then
merged back via PR. `release-*` branches often show up in merge commit
messages.

### The `latest` branch

The `latest` branch is the most common merge source into `master`:

- 166 merges to master come `from WPDevelopers/latest`
- It appears to function as a staging/integration branch that accumulates
  feature branches before they land on `master`

When reviewing recent `master` history, most PRs look like:
`Merge pull request #NNN from WPDevelopers/latest`

## PR and merge conventions

- PR numbers are sequential (#507 as of v6.0.7, 2026-04-02)
- **439 unique PR numbers** referenced in master's commit history
- Merge commit format: `Merge pull request #NNN from WPDevelopers/<branch>`
- Release PRs: `Merge pull request #NNN from WPDevelopers/release-X.Y.Z`
- Integration PRs: `Merge pull request #NNN from WPDevelopers/latest`

## Commit message conventions

The project has **no strict conventional-commits discipline**, but patterns
emerge:

| Prefix / Form              | Frequency / Example                                        |
|----------------------------|------------------------------------------------------------|
| `Merge pull request #...`  | ~1,940 / Merges from branches to master                    |
| `Merge branch '...'`       | ~several hundred / Local merges                             |
| `update ...`               | ~1,316 / Most common non-merge verb ("update version", "update changelog", "update pot file") |
| `fix ...` / `fixed ...`    | ~777 / Bug fixes                                            |
| `Add ...` / `add ...`      | ~682 / New features/blocks                                  |
| `refactor: ...`            | Occasional, used for structural changes                     |
| `released version X.Y.Z`   | Release tag markers                                         |
| `prepared release version` | Pre-release bumps                                           |

Examples from recent master history:
```
refactor: Blocksy theme issue with EB stylehandler
fixed: Blocksy Content Post Type issue with stylehandler
added social share text support
Added no text block support
Added blocks supports
Fixed countdown loLocateString issue
updated: wpml config for post meta
fixed loop builder issue
fixed image comparison block
countdown block issue fixed
```

Lowercase/Title Case is inconsistent. Don't rely on case when grepping —
use `-i`.

## Release cadence

Releases per calendar year (from tag creation dates):

| Year | Releases | Notes                                                      |
|------|----------|------------------------------------------------------------|
| 2021 | 10       | Early days — block additions, pattern foundation           |
| 2022 | 29       | Accelerating feature work                                   |
| 2023 | 43       | Peak velocity — 253 commits in Jan 2023 alone              |
| 2024 | 47       | Sustained high cadence                                      |
| 2025 | 29       | Stabilization / quality focus                               |
| 2026 | 8        | Through April — 6.0.x series                                |

Recent releases (last 12 months):

```
2026-04-02  v6.0.7
2026-03-25  v6.0.6
2026-03-09  v6.0.5
2026-02-26  v6.0.4
2026-02-10  v6.0.3, v6.0.2 (same day)
2026-01-26  v6.0.1
2026-01-15  v5.9.2
2025-12-24  v5.9.1
2025-12-17  v5.9.0
2025-12-08  v5.8.2
2025-11-27  v5.8.1
2025-11-17  v5.8.0
```

Roughly one point release every 2-3 weeks. Patch releases can land
within hours of each other when hotfixes are needed.

## Release mechanics

1. Create `release-X.Y.Z` branch from `master`
2. Bump version in `essential-blocks.php` header and `Plugin.php::$version`
3. Update `readme.txt` "Stable tag" and changelog section
4. Regenerate `.pot` file (`npm run pot`)
5. Last-mile bug fixes merged into the release branch
6. `npm run release` runs validation + build
7. Merge release branch back to `master` via PR
8. Tag `vX.Y.Z` on master
9. Deploy to WordPress.org and distribute

The release scripts in `scripts/` automate steps 5-8. Grunt handles the zip
packaging for wp.org.

## Contributors

22 total contributors. Top 5 by commit count on `origin/master`:

| Rank | Name                                   | Commits | Notes                                     |
|------|----------------------------------------|---------|-------------------------------------------|
| 1    | Sumaiya Siddika                        | 1,990   | Lead front-end / block development        |
| 2    | Jamil Uddin (incl. "MD Jamil Uddin", "Jamil")  | 2,864   | Top overall — architecture + core         |
| 3    | Monir (incl. "Monir Hossain", "Md. Monir Hossain", "fencermonir") | 1,627   | Core + controls + admin |
| 4    | hz-tyfoon (Hanzala)                    | 774     | Early controls submodule founder           |
| 5    | Tasnim Alam                            | 471     | Earliest commits — block foundation        |

Additional contributors:
- Rahat Hossain (412)
- Priyo Mukul / PriyoMukul / priyomukul (~104)
- Rupok (13)
- haunzala (9)
- Sapan Mozammel (8)
- M Asif Rahman (3)

**Ownership by area** (derived from commit patterns):

- **Controls submodule:** Jamil Uddin (top) + Monir + hz-tyfoon. Hanzala
  founded it in June 2021 (first commit: `919bf36` "Initial commit" on
  2021-06-07).
- **Post blocks (grid, carousel):** Jamil Uddin + Sumaiya
- **Form block:** Jamil, significant refactoring by multiple devs
- **Admin dashboard:** Priyo Mukul (early), Monir (ongoing)
- **Early blocks (2020):** Tasnim Alam did most of the foundation work

## Activity hotspots (commit-count leaderboard)

Top 15 PHP/JS source file churn (excludes build artifacts, node_modules, dist):

| Commits | File                                                      |
|---------|-----------------------------------------------------------|
| 253     | `package.json`                                            |
| 232     | `includes/Plugin.php`                                     |
| 230     | `essential-blocks.php`                                    |
| 178     | `includes/Admin/Admin.php`                                |
| 146     | `includes/Core/Scripts.php`                               |
| 132     | `includes/blocks.php`                                     |
| 112     | `webpack.config.js`                                       |
| 84      | `includes/class-essential-blocks.php` (legacy)            |
| 69      | `includes/essential-admin.php` (legacy)                    |
| 63      | `includes/Utils/Helper.php`                               |
| 61      | `src/blocks/image-gallery/src/edit.js`                    |
| 57      | `src/blocks/image-gallery/src/inspector.js`               |
| 51      | `includes/Integrations/Form.php`                          |
| 44      | `includes/frontend-loader.php` (legacy)                    |
| 43      | `src/blocks/timeline/src/inspector.js`                    |
| 42      | `includes/Modules/StyleHandler.php`                       |
| 40      | `includes/Core/Block.php`                                 |

**What this tells you:**

- `Plugin.php`, `essential-blocks.php`, `Admin.php`, `Scripts.php`,
  `blocks.php` are the hottest stable files — every release touches them.
- `image-gallery`, `timeline`, `lottie-animation`, `slider`, `form`,
  `accordion` are the blocks under most active iteration.
- `Form.php` (integrations) has 51 commits — form block logic is stable
  but slowly accreting plugin integrations.
- `StyleHandler.php` is critical infrastructure — 42 commits spread across
  years indicates careful evolution, not churn.

## Peak activity months (top 10)

| Month       | Commits |
|-------------|---------|
| 2023-01     | 253     |
| 2023-10     | 249     |
| 2023-08     | 242     |
| 2021-09     | 239     |
| 2024-05     | 203     |
| 2021-12     | 197     |
| 2024-10     | 196     |
| 2023-12     | 192     |
| 2024-01     | 187     |
| 2023-11     | 182     |

First commit: `a7a461a3d` by Tasnim Alam, 2020-04-05 ("Update gitignore").
First block added: `fcff6d20b` 2020-04-06 ("Add button block for testing").

## How to find commits, PRs, and authors quickly

```bash
EB="<free path>"

# Who last touched this file?
git -C "$EB" log -1 --format='%h %an %ad %s' --date=short -- path/to/file.php

# Full blame for a specific line:
git -C "$EB" blame -L 100,120 -- includes/Plugin.php

# Find PR that introduced a change:
git -C "$EB" log --oneline --grep="80756"

# Get all commits on a branch vs master:
git -C "$EB" log origin/master..origin/80756-wpml-support --oneline

# List merged branches (who owns what):
git -C "$EB" log --all --oneline | grep -oE "from WPDevelopers/[^ ]+" | sort | uniq -c | sort -rn

# See what was in a release:
git -C "$EB" log v5.8.1..v5.9.0 --oneline

# Find the first commit that introduced a string:
git -C "$EB" log -S'eb_post_grid_query_results' --oneline --reverse
```

## Ticket / issue reference

The project-management system is at `https://projects.startise.com`. Issue
URLs look like `https://projects.startise.com/fbs-80756`. The 5-digit number
embedded in branch names is that ticket ID. If a user hands you an issue
URL, extract the number and search the repo for matching branches.

(The `wp-eb-test` skill has a full flow for fetching issue details with
credentials from `c.txt`. This skill intentionally stays code-focused —
ask the user to paste issue text if needed instead of fetching.)
