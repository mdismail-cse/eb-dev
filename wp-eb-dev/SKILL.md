---
name: wp-eb-dev
description: >
  Senior-developer-level mastery of the entire Essential Blocks ecosystem:
  the free plugin (github.com/WPDevelopers/essential-blocks), the pro plugin
  (github.com/WPDevelopers/essential-blocks-pro), and the shared controls
  submodule (github.com/EssentialBlocks/controls). Use when the user wants
  to investigate EB bugs in free/pro/controls, trace where a block/control/
  hook is defined across the three repos, plan a new block or feature,
  analyze cross-repo impact, read an issue and explain root cause, or
  understand the architecture, build systems, commit history, release
  alignment, or free↔pro↔controls interop. Also trigger when the user says
  "eb dev", "essential blocks dev", "investigate eb", "plan eb feature",
  "eb architecture", "eb hooks", "eb blocks", "eb controls", "eb pro",
  "essential blocks pro", or references any of the three repos. Read-only —
  analyzes, explains, and produces plans/patches as text. Never edits,
  builds, commits, or pushes code.
---

# Essential Blocks — Senior Dev Companion

Senior-dev across three coupled repos (free + pro + controls). Read-only.

## Repo map

| Repo | GitHub | Default branch | Path |
|---|---|---|---|
| Free | `WPDevelopers/essential-blocks` | `master` | `wp-content/plugins/essential-blocks/` |
| Controls | `EssentialBlocks/controls` (submodule of free) | `master` | `…/essential-blocks/src/controls/` |
| Pro | `WPDevelopers/essential-blocks-pro` | **`main`** ← not master | `wp-content/plugins/essential-blocks-pro/` |

**Dependency chain:** Pro requires free (boots on `essential_blocks::init`).
Free embeds controls as a submodule. Pro can replace free blocks
(`ProForm` overrides `Form`) and extends others via `src/blocks-extends/`.

**Snapshot:** Free 7,992 commits / v6.0.7 · Pro 2,395 / 2.7.9 · Controls
1,427 / no-tags. As of `fc51997e8` (free) on 2026-04-02.

## How to operate

**Mode** — pick one from the user's request, ask if ambiguous:

| Mode | When | Output |
|---|---|---|
| investigate | "why is X broken" | root cause with `file:line` cites |
| plan | "add block / refactor" | step plan + files + risks + tests |
| explain | "how does X work" | grounded explanation |
| impact | "if I change X what breaks" | dependency trace across repos |
| issue triage | "issue #80756" | map to files, propose repro |

**Truth hierarchy** (when reference contradicts code, code wins):
1. Live source via `Read` / `Grep` / `git`
2. Reference files in `references/`
3. Memory / common sense

Always cite `file:line`. If you can't, grep first.

## Read-only contract

OK: `Read`, `Grep`, `Glob`, `git log/diff/show/blame/fetch`, write to `/tmp/`.

NOT OK: any git mutation (`add`, `commit`, `push`, `merge`, `rebase`, `reset`,
`stash`, `checkout`, `tag`, `rm`, `mv`, `clean`), editing repo files
(`Edit`/`Write` on EB paths), running builds (`npm/pnpm install/build`),
shell redirects into repo paths.

If asked to fix → produce the patch as a code block in your response.
Skill may write to `~/.claude/skills/wp-eb-dev/references/` only when
explicitly refreshing the skill (see `update-guide.md`).

## Starting checklist

```bash
# 1. Locate (last-known: /Users/ismail/Local Sites/essential-blocks-test-2/...)
find /Users/ismail -maxdepth 8 -name ".git" -path "*plugins/essential-blocks*" -type d 2>/dev/null

# 2. Orient (always git -C, never cd)
EB="<free path>"; EB_PRO="<pro path>"; EBC="$EB/src/controls"
git -C "$EB" rev-parse --abbrev-ref HEAD
git -C "$EB" submodule status      # controls SHA pinned by free

# 3. Issue ID? Branches follow {5-digit-id}-{name} in free, {4-digit} in pro
git -C "$EB" branch -a | grep "<issue_id>"
```

If repo not found → ask the user, don't guess paths.

## Operation handoff (when user wants to DO, not investigate)

The plugin ships its own task-oriented skills at
`<plugin-checkout>/.claude/skills/`. They run inside the plugin checkout
with write access. wp-eb-dev stays read-only. If the user asks to
perform one of these operations, point them at the matching skill:

| User wants to… | Hand off to (in plugin's `.claude/skills/`) |
|---|---|
| Scaffold a new block | `create-block` |
| Scaffold a new control | `create-control` |
| Add a control to an existing block | `add-block-control` |
| Modify an existing block | `update-block` |
| Add a `deprecated.js` migration | `deprecate-block` |
| Add a REST endpoint | `add-api-endpoint` |
| Add WPML translatable attrs | `add-wpml-support` |
| Update a monthly campaign notice | `update-campaign-notice` |
| Debug StyleHandler / CSS-not-applying | `debug-style-handler` |

For new-block sanity checks the plugin also ships a `block-audit`
sub-agent at `.claude/agents/block-audit.md` (7 static checks).

You can still investigate, plan, and explain those operations — just
don't execute them. Produce a plan, then say "to actually scaffold,
run the plugin's `create-block` skill."

## Reference files (load on demand, not pre-emptively)

Each reference starts with a TL;DR — read that first to confirm relevance.

| User asks about… | Read |
|---|---|
| Free architecture, classes, bootstrap | `architecture.md` |
| **REST API design** (namespace, Base, Server, response patterns) | `architecture.md` §REST |
| **WPML config** (`wpml-config.xml` grammar, what to translate) | `architecture.md` §WPML |
| Pro plugin (any aspect) | `pro-architecture.md` |
| Controls API (37 controls, how to add one) | `controls-api.md` |
| Controls repo history (who/when/branches) | `controls-deep-dive.md` |
| Specific free block | `blocks-inventory.md` |
| Hooks (`do_action`/`apply_filters`) | `hooks-reference.md` |
| **CSS not applying / `blockMeta` / deprecated** | `css-pipeline.md` |
| **Bug investigation (any)** | `common-bugs.md` first, then `investigation-playbook.md` |
| What changed in vX.Y.Z | `release-history.md` |
| Build, webpack, dist, scripts | `build-and-dev.md` |
| Branches, PRs, contributors | `git-workflow.md` |
| Commit hotspots, file churn | `commit-insights.md` |
| Refresh skill from fresh repos | `update-guide.md` |

Cross-repo questions → start with the relevant free reference, then
`pro-architecture.md`, then grep across all three.

## Conventions to memorize (no lookup needed)

**Naming:** JS slug kebab → PHP class Pascal → WP name `essential-blocks/<kebab>` →
asset handle `essential-blocks-<name>-frontend`. Attributes camelCase
(`columnNumber`). Options/hooks `eb_*` snake_case. blocks.php registry key
uses underscores (`my_block`), not dashes. Text domain literal
`"essential-blocks"` everywhere. JS imports always `@wordpress/element`,
never raw `react`.

**Block registration (mandatory):** Use `ebConditionalRegisterBlockType`
from `@essential-blocks/controls`, NEVER raw `registerBlockType`. It
handles pro/free gating + admin-toggle merge + category registration.

**4 mandatory core attributes** (verified in 5/5 sampled blocks):
Every block's `attributes.js` includes `resOption` (string, default
`"Desktop"`), `blockId` (string), `blockRoot` (string, default
`"essential_block"`), `blockMeta` (object). Removing any breaks the
responsive system, style isolation, or CSS pipeline.

**Edit/Save wrappers (near-universal — 62/66 blocks):** Edit exports as
`memo(withBlockContext(defaultAttributes)(Edit))`. Edit JSX wrapped in
`<BlockProps.Edit>` (133+ instances); Save in
`<BlockProps.Save attributes={attributes}>` (160+ instances). Outer
wrapper class `eb-parent-wrapper eb-parent-${blockId} ${classHook}`,
inner `eb-{name}-wrapper ${blockId}`. CSS selectors target
`.eb-{name}-wrapper.${blockId}`.

**Block categories enum** (8 values — verified):
`content`, `creative`, `dynamic`, `form`, `layout`, `marketing`,
`social`, `woocommerce`.

**File layouts (BOTH coexist — always check both when grep'ing):**
- **Layout B — flat (DOMINANT, ~60 of 66 blocks):** `src/blocks/<name>/src/{edit,save,style,inspector}.js`
- **Layout A — components/ subdir (rare, ~few blocks like `button`):** `src/blocks/<name>/src/components/{edit,save,style,inspector}.js`

**Block types:** Dynamic = `save: () => null` + PHP `render_callback()` + `views/<name>.php`.
Static = JSX `save()`, no render_callback.

**Singleton trait:** `EssentialBlocks\Traits\HasSingletone` (note the
misspelling). Always `ClassName::get_instance()`, never `new`.

**Pro detection:** `defined('ESSENTIAL_BLOCKS_IS_PRO_ACTIVE') && ESSENTIAL_BLOCKS_IS_PRO_ACTIVE`.
Backstop filter: `Plugin::filter_pro_blocks_frontend()`.

**Responsive values:** flat sibling keys with device suffix
(`columnNumberDesktop` / `…Tablet` / `…Mobile`), not nested objects.

**CSS pipeline:** JS `style.js` → `StyleComponent.js` minifies → saves to
`blockMeta` attribute → PHP `StyleHandler::write_css_from_content()` →
file at `uploads/eb-style/eb-style-{postId}.min.css` → enqueued as
`eb-block-style-{postId}` on `wp_enqueue_scripts`. Full trace in
`css-pipeline.md`.

## Red flags to scan for (in any code review)

PHP: missing `esc_html`/`esc_attr`/`esc_url`/`wp_kses_post` on output;
raw `$_POST`/`$_GET` without `sanitize_*`; missing `wp_verify_nonce` /
`check_ajax_referer` on AJAX; missing `current_user_can` on privileged ops;
raw `$wpdb` without `->prepare()`; hardcoded strings without `__()`.

JS: `setAttributes` without nullish-coalescing on undefined nested attrs;
new `save()` output without a `deprecated` entry → "unexpected content"
errors; pro features in free without `ESSENTIAL_BLOCKS_IS_PRO_ACTIVE` gate;
submodule pointer change without matching free release branch.

Flag in your response — don't fix unless asked.

## Hotspots (highest-risk files to touch)

Free: `Plugin.php` 232 commits · `essential-blocks.php` 230 · `Admin.php` 178 ·
`Scripts.php` 146 · `blocks.php` 132 · `Form.php` 51 · `StyleHandler.php` 42 ·
`Block.php` 40. Volatile blocks: form, button, slider, accordion, post-grid,
image-gallery, lottie, timeline.

Pro: `essential-blocks-pro.php` 86 · `Scripts.php` 44 · `FormBlockHandler.php`
41 · `HandleTagsResult.php` 33. Hot block: mega-menu.

Controls: `backend.scss` 116 · `index.js` 95 · `custom-query/index.js` 38.

Use these for risk assessment in plans/reviews. Full data in
`commit-insights.md`.

## Ask, don't guess

Stop and ask if unclear: which repo path · which branch · pro in scope?
· trace impact or focus on one block · fix patch or root-cause only.
A clarifying question is cheaper than a wrong answer.
