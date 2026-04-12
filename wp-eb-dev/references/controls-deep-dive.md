# Essential Blocks Controls — Deep Dive (Standalone Repo)

The controls library is **its own standalone git repository**, hosted at
`git@github.com:EssentialBlocks/controls.git` (note the `EssentialBlocks`
org, not `WPDevelopers`), and mounted as a git submodule inside
`essential-blocks/src/controls/`.

This file complements `controls-api.md` (which covers the API surface).
Here, we cover the repo itself: history, branching, how it releases with
free, and how to navigate it.

If you're looking for the list of 37 controls or how to add one, read
`controls-api.md` first. If you're here to understand "why is the
submodule pointer out of sync", "who owns this control", or "when was X
introduced", this is your file.

## 1. Quick facts (as of origin/master 4ab3a6033, 2026-04-02)

| Metric                                    | Value                                             |
|--------------------------------------------|---------------------------------------------------|
| Total commits on `master`                  | 1,427                                             |
| Total commits on release-5.8.1 (submodule pointer) | 1,377                                     |
| Timespan                                    | 2021-06-07 → 2026-04-02 (~5 years)                |
| Contributors                                 | 15                                                 |
| Default branch                               | `master` (not `main`)                             |
| Release tags                                 | 0 — releases tracked via branches, not tags        |
| Merge PRs                                    | ~183 pull requests                                 |
| First commit                                  | `919bf36` "Initial commit" by haunzala (Hanzala), 2021-06-07 |
| Latest merge                                  | `#200 from EssentialBlocks/release-6.0.7`         |

## 2. Repository origin story

The controls submodule was **extracted from the free plugin** in June 2021.
Commit `39e9800` by hz-tyfoon reads:

> "all utilities brought here as they were in this 'dc6a1ae12bec81869a2c414'
> commit of hanzala-dev branch"

That's the smoking gun — Hanzala (hz-tyfoon / haunzala) was working on a
dev branch inside free, then the work was lifted out into its own repo to
allow free and pro to share controls. The early commits in June 2021 are
all Hanzala bringing existing work into the new repo.

## 3. Contributors (top 15)

| Rank | Contributor                                                            | Commits |
|------|------------------------------------------------------------------------|---------|
| 1    | Jamil Uddin (incl. "MD Jamil Uddin" + "Jamil")                         | 620     |
| 2    | Monir (incl. "Monir Hossain", "Md. Monir Hossain", "fencer monir", "fencermonir") | 374     |
| 3    | hz-tyfoon (incl. "haunzala")                                            | 191     |
| 4    | Sumaiya Siddika                                                         | 121     |
| 5    | Rahat Hossain                                                           | 99      |
| 6    | Priyo Mukul (incl. "PriyoMukul")                                         | 20      |
| 7    | Fuad Ragib                                                              | 2       |

Ownership is more concentrated than the free plugin — top 3 account for
~83% of commits. Hanzala is the founder but stepped back; Jamil + Monir
are the steady-state maintainers.

## 4. Monthly activity — peak periods

```
2024-05  ████████████████████  79
2025-08  ██████████████        58
2025-06  ██████████████        55
2021-06  ██████████████        55   (founding month)
2021-11  ████████████          51
2024-10  ████████████          47
2021-07  ████████████          46
2021-10  ███████████           43
2023-05  ███████████           41
2024-01  ███████████           40
```

Two main bursts:
- **2021 Q3-Q4** — Founding. ~180 commits across June–November as the
  control library was fleshed out.
- **2024 Q2-Q4** — Major expansion. Typography v2, dimensions v2,
  advanced query controls, liquid glass effect, AI HOCs added here.

## 5. Branch naming

Controls uses the same branch conventions as free, but with its own
internal numbering:

- **4-digit branch names** like `1006-dynamic-tags-text-fields`,
  `1041-global-controls`, `1074-security-issue`,
  `1100-advanced-heading-issue`, `1102-embed-press-issue`,
  `1105-dnd-kit`, `1110-reduce-google-font-load`. These are internal
  control-repo tickets.
- **5-digit branch names** like `17949-post-grid-featured-post`,
  `18077-global-controls-phase-2`, `80244-sortable-css` — these cross
  into the shared free issue tracker.
- **Release branches**: `release-4.2.0`, `release-5.0.10`, `release-6.0.7`,
  etc. — mirror the free plugin's release versions.
- **Dev branches**: `sumaiya-dev`, `jamil-dev`, `monir-dev`, etc.

When the free plugin is on release-6.0.7, the controls submodule is on its
own `release-6.0.7` branch — they're kept in lockstep.

## 6. How controls releases

**Controls has no tags.** Releases are tracked only via branches. The flow:

1. Feature branches (1005-*, 80XXX-*) merge to a `release-X.Y.Z` branch
2. That branch is tested end-to-end in the free plugin
3. A PR merges `release-X.Y.Z` to `master` in the controls repo
4. In parallel, the free plugin's `release-X.Y.Z` branch updates the
   submodule pointer via `git submodule update --remote` or a manual
   pointer bump
5. Free's release PR includes the new submodule SHA
6. When free cuts its tag (e.g., `v6.0.7`), the controls pointer is frozen
   at that SHA

**Implication for investigation:** The controls commit currently active
in your checkout is determined by the free plugin's branch, NOT by
"current controls master". If you're on free `release-5.8.1`, the
controls submodule is pinned at the SHA that was in master when 5.8.1
was cut — which may be behind controls' current master by dozens of
commits.

To see what's ahead:

```bash
EB="<free path>"
EBC="$EB/src/controls"

# Submodule pointer from free:
git -C "$EB" submodule status
# → 1fed41a7c0... src/controls (heads/release-5.8.1)

# What's on controls master that's NOT in that pointer?
git -C "$EBC" fetch origin
git -C "$EBC" log 1fed41a7c0..origin/master --oneline
```

## 7. Top file hotspots

Top 20 most-committed files on controls master:

```
 116  src/backend.scss                          # Main stylesheet
  95  src/index.js                              # Barrel export
  75  src/group-controls/index.js               # Group controls entry
  62  src/helpers/index.js                      # Helpers barrel
  48  backend.scss                              # (older path, renamed)
  41  src/group-controls/components/advanced-controls.js
  40  src/components/Image/general.js
  39  package.json
  38  src/controls/custom-query/index.js        # Hot query builder
  32  src/controls/color-control/index.js
  28  src/components/template-browse/index.js
  26  src/controls/custom-query/apiData.js
  24  src/components/Image/index.js
  24  src/components/BlockComponents/EBBlockProps.js
  23  src/controls/woocommerce-query/index.js
  23  src/controls/icon-picker/index.js
  23  src/components/AI/AIImagePopover.js       # AI HOC
  21  (older) background-control/index.js
  20  src/controls/border-shadow-control/index.js
  20  (older) helpers/index.js
  19  src/components/Image/attributes.js
  19  src/components/Button/index.js
```

**Reading this:**

- `backend.scss` and `index.js` at the top is routine churn (global
  style/export updates on nearly every PR).
- Real-feature churn is in `custom-query/`, `color-control/`,
  `woocommerce-query/`, `icon-picker/`, `border-shadow-control/` — these
  are the most-iterated individual controls.
- `src/components/Image/` and `src/components/Button/` appear multiple
  times — these are shared components used inside controls.
- `AIImagePopover.js` and `template-browse/` point to the AI / templately
  integration work that landed mid-2024.

## 8. Path layout reminder

Inside the controls repo root (`src/controls/` from the free plugin):

```
.
├── LICENSE
├── package.json
├── pnpm-lock.yaml
├── webpack.config.js
├── backend.scss                    # legacy root stylesheet (some older commits)
├── src/
│   ├── index.js                    # Main barrel export
│   ├── backend.scss                # Active root stylesheet
│   ├── controls/                   # 37 individual control dirs
│   ├── group-controls/             # Group/preset controls
│   ├── components/                 # Shared UI (Image, Button, AI, template-browse, BlockComponents)
│   ├── extensions/                 # Plugin-style extensions
│   ├── helpers/                    # CSS-generation helpers (barrel + individual files)
│   ├── hoc/                        # Higher-order components (withBlockContext, AI wrappers)
│   ├── hooks/                      # React hooks
│   ├── utils/                      # Small utilities
│   ├── icons/                      # Icon assets
│   ├── shape-divider-svg/          # SVG presets for shape-divider control
│   └── extras/                     # Misc widgets
├── dist/                           # Build output (window.EBControls)
├── frontend/                       # Frontend-only controls (smaller bundle)
└── node_modules/
```

The `dist/` dir is committed in some branches — check before assuming.

## 9. Build pipeline

`webpack.config.js` inside the controls repo is independent of the free
plugin's webpack config. It produces:

- `dist/index.js` — the main `window.EBControls` bundle
- `dist/index.asset.php` — WP dependency manifest
- A CSS bundle compiled from `src/backend.scss` + nested SCSS

**Linking for dev:** When you run `npm run start` in the free plugin root,
the `prestart` script creates a symlink:

```bash
pnpm link <path-to-src/controls>   # or npm link
```

This wires `@essential-blocks/controls` → the local submodule source in
`node_modules/`. Changes to controls source show up in free's dev build
automatically.

## 10. Common investigation queries

### "When was this control added?"

```bash
EBC="<free path>/src/controls"
git -C "$EBC" log --reverse --follow -- src/controls/liquid-glass-effect-control/ | head
```

### "Why does my block not see a new control?"

```bash
# Is the submodule pointer current?
git -C "<free path>" submodule status
git -C "$EBC" log origin/master --oneline | head

# Is the new control exported from the barrel?
grep -n "LiquidGlassEffectControl" "$EBC/src/index.js"

# Is the build fresh?
ls -lt "$EBC/dist/"
```

### "Who owns this control?"

```bash
git -C "$EBC" log --format='%an' -- src/controls/custom-query/ | sort | uniq -c | sort -rn
```

### "What's changed in controls since the free plugin cut its release?"

```bash
EB="<free path>"; EBC="$EB/src/controls"
POINTER=$(git -C "$EB" rev-parse HEAD:src/controls)
git -C "$EBC" log "$POINTER..origin/master" --oneline
```

## 11. Common gotchas

- **"Controls commit not found"**: If `git -C src/controls log <sha>`
  fails, the submodule hasn't fetched. Run `git -C src/controls fetch origin`.
- **Dist and source drift**: If `dist/index.js` and `src/index.js` disagree
  (e.g., dist is older), a rebuild is needed. This skill never runs
  builds — recommend the command to the user.
- **Submodule on a detached HEAD**: After `git submodule update`, you're
  usually in detached HEAD state. That's normal — don't panic-commit
  from inside the submodule.
- **Pro may use pro-specific controls**: Pro has its own `src/controls/`
  dir (inside the pro plugin, NOT a submodule — just pro-specific React
  components). Don't confuse "pro controls" with "the shared controls
  submodule".
- **Two separate repos for backend.scss**: Older commits modified
  `backend.scss` at the repo root; newer commits modify `src/backend.scss`.
  When grepping, check both.
- **No version tags**: Don't search for `v1.0.0` — controls releases via
  branches, not tags. Use `origin/release-X.Y.Z` to find release state.

## 12. When to look here vs. `controls-api.md`

| Need                                                         | Read                                    |
|---------------------------------------------------------------|-----------------------------------------|
| List of controls, export names, usage examples                 | `controls-api.md`                       |
| How to add a new control, responsive-value patterns           | `controls-api.md`                       |
| Who introduced a control, when                                 | This file + `git log` queries           |
| Submodule pointer, branching, release alignment                | This file                                |
| File hotspots / churn analysis                                 | This file                                |
| Debug a build/link/link-drift issue                            | This file + `build-and-dev.md`          |
