# wp-eb-dev — Essential Blocks Senior-Dev Skill for Claude Code

A read-only Claude Code skill that gives Claude senior-developer-level
mastery of the entire **Essential Blocks** WordPress plugin ecosystem:

- **Free plugin** — [WPDevelopers/essential-blocks](https://github.com/WPDevelopers/essential-blocks) — 7,992+ commits, 60+ blocks, 166+ releases
- **Pro plugin** — [WPDevelopers/essential-blocks-pro](https://github.com/WPDevelopers/essential-blocks-pro) — 2,395+ commits, 22 standalone blocks + 7 block-extends
- **Controls submodule** — [EssentialBlocks/controls](https://github.com/EssentialBlocks/controls) — 1,427+ commits, 37 controls

The skill enables Claude to investigate bugs, plan new blocks/features,
trace hooks across repos, analyze release impact, and explain architecture
— all while staying strictly **read-only** (no edits, builds, commits, or
pushes to the EB plugin repos).

## What's in this repo

```
wp-eb-dev/
├── SKILL.md                      # Entry point (loaded every session)
└── references/                   # Loaded on-demand, each starts with a TL;DR
    ├── architecture.md           # Free plugin: classes, bootstrap, REST, WPML
    ├── pro-architecture.md       # Pro plugin: dynamic tags, licensing, extends
    ├── controls-api.md           # 37 controls + helpers + hooks (API surface)
    ├── controls-deep-dive.md     # Controls repo history, branching, releases
    ├── blocks-inventory.md       # All 60+ blocks: slug, class, paths, churn
    ├── hooks-reference.md        # Every eb_* action/filter with file:line
    ├── css-pipeline.md           # Attribute → CSS file pipeline + deprecated
    ├── common-bugs.md            # 11 recurring bug patterns from 777 fixes
    ├── release-history.md        # Per-version changelogs v5.9.0→v6.0.7
    ├── investigation-playbook.md # 9 investigation/planning recipes
    ├── build-and-dev.md          # Webpack, npm scripts, dist layout
    ├── git-workflow.md           # Branches, PRs, contributors, releases
    ├── commit-insights.md        # Hotspots, evolution, monthly heatmap
    └── update-guide.md           # How to refresh references from fresh repos
```

15 files, ~4,500 lines. SKILL.md is ~225 lines (kept tight — it's loaded
every session). References average ~280 lines and load only when relevant.

## Quick numbers

| Repo | Commits | Latest | Default branch |
|---|---|---|---|
| Free | 7,992 | v6.0.7 (2026-04-02) | `master` |
| Pro | 2,395 | v2.7.8 / 2.7.9 (2026-04-02) | **`main`** |
| Controls | 1,427 | (no tags) | `master` |

Snapshot date: **2026-04-02**, repo HEAD `fc51997e8` (free).

## Installation

This is a Claude Code skill. Install by copying `wp-eb-dev/` into your
Claude skills directory:

```bash
# For global use (every session, every project):
cp -R wp-eb-dev ~/.claude/skills/

# OR for project-local use (only when running Claude in this project):
mkdir -p .claude/skills
cp -R wp-eb-dev .claude/skills/
```

The skill will auto-activate when you mention Essential Blocks, ask to
investigate an EB bug, plan an EB feature, etc. See `wp-eb-dev/SKILL.md`
for the full trigger list.

## What the skill does

When a user asks about Essential Blocks, Claude uses the skill to:

1. **Locate the local repo clones** (free + pro + controls submodule)
2. **Operate in one of 5 modes:** investigate, plan, explain, impact, issue triage
3. **Load only the relevant reference file(s)** based on the question (each
   reference has a TL;DR for fast triage — no need to read the whole file)
4. **Cite every claim** with `file:line` from the actual repo
5. **Defer execution work** to the plugin's own task-oriented skills (the
   plugin ships its own `.claude/skills/` for `create-block`, `deprecate-block`,
   `add-api-endpoint`, etc.)

## What the skill does NOT do

- **No edits** to the EB repos (free, pro, controls)
- **No builds** (`npm/pnpm install/build` etc.)
- **No git mutations** (no commits, pushes, merges, rebases, branch switches)
- **No fetching of issue tracker URLs** (use the `wp-eb-test` skill for that)

If a user asks for a fix, Claude produces the patch as text in the response
for the user to apply manually.

## Operation handoff

When the user wants to *do* something (not investigate), the skill points
them at the plugin's own write-mode skills:

| User wants to… | Hand off to (plugin's `.claude/skills/`) |
|---|---|
| Scaffold a new block | `create-block` |
| Scaffold a new control | `create-control` |
| Add a control to an existing block | `add-block-control` |
| Modify an existing block | `update-block` |
| Add a `deprecated.js` migration | `deprecate-block` |
| Add a REST endpoint | `add-api-endpoint` |
| Add WPML translatable attributes | `add-wpml-support` |
| Update a monthly campaign notice | `update-campaign-notice` |
| Debug StyleHandler / CSS-not-applying | `debug-style-handler` |

## Refreshing the reference data

The repos move fast (~one release every 2 weeks). When the references
drift from current `origin/master`, follow `wp-eb-dev/references/update-guide.md`
— a 13-step procedure to re-extract commit history, hotspots, and counts.

You can also hand the guide to Claude:

> "Read `~/.claude/skills/wp-eb-dev/references/update-guide.md` and refresh
> the skill's reference files against the current state of the three
> Essential Blocks repos."

## Verification & confidence

Every fact in the references is grounded in actual repo code as of the
snapshot date. Where claims couldn't be 100% verified, they're hedged
("near-universal — 62/66 blocks") or marked for re-grep before acting on
them. Line numbers drift between releases — always re-grep before patching.

## License

MIT (or whatever you prefer — this is your repo).

## Credits

Built and maintained by [@mdismail-cse](https://github.com/mdismail-cse).
