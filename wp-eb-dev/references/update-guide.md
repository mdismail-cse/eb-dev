# Update Guide — Refreshing wp-eb-dev Skill Resources

This skill's reference files are snapshots of the Essential Blocks
ecosystem at the time they were written. The repos move fast (one release
every ~2 weeks on the free plugin), so the snapshots eventually drift.

Use this guide when you want to refresh the skill with fresh data from
`origin/master` (free + controls) and `origin/main` (pro).

**You can run this guide yourself (manually with copy-paste commands), or
hand it to Claude Code and say "refresh the wp-eb-dev skill references
using update-guide.md".**

## When to refresh

- After a major EB release (e.g., free crossing a minor version like 6.1 → 6.2)
- After a large pro feature lands (new block categories, new dynamic tag sources)
- When you notice the skill is citing stale file paths or missing a
  block/control/hook you know exists
- At least once per quarter for hotspot/commit-insight data

## What gets refreshed

The skill's reference files under `~/.claude/skills/wp-eb-dev/references/`:

| File                          | Source of truth                                             | Refresh needed when                                      |
|-------------------------------|-------------------------------------------------------------|----------------------------------------------------------|
| `architecture.md`             | Free plugin structure, classes, PHP flow                    | Major refactors; new top-level dirs; new core classes    |
| `blocks-inventory.md`         | `includes/blocks.php` + `src/blocks/` + `includes/Blocks/`  | Any new block added or removed                           |
| `controls-api.md`             | `src/controls/src/controls/` + `src/controls/src/index.js`  | Any new control or export renamed                        |
| `controls-deep-dive.md`       | Controls repo git history + branches + contributors         | New contributors, branch-convention changes              |
| `hooks-reference.md`          | `do_action`/`apply_filters` across free + pro               | New hooks added or removed                               |
| `build-and-dev.md`            | `package.json` scripts + `webpack.config.js`                | Build config changes                                     |
| `git-workflow.md`             | Branch names, PR conventions, contributor ranks              | Team changes; new branching conventions                  |
| `commit-insights.md`          | Full `git log` analysis (free)                              | Periodic (quarterly); after major release cycles         |
| `pro-architecture.md`         | Pro plugin structure + block manifest + integrations        | Pro version bumps; new pro blocks or integrations        |
| `investigation-playbook.md`   | Generic recipes — rarely changes                             | Only when workflow/tooling changes                       |
| `SKILL.md` (entry)             | Top-level intro + quick numbers                             | When any quick number is >10% off                         |

## Step-by-step refresh procedure

### Step 0 — Prerequisites

You need local clones of all three repos. If you don't have them:

```bash
# Typical dev location
mkdir -p "/Users/ismail/Local Sites/essential-blocks-test-2/app/public/wp-content/plugins"
cd "/Users/ismail/Local Sites/essential-blocks-test-2/app/public/wp-content/plugins"

# Free plugin (with controls submodule)
git clone --recursive git@github.com:WPDevelopers/essential-blocks.git

# Pro plugin
git clone git@github.com:WPDevelopers/essential-blocks-pro.git
```

Set environment variables for the rest of the guide:

```bash
export EB="/Users/ismail/Local Sites/essential-blocks-test-2/app/public/wp-content/plugins/essential-blocks"
export EB_PRO="/Users/ismail/Local Sites/essential-blocks-test-2/app/public/wp-content/plugins/essential-blocks-pro"
export EBC="$EB/src/controls"
export SKILL="$HOME/.claude/skills/wp-eb-dev"
export REF="$SKILL/references"
```

### Step 1 — Fetch fresh data

```bash
git -C "$EB"     fetch origin --tags
git -C "$EB_PRO" fetch origin --tags
git -C "$EBC"    fetch origin --tags    # controls has no tags but harmless
```

Note the tips:

```bash
git -C "$EB"     rev-parse origin/master
git -C "$EB_PRO" rev-parse origin/main
git -C "$EBC"    rev-parse origin/master
```

Also note current submodule pointer in free:

```bash
git -C "$EB" submodule status
```

### Step 2 — Collect headline numbers

These feed into `SKILL.md` (quick numbers) and `commit-insights.md`.

```bash
# Free
echo "Free commits:     $(git -C "$EB" log origin/master --oneline | wc -l)"
echo "Free tags:        $(git -C "$EB" tag | wc -l)"
echo "Free latest tag:  $(git -C "$EB" for-each-ref 'refs/tags/v*' --format='%(refname:short) %(creatordate:short)' --sort=-creatordate | head -1)"
echo "Free latest commit: $(git -C "$EB" log origin/master -1 --format='%h %ad %s' --date=short)"
echo "Free authors:     $(git -C "$EB" log origin/master --format='%an' | sort -u | wc -l)"

# Pro
echo "Pro commits:      $(git -C "$EB_PRO" log origin/main --oneline | wc -l)"
echo "Pro tags:         $(git -C "$EB_PRO" tag | wc -l)"
echo "Pro latest tag:   $(git -C "$EB_PRO" for-each-ref 'refs/tags/*' --format='%(refname:short) %(creatordate:short)' --sort=-creatordate | head -1)"
echo "Pro latest commit:$(git -C "$EB_PRO" log origin/main -1 --format='%h %ad %s' --date=short)"
echo "Pro authors:      $(git -C "$EB_PRO" log origin/main --format='%an' | sort -u | wc -l)"

# Controls
echo "Controls commits: $(git -C "$EBC" log origin/master --oneline | wc -l)"
echo "Controls tags:    $(git -C "$EBC" tag | wc -l)"
echo "Controls latest:  $(git -C "$EBC" log origin/master -1 --format='%h %ad %s' --date=short)"
echo "Controls authors: $(git -C "$EBC" log origin/master --format='%an' | sort -u | wc -l)"
```

### Step 3 — Block inventory (free)

Dump the current block manifest and block directories:

```bash
# Block slugs from manifest:
grep -E "^\s+'[a-z0-9_]+'\s*=>\s*array" "$EB/includes/blocks.php" \
    | sed -E "s/^\s+'([a-z0-9_]+)'.*/\1/" > /tmp/eb-block-slugs.txt
wc -l /tmp/eb-block-slugs.txt

# Block JS directories:
ls "$EB/src/blocks" > /tmp/eb-block-dirs.txt
wc -l /tmp/eb-block-dirs.txt

# Block PHP classes:
ls "$EB/includes/Blocks" > /tmp/eb-block-classes.txt
wc -l /tmp/eb-block-classes.txt

# Diff against what's in blocks-inventory.md:
# → Any new blocks need adding; removed blocks need removing
```

Cross-reference with `blocks-inventory.md` and update tables for any
additions/removals.

### Step 4 — Block inventory (pro)

```bash
ls "$EB_PRO/src/blocks"          > /tmp/pro-block-dirs.txt
ls "$EB_PRO/src/blocks-extends"  > /tmp/pro-block-extends.txt
ls "$EB_PRO/includes/Blocks"     > /tmp/pro-block-classes.txt
grep -E "^\s+'[a-z0-9_]+'\s*=>" "$EB_PRO/includes/blocks.php" \
    | sed -E "s/^\s+'([a-z0-9_]+)'.*/\1/" > /tmp/pro-block-slugs.txt

wc -l /tmp/pro-block-*.txt
```

Update `pro-architecture.md` tables.

### Step 5 — Controls inventory

```bash
ls "$EBC/src/controls" > /tmp/controls-list.txt
wc -l /tmp/controls-list.txt
```

If the count differs from 37, update `controls-api.md` and
`controls-deep-dive.md`.

Also check for renames in `src/controls/src/index.js`:

```bash
head -60 "$EBC/src/index.js"
```

### Step 6 — Hooks audit

```bash
# All action/filter fires in free:
grep -rn "do_action\|apply_filters" "$EB/includes" "$EB/views" 2>/dev/null \
    | grep -oE "(do_action|apply_filters)\( *['\"]eb[a-z_/\\${}:-]*['\"]" \
    | sort -u > /tmp/free-hooks.txt

# Same in pro:
grep -rn "do_action\|apply_filters" "$EB_PRO/includes" "$EB_PRO/views" 2>/dev/null \
    | grep -oE "(do_action|apply_filters)\( *['\"]eb[a-z_/\\${}:-]*['\"]" \
    | sort -u > /tmp/pro-hooks.txt

wc -l /tmp/free-hooks.txt /tmp/pro-hooks.txt
```

Compare against `hooks-reference.md`. Add any new hooks; mark any that
were removed.

### Step 7 — Commit insights (free)

```bash
# Full commit dump:
git -C "$EB" log origin/master --format='%h|%ad|%an|%s' --date=short > /tmp/free-commits.txt

# Authors:
awk -F'|' '{print $3}' /tmp/free-commits.txt | sort | uniq -c | sort -rn > /tmp/free-authors.txt
head -20 /tmp/free-authors.txt

# Monthly activity:
awk -F'|' '{print substr($2,1,7)}' /tmp/free-commits.txt | sort | uniq -c > /tmp/free-monthly.txt
sort -rn /tmp/free-monthly.txt | head -10

# File hotspots (source only, exclude dist/node_modules):
git -C "$EB" log origin/master --format='' --name-only 2>/dev/null \
    | grep -v '^$' | sort | uniq -c | sort -rn \
    | grep -E '(^ +[0-9]+ includes/|^ +[0-9]+ src/blocks/|^ +[0-9]+ src/controls/|^ +[0-9]+ essential-blocks\.php|^ +[0-9]+ webpack|^ +[0-9]+ package\.json)' \
    | head -40 > /tmp/free-hotspots.txt

# Commit count per block (rough):
awk -F'|' '{print tolower($4)}' /tmp/free-commits.txt > /tmp/free-msgs-lower.txt
for b in form button slider accordion post-grid infobox image-gallery team-member toggle-content; do
    c=$(grep -c "\b${b//-/[ -]}\b" /tmp/free-msgs-lower.txt)
    printf "%5d %s\n" "$c" "$b"
done | sort -rn
```

Update `commit-insights.md` numbers.

### Step 8 — Commit insights (pro)

```bash
git -C "$EB_PRO" log origin/main --format='%h|%ad|%an|%s' --date=short > /tmp/pro-commits.txt
awk -F'|' '{print $3}' /tmp/pro-commits.txt | sort | uniq -c | sort -rn | head -10
awk -F'|' '{print substr($2,1,7)}' /tmp/pro-commits.txt | sort | uniq -c | sort -rn | head -10

git -C "$EB_PRO" log origin/main --format='' --name-only 2>/dev/null \
    | grep -v '^$' | sort | uniq -c | sort -rn \
    | grep -E '(^ +[0-9]+ includes/|^ +[0-9]+ src/blocks/|^ +[0-9]+ src/blocks-extends/|^ +[0-9]+ essential-blocks-pro|^ +[0-9]+ webpack|^ +[0-9]+ package\.json|^ +[0-9]+ views/)' \
    | head -30
```

Update `pro-architecture.md`.

### Step 9 — Commit insights (controls)

```bash
git -C "$EBC" log origin/master --format='%h|%ad|%an|%s' --date=short > /tmp/controls-commits.txt
awk -F'|' '{print $3}' /tmp/controls-commits.txt | sort | uniq -c | sort -rn | head -10

git -C "$EBC" log origin/master --format='' --name-only 2>/dev/null \
    | grep -v '^$' | sort | uniq -c | sort -rn | head -30
```

Update `controls-deep-dive.md`.

### Step 10 — Release tag lists

```bash
# Free:
git -C "$EB" for-each-ref 'refs/tags/v*' --format='%(creatordate:short)|%(refname:short)' --sort=-creatordate | head -15

# Pro:
git -C "$EB_PRO" for-each-ref 'refs/tags/*' --format='%(creatordate:short)|%(refname:short)' --sort=-creatordate | head -15

# Controls: (no tags — check release branches instead)
git -C "$EBC" branch -r | grep 'release-' | tail -10
```

Update "recent releases" tables in `git-workflow.md` and `pro-architecture.md`.

### Step 11 — Branch and PR conventions

Check whether branch patterns still hold:

```bash
# Free — 5-digit issue IDs:
git -C "$EB" branch -r | grep -oE '^\s+origin/[0-9]{5}-' | head

# Pro — 4-digit + occasional 5-digit:
git -C "$EB_PRO" branch -r | grep -oE '^\s+origin/[0-9]{4,5}-' | head

# Controls — same mix as pro:
git -C "$EBC" branch -r | grep -oE '^\s+origin/[0-9]{4,5}-' | head
```

If conventions changed, update `git-workflow.md` and `controls-deep-dive.md`.

### Step 12 — Apply updates to reference files

For each reference file, update the affected sections:

1. **`SKILL.md`** — Update the "Quick Numbers" section with fresh counts
2. **`architecture.md`** — Only if file paths or class names changed; otherwise leave
3. **`blocks-inventory.md`** — Add new blocks, remove removed ones; update commit counts
4. **`controls-api.md`** — Add/remove controls from the 37-list
5. **`controls-deep-dive.md`** — Refresh commit count, author count, file hotspots, dates
6. **`hooks-reference.md`** — Add new hooks, remove removed ones
7. **`build-and-dev.md`** — Only if `package.json` scripts changed
8. **`git-workflow.md`** — Refresh contributor counts, release table, recent releases
9. **`commit-insights.md`** — Full refresh of all numbers, hotspots, tables
10. **`pro-architecture.md`** — Update version tables, block lists, hotspots, author counts
11. **`investigation-playbook.md`** — Only if workflow/tooling changes

### Step 13 — Sanity check

```bash
ls -la "$REF"/*.md
for f in "$REF"/*.md; do
    echo "=== $f ==="
    head -5 "$f"
    echo
done
```

Spot-check that each file still reflects reality for the areas you care
about. The skill is ready to use again.

## Letting Claude do the refresh

You can hand this whole guide to Claude Code and say:

> "Read `~/.claude/skills/wp-eb-dev/references/update-guide.md` and refresh
> the skill's reference files against the current state of the three
> Essential Blocks repos."

Claude will:
1. Locate the three clones
2. Fetch fresh
3. Run every query in this guide
4. Diff the results against the existing reference files
5. Apply targeted edits (not full rewrites) to bring them up to date
6. Report back with a summary of what changed

**Caveat:** Claude in this skill is read-only for the EB repos themselves,
but it **can** write to `~/.claude/skills/wp-eb-dev/references/`. That's
the intended exception — reference files are skill-owned, not repo-owned.
So the refresh will edit reference files, not EB source.

## Partial refreshes

You don't have to do everything at once. Common partial refreshes:

**"Just update the blocks list."**
- Step 3 (free blocks) + Step 4 (pro blocks)
- Edit `blocks-inventory.md` + pro section of `pro-architecture.md`

**"Just update the hotspot numbers."**
- Step 7 + Step 8 + Step 9
- Edit `commit-insights.md`, `pro-architecture.md`, `controls-deep-dive.md`

**"Check for new hooks after a pro release."**
- Step 6
- Edit `hooks-reference.md`

**"Just bump the 'as of' dates."**
- Step 2
- Edit the Quick Numbers section in `SKILL.md` + any "as of YYYY-MM" lines
  in the references

## Pitfalls

- **Don't delete commit-count numbers without replacing them** — future
  investigations rely on the hotspot data. Update in place.
- **Don't assume the branch conventions haven't changed** — always spot-check
  actual branches before editing `git-workflow.md`.
- **Don't skip the pro/controls sections** just because free looks unchanged.
  Pro's release cycle is independent; controls has no tags at all.
- **Be careful with the dependency chain note** (pro requires free, free
  ships controls submodule pointer). This doesn't change, but verify.
- **Author name consolidation**: people sometimes commit under multiple
  names (e.g., "MD Jamil Uddin" + "Jamil Uddin" + "Jamil"). When you recompute
  ranks, combine them by hand — a raw `uniq -c` will undercount.
