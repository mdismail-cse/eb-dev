# Essential Blocks — Investigation & Planning Playbook

> **TL;DR:** 9 step-by-step recipes — investigate bug, plan new block,
> trace a hook, reproduce from issue ID, "where is X", produce a patch,
> sanity-check a PR, estimate effort. Each ends with a deliverable.
> All read-only. Read when you need a procedure, not facts.

When a user hands you a bug report, an issue ID, or a feature request,
start with the matching recipe.

## Recipe 1 — Investigate a bug report

**Inputs:** a description (sometimes an issue URL or 5-digit ID), a branch name (maybe).

### Step 1. Frame the problem

Restate the bug in one sentence. If the report is vague, ask:
- Which block?
- What user action triggers it?
- Expected vs actual behavior?
- Editor or frontend?
- Free alone, or with pro active?
- Any specific WordPress / theme / other plugin involved?

### Step 2. Locate the suspect code

Pick the entry point based on symptom:

| Symptom                                                 | Start at                                                       |
|---------------------------------------------------------|----------------------------------------------------------------|
| "Block doesn't render"                                   | `includes/Blocks/<Name>.php::render_callback()` + `views/<name>.php` |
| "Block not in inserter"                                  | `includes/blocks.php` visibility + `ebConditionalRegisterBlockType` in `src/blocks/<name>/src/index.js` |
| "Editor crashes on save"                                 | `src/blocks/<name>/src/edit.js` + attributes schema + save function |
| "Validation error — block contains unexpected content"  | `src/blocks/<name>/src/save.js` + attribute migrations          |
| "CSS not applying"                                       | `src/blocks/<name>/src/style.scss` + `includes/Modules/StyleHandler.php` |
| "Frontend JS broken"                                      | `src/blocks/<name>/src/frontend.js` + enqueue in block PHP     |
| "REST API returns wrong data"                             | `includes/API/<Name>.php` + `Utils/QueryHelper.php`             |
| "Form doesn't submit"                                    | `includes/Integrations/Form.php` + `src/blocks/form/`           |
| "Admin setting doesn't save"                             | `includes/Admin/Admin.php` AJAX handlers + `Utils/Settings.php`  |
| "Pro block shows when pro inactive"                      | `Plugin::filter_pro_blocks_frontend()` + block registration     |
| "Theme conflict"                                         | Theme name in commit log + `StyleHandler.php`                   |
| "Build/asset missing"                                    | `includes/Utils/Enqueue.php` + `webpack.config.js` entries      |

### Step 3. Read and trace

```bash
EB="<free path>"
# Find the suspect file:
grep -rn "search_term" "$EB/includes" "$EB/src/blocks/<name>" "$EB/views"

# Trace call flow up:
grep -rn "function_name" "$EB/"

# Trace call flow down (what does this function do):
# → Read the file at the line numbers found
```

### Step 4. Check git history for context

```bash
# Recent changes to the suspect file:
git -C "$EB" log --oneline -10 -- path/to/file.php

# Who last touched this line (blame):
git -C "$EB" blame -L 100,140 -- path/to/file.php

# Did a recent commit introduce the regression?
git -C "$EB" log --since='30 days ago' --oneline -- path/to/file.php

# Compare against a known-good tag:
git -C "$EB" diff v6.0.5..v6.0.7 -- path/to/file.php
```

### Step 5. Check cross-repo dependencies

If the bug is in controls:
```bash
EBC="<free path>/src/controls"
git -C "$EBC" log --oneline -10 -- src/controls/<control>/
# Submodule pointer in parent:
git -C "$EB" log --oneline -- src/controls
```

If the bug might involve pro:
```bash
EB_PRO="<pro path>"
grep -rn "essential-blocks/<name>" "$EB_PRO/"
grep -rn "add_filter.*eb_.*<name>" "$EB_PRO/"
```

### Step 6. Produce the report

Deliverable — paste this into your response:

```
## Root cause analysis: <one-line bug summary>

**Affected block / area:** <block name or module>
**Severity:** <user-visible? editor-only? silent data loss?>
**Introduced in:** <commit hash and date, if identifiable>
**Last-known-good:** <commit/tag, if identifiable>

### Where it happens
- `path/to/file.php:LINE` — <what the code does>
- `path/to/other.js:LINE` — <how it's called>

### Why it happens
<Explain the faulty logic. Quote minimal code if needed.>

### Cross-component impact
- [ ] Free only
- [ ] Controls submodule
- [ ] Pro plugin (explain why)

### Suggested fix (high-level, not a patch unless asked)
<One-paragraph fix direction.>

### Test plan (for the fix)
- <test case 1>
- <test case 2>

### Red flags noticed along the way
<security / accessibility / i18n / perf concerns unrelated to the bug>
```

## Recipe 2 — Plan a new block

**Inputs:** block name, rough requirements, static vs dynamic.

### Step 1. Clarify scope

Ask:
- Static (client-saved JSX) or dynamic (server-rendered)? If the block
  pulls data from DB or needs conditional rendering → dynamic.
- Free or pro?
- Which existing block is most similar? (Use that as the template.)
- Does it need any new control that doesn't exist yet?
- Frontend interactivity? (sliders, carousels, filters, AJAX)

### Step 2. Pick a reference block

Find a similar existing block to mirror:

- Content block, static, simple → `button`, `infobox`, `advanced-heading`
- Content block, static, nested children → `accordion`, `advanced-tabs`
- Dynamic, query-driven → `post-grid`, `post-carousel`
- Dynamic, API-driven → `instagram-feed`, `nft-gallery`, `google-map`
- Form-related → `form` + child field blocks under `form-*-field/`
- WooCommerce → `woo-product-grid`

Read the reference block end-to-end (`src/blocks/<ref>/`, `includes/Blocks/<Ref>.php`,
`views/<ref>.php` if dynamic) before planning.

### Step 3. Produce the plan

Deliverable:

```
## Plan: Add `<block-name>` block

**Type:** static / dynamic
**Category:** content / creative / marketing / query / forms / woocommerce
**Reference block:** <existing block this mirrors>
**Estimated scope:** <small / medium / large>

### Files to create

- `src/blocks/<name>/block.json` — WP metadata
- `src/blocks/<name>/src/index.js` — ebConditionalRegisterBlockType call
- `src/blocks/<name>/src/edit.js` — Editor component
- `src/blocks/<name>/src/save.js` — () => null (dynamic) OR JSX (static)
- `src/blocks/<name>/src/attributes.js` — Attribute schema
- `src/blocks/<name>/src/style.scss` — Frontend styles
- `src/blocks/<name>/src/editor.scss` — Editor-only styles
- `src/blocks/<name>/src/frontend.js` — Frontend JS (if interactive)
- `src/blocks/<name>/src/icon.js` — Block icon
- `includes/Blocks/<Name>.php` — PHP block class extending Block (or PostBlock)
- `views/<name>.php` — Server-render template (dynamic only)

### Files to modify

- `includes/blocks.php` — Register new block in manifest
- `webpack.config.js` — Add frontend entry for the block (if interactive)
- `languages/essential-blocks.pot` — Regenerate after adding strings

### Attributes to define

- <attr1>: type, default, responsive?
- <attr2>: ...

### Controls to use (from @essential-blocks/controls)

- <ControlName> for <what>
- ...

### Hooks/filters to expose

- `eb_frontend_styles/<name>` — auto (from Block base)
- `eb_frontend_scripts/<name>` — auto
- (Add any custom block-specific filters needed)

### Free ↔ Pro considerations

- [ ] Is any part pro-only? Gate it via `ESSENTIAL_BLOCKS_IS_PRO_ACTIVE`
- [ ] Does pro extend this block? Expose filter points if so.

### Risks / gotchas

- <e.g., "Block shares controls with X — changes might ripple">
- <e.g., "Frontend script size — keep under 50KB">

### Test plan

1. Inserts cleanly in editor; no console errors
2. All responsive controls save/restore across devices
3. Save → reload → block renders with saved state
4. Frontend renders matches editor preview
5. (If dynamic) render_callback returns valid HTML for empty state
6. (If form-related) integrates with existing form flow
7. Pro deactivated: block (or its pro parts) degrade gracefully
```

## Recipe 2.5 — Modify an existing block

**Inputs:** block name + change description.

### Step 1. Classify the change shape

Different change types require different files. Use this matrix:

| Change shape | Files to touch | `deprecated.js` needed? |
|---|---|---|
| Editor-only bug (inspector/UX) | `edit.js` (or `components/edit.js`) | No |
| Save markup change (HTML/wrapper/class) | `save.js` + `deprecated.js` | **YES** |
| New attribute added | `attributes.js` + `edit.js` + `style.js` | Only if `save()` output changes |
| Attribute renamed/removed | `attributes.js` + `edit.js` + `save.js` + `deprecated.js` with `migrate()` | **YES** |
| New control in inspector | `attributes.js` + `edit.js` (4 files if new generator pair: + `style.js` + `constants.js`) | No |
| CSS-only fix in style.js | `style.js` (or `components/style.js`) | No |
| Frontend JS bug | `frontend.js` | No |
| PHP render_callback fix | `includes/Blocks/<Pascal>.php` | No |
| Free → Pro feature gate | `includes/Blocks/<Pascal>.php` + check `ESSENTIAL_BLOCKS_IS_PRO_ACTIVE` | No |

### Step 2. Read history before touching

```bash
EB="<free path>"
# Recent activity in this block:
git -C "$EB" log --since='6 months ago' --oneline -- src/blocks/<name>/ includes/Blocks/<Pascal>.php

# Volatility — has anyone touched this same file recently?
git -C "$EB" log --oneline -- src/blocks/<name>/src/<file>.js | head -10
```

### Step 3. If editing `save()` output → ALSO add `deprecated.js`

Without a deprecated entry, every existing instance of this block in the
wild will throw "block contains unexpected content" on next edit. See
`css-pipeline.md` "deprecated migration system" for the pattern.

### Step 4. After patching, run the project's static audit

The plugin ships a sub-agent at `<plugin-checkout>/.claude/agents/block-audit.md`
that checks 7 invariants (file existence, block.json validity, deprecated
coverage, generator pairs, registration shape). Recommend the user invoke
it before committing.

## Recipe 3 — Plan a feature or refactor across blocks

**Inputs:** description of the change ("add dark mode to all heading blocks",
"migrate to dimensions-control-v2", etc.).

### Step 1. Inventory impact

```bash
EB="<free path>"
# Which blocks import the old thing?
grep -rln "OldControlName" "$EB/src/blocks/"

# Which controls expose it?
grep -rln "OldControlName" "$EB/src/controls/src/controls/"

# Which PHP files reference it?
grep -rn "old_attribute_name" "$EB/includes/"
```

Count affected files. If > 10, plan the migration in waves.

### Step 2. Identify risk

- Old saved blocks in the wild — will they still render? Attribute
  migrations (`deprecated` arrays) may be needed.
- Free ↔ pro — does pro re-export/extend the thing?
- Controls submodule — if the change is in controls, both free and pro
  rebuild required.
- Webpack entries — new files mean new entries.

### Step 3. Propose phasing

Typical phasing for a cross-block change:

1. Add new API alongside old (both exported from controls)
2. Migrate blocks one by one, test after each
3. Mark old API deprecated (console warning in dev)
4. Remove old API in a major version

Deliverable: a phased plan with file lists per phase, risks, and
rollback path.

## Recipe 4 — Trace an action or filter

**Input:** a hook name.

```bash
EB="<free path>"
EB_PRO="<pro path>"

# Who fires it:
grep -rn "do_action\( *['\"]HOOK_NAME" "$EB" "$EB_PRO"
grep -rn "apply_filters\( *['\"]HOOK_NAME" "$EB" "$EB_PRO"

# Who listens:
grep -rn "add_action\( *['\"]HOOK_NAME" "$EB" "$EB_PRO"
grep -rn "add_filter\( *['\"]HOOK_NAME" "$EB" "$EB_PRO"
```

Produce:
- Hook signature (args + expected return type for filters)
- Firing sites (`file:line`)
- Listener sites (`file:line`)
- Default/initial value
- Typical use cases
- Risk of breaking existing listeners if signature changes

See also: `hooks-reference.md` for the exhaustive list of known EB hooks.

## Recipe 5 — Reproduce from an issue URL

If the user hands you `https://projects.startise.com/fbs-80756` or similar:

1. Extract the issue number (`80756`)
2. Search for a matching branch and its commits:
   ```bash
   git -C "$EB" branch -a | grep 80756
   git -C "$EB" log --all --grep='80756' --oneline
   ```
3. Read any commits on that branch to understand what was attempted
4. If the issue text isn't accessible (this skill doesn't fetch PM URLs),
   ask the user to paste the reproduction steps
5. Follow Recipe 1 (Investigate) from there

**Don't** try to log in to the PM system. The `wp-eb-test` skill has
that flow, not this one.

## Recipe 6 — "Where is X defined?"

Common lookups:

| "Where is…"                                  | Try                                                        |
|-----------------------------------------------|------------------------------------------------------------|
| A block's PHP class                          | `includes/Blocks/<PascalCase>.php`                         |
| A block's edit component                     | `src/blocks/<kebab>/src/edit.js`                           |
| A block's attributes                         | `src/blocks/<kebab>/src/attributes.js`                     |
| A block's frontend script                    | `src/blocks/<kebab>/src/frontend.js`                       |
| The block manifest / enable list              | `includes/blocks.php`                                      |
| A control component                           | `src/controls/src/controls/<kebab>/`                       |
| A control export                              | `src/controls/src/index.js` (grep for the name)           |
| A helper function (JS)                        | `src/helpers/` or `src/controls/src/helpers/`              |
| A helper function (PHP)                       | `includes/Utils/Helper.php` or `includes/Utils/QueryHelper.php` |
| A REST endpoint                               | `includes/API/<Name>.php`                                  |
| An AJAX handler                               | `grep 'wp_ajax_eb_' includes/Admin/Admin.php`              |
| A settings option                             | `get_option('eb_settings')` — see `Utils/Settings.php`    |
| A constant (ESSENTIAL_BLOCKS_*)               | `includes/Plugin.php::define_constants()`                  |
| A CSS class in output                         | `views/*.php` (server-rendered) or `src/blocks/*/src/save.js` (static) |
| A CSS rule                                    | `src/blocks/*/src/style.scss` or inline via `StyleHandler` |
| A webpack entry                               | `webpack.config.js`                                         |
| An admin menu page                            | `includes/Admin/Admin.php::admin_menu()`                   |

Use `grep -rn` first, `git log -S` if the thing was renamed/moved.

## Recipe 7 — Produce a patch (text-only)

The user asks: "Fix it." If the fix is small and scoped, produce a patch
in your response. Do **not** use `Edit` or `Write` against the repo.

Format:

````
### Patch: `includes/Blocks/PostGrid.php`

```diff
--- a/includes/Blocks/PostGrid.php
+++ b/includes/Blocks/PostGrid.php
@@ -120,7 +120,7 @@ class PostGrid extends PostBlock {
     public function render_callback( $attributes, $content ) {
-        $posts = QueryHelper::get_posts( $attributes['queryData'] );
+        $posts = QueryHelper::get_posts( $attributes['queryData'] ?? [] );
         return $this->render_template( 'post-grid', [ 'posts' => $posts, 'attrs' => $attributes ] );
     }
 }
```

### How to apply
1. Open the file at the path above
2. Replace the line inside `render_callback`
3. Save and test with a post-grid block that has no query configured
````

If the fix is multi-file or touches >30 lines, instead give a step-by-step
plan with one minimal patch per step. Don't produce a single 500-line diff.

## Recipe 8 — Sanity-check a PR

**Input:** a branch name or a PR number.

```bash
EB="<free path>"
# Diff against master:
git -C "$EB" diff origin/master...origin/<branch> --stat
git -C "$EB" log origin/master..origin/<branch> --oneline

# Whole diff for review:
git -C "$EB" diff origin/master...origin/<branch>

# Files in the PR:
git -C "$EB" diff --name-only origin/master...origin/<branch>

# Submodule pointer moved?
git -C "$EB" diff origin/master...origin/<branch> -- src/controls
```

Walk through the diff with this checklist:

- [ ] Each touched PHP file: sanitization, escaping, nonces, caps
- [ ] Each touched block JS: attribute schema changes → migration needed?
- [ ] Each touched CSS: scoped to block class, no global leakage
- [ ] New files: correct directory, naming convention
- [ ] Hook additions: follow `eb_` prefix, documented
- [ ] `blocks.php` entry added if new block
- [ ] Submodule pointer change if controls touched
- [ ] `readme.txt` changelog entry (for user-visible changes)

Deliverable: a review comment as bullet points, grouped by Concern /
Suggestion / Nit.

## Recipe 9 — Estimate effort and risk

When asked "how big is this?", look at:

- **# of files touched** (from diff or plan)
- **Hotness of those files** (commit-insights.md hotspots)
- **Cross-repo reach** (free only, or controls+free, or all three)
- **Number of consumers** of any shared code being changed
- **Test coverage** (usually low — assume manual testing dominates)

Report: "small / medium / large" + "low / medium / high risk" + the top
2-3 reasons for each rating.

## General principles

- **Ground every claim in a path:line citation.** If you can't, grep first.
- **Never trust a stale memory.** If the reference files say something,
  verify it still matches the current source before acting on it.
- **When in doubt, ask.** A clarifying question beats a wrong plan.
- **Don't touch the repo.** Produce patches as text, not edits.
- **Cite the hotspots.** If you're recommending a change to `Plugin.php`,
  acknowledge it's a heavily-touched file and suggest minimal-surface
  changes.
