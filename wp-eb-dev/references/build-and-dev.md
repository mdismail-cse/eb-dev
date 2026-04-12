# Essential Blocks — Build & Dev Reference

How the plugin is built, how scripts wire together, and where compiled
artifacts land. This skill never runs builds itself — this reference
exists so you can explain, plan, and diagnose build issues.

## Tooling

- **Package manager:** `pnpm` (primary) — `pnpm-lock.yaml` is committed
- **Node build:** `@wordpress/scripts` — wp-scripts handles webpack, ESLint,
  Babel, PostCSS, SVGR presets
- **Release tools:** Grunt tasks in `Gruntfile.js` for distribution zip

`package.json` engines may or may not pin a Node version — check it at the
top of the file. Typical working version is Node LTS 18+.

## npm / pnpm scripts (from `package.json`)

| Script               | What it does                                                                    |
|----------------------|---------------------------------------------------------------------------------|
| `prestart`           | `pnpm link` / `npm link` the `src/controls` submodule into `node_modules/`      |
| `start`              | `wp-scripts start` — dev watch build, fast rebuilds, sourcemaps                 |
| `prebuild`           | Same as `prestart` — ensure submodule is linked before production build         |
| `build`              | `wp-scripts build` — production minified build                                   |
| `postbuild`          | Post-processing (e.g., copy assets, patch references)                            |
| `release`            | `scripts/release.sh` + `build` — full distribution release                       |
| `packages-update`    | Update `@wordpress/*` packages to latest                                         |
| `pot`                | Regenerate `.pot` file in `languages/essential-blocks.pot`                      |
| `lint:js`            | `wp-scripts lint-js src`                                                         |
| `lint:js:fix`        | Same with `--fix`                                                                 |

**The submodule link step** (`prestart` / `prebuild`) is critical. Without
it, `@essential-blocks/controls` can't be resolved and every block bundle
fails to build. If you see "cannot find module @essential-blocks/controls",
re-running `npm run prestart` (or deleting `node_modules/@essential-blocks`
and re-linking) fixes it.

## Webpack config (`webpack.config.js`)

The config extends `@wordpress/scripts/config/webpack.config` and adds
40+ entry points. High-level structure:

```
entries = {
    // --- Controls library (built separately as window.EBControls) ---
    'admin/controls/controls':          'src/controls/src/index.js',
    'admin/controls/frontend-controls': 'src/controls/src/frontend/index.js',

    // --- Block editor bundle (all block edit.js files combined) ---
    'admin/editor/editor':   './src/blocks/<dynamic entry collector>',
    'admin/editor/frontend': './src/blocks/<frontend CSS collector>',

    // --- Per-block frontend scripts (60+ entries) ---
    'blocks/post-grid/frontend':        './src/blocks/post-grid/src/frontend.js',
    'blocks/slider/frontend':           './src/blocks/slider/src/frontend.js',
    // ... one per block

    // --- Admin React apps ---
    'admin/dashboard/admin':     './src/admin/dashboard/index.js',
    'admin/quick-setup/index':   './src/admin/quick-setup/index.js',

    // --- Store ---
    'admin/store/store':         './src/store/index.js',

    // --- Modules (feature bundles) ---
    'modules/write-with-ai/index':     './src/modules/write-with-ai/index.js',
    'modules/ai-for-woo/index':        './src/modules/ai-for-woo/index.js',
    'modules/templately-installer/index': './src/modules/templately-installer/index.js',

    // --- Global styles ---
    'admin/global-styles/global-styles': './src/global-styles/index.js',
}
```

### Key externals / aliases

```javascript
externals: {
    '@essential-blocks/controls': 'EBControls',    // window.EBControls
    '@wordpress/element':         'wp.element',
    '@wordpress/i18n':            'wp.i18n',
    // ... other wp globals
}
```

The `@essential-blocks/controls` externalization is what lets each block
bundle stay small. The compiled controls bundle is loaded once as a global,
and all blocks reference it by global variable.

### Plugins in use

- `CopyWebpackPlugin` — copies static assets (block.json, icons)
- `MiniCssExtractPlugin` — extracts CSS to separate files
- `RemoveEmptyScriptsPlugin` — removes empty JS output for CSS-only entries
- `ImportValidationPlugin` (custom) — validates that controls imports resolve

### SVGR

`.svg` files can be imported as React components via SVGR config in
`babel.config.json` / `webpack.config.js`. This is how block icons are
authored — write SVG, import as component, drop in JSX.

## Build output (`dist/`)

Typical structure after a production build:

```
dist/
├── admin/
│   ├── controls/
│   │   ├── controls.js                 # window.EBControls library
│   │   ├── controls.css                # Control styles
│   │   ├── controls.asset.php          # WP deps manifest
│   │   └── frontend-controls.js
│   ├── editor/
│   │   ├── editor.js                   # All block edit components
│   │   ├── editor.css
│   │   ├── editor.asset.php
│   │   ├── frontend.js                 # Combined block frontend styles loader
│   │   └── frontend.css
│   ├── dashboard/
│   │   ├── admin.js                    # Dashboard React app
│   │   ├── admin.css
│   │   └── admin.asset.php
│   ├── quick-setup/
│   │   └── index.js
│   ├── store/
│   │   └── store.js
│   └── global-styles/
│       └── global-styles.js
├── blocks/
│   ├── post-grid/
│   │   ├── frontend.js                 # Per-block frontend script
│   │   ├── frontend.css
│   │   └── frontend.asset.php
│   ├── slider/
│   │   └── ...
│   └── ... (60+ block dirs)
├── modules/
│   ├── write-with-ai/index.js
│   ├── ai-for-woo/index.js
│   └── templately-installer/index.js
└── block-manifest.json                 # All block registrations
```

The `.asset.php` files are auto-generated dependency manifests —
`@wordpress/scripts` produces them and PHP `register_script()` calls read
them to pass deps to WP.

**Legacy output:** Some older blocks still write compiled `block.json` and
icons into `assets/blocks/<name>/` rather than `dist/`. The PHP block
classes reference `ESSENTIAL_BLOCKS_BLOCK_DIR` = `'assets/blocks/'`, so
the build pipeline copies/links compiled assets back into `assets/blocks/`
for WP `register_block_type()` to pick them up.

## Scripts directory (`scripts/`)

Release/automation shell scripts — not part of the normal dev loop.

| File                                        | Purpose                                       |
|---------------------------------------------|-----------------------------------------------|
| `scripts/release.sh`                        | Main release orchestrator                     |
| `scripts/config/release-config.yaml`        | Version, changelog paths, distribution config |
| `scripts/utils/backup.sh`                   | Pre-release backup of working tree            |
| `scripts/utils/validation.sh`               | Validation checks (linting, syntax)           |
| `scripts/utils/file-utils.sh`               | File ops (copy, rename, clean)                |
| `scripts/utils/notice-handler.sh`           | Update release notices in readme.txt          |

`npm run release` invokes these in sequence, then triggers `wp-scripts build`.

## Grunt tasks (`Gruntfile.js`)

Legacy distribution tasks. Primarily used to:
- Copy plugin files to a temp dir
- Remove dev-only files (`.gitignore`, `node_modules`, `src/`)
- Create a distributable `.zip` for WordPress.org / manual install

Most day-to-day dev doesn't touch Grunt. It runs automatically from
`scripts/release.sh`.

## ESLint / Prettier

- ESLint config inherited from `@wordpress/scripts` (`.eslintrc.js` or
  package.json eslint entry)
- Run with `npm run lint:js`
- No Prettier config committed — wp-scripts handles formatting via ESLint

## PHPUnit

`phpunit.xml.dist` is present but the test suite is minimal. Don't assume
coverage exists for areas you're changing — check the `tests/` directory
(if present) before claiming "tests pass".

## Typical dev loop

1. Clone free plugin with submodule: `git clone --recursive <url>`
2. Install deps: `pnpm install` (or `npm install`)
3. Start dev build: `npm run start` (watches and rebuilds on change)
4. Activate plugin in a local WP install
5. Edit → webpack rebuilds → refresh editor/frontend

If you touch controls (`src/controls/src/`), the submodule itself has to be
rebuilt too — `npm run start` at the free root **does** cover this because
the link-step ensures the submodule source is watched. But if you've done
a fresh install and something's off, run `pnpm install` inside
`src/controls/` as well.

## Common build failure modes

| Symptom                                                | Likely cause                                                 | Fix (plan — don't run it yourself)                       |
|--------------------------------------------------------|--------------------------------------------------------------|----------------------------------------------------------|
| `Cannot find module @essential-blocks/controls`        | Submodule not linked                                          | Re-run `npm run prestart` or delete and reinstall         |
| Blocks missing in editor after build                    | Submodule pointer out of sync with source                    | Rebuild + check `git submodule status`                   |
| CSS from recent SCSS change not showing                 | `dist/` stale; browser cache; or wp-scripts watcher hung     | Kill watcher, delete `dist/`, rebuild                     |
| `SyntaxError: Unexpected token` in production           | Missing Babel preset for new syntax                          | Check `babel.config.json` for up-to-date presets         |
| Empty `.js` bundle for CSS-only entry                   | Normal — `RemoveEmptyScriptsPlugin` removes it               | Not a bug                                                 |
| Controls bundle 10x size                                | Accidentally bundled `@wordpress/*` into controls            | Check webpack externals in `src/controls/webpack.config.js` |
| `register_block_type` error about missing asset file   | `.asset.php` not generated / file path mismatch              | Verify `dist/blocks/<name>/frontend.asset.php` exists    |

## Constants the build cares about

Defined in `includes/Plugin.php::define_constants()`:

- `ESSENTIAL_BLOCKS_FILE` — absolute path to `essential-blocks.php`
- `ESSENTIAL_BLOCKS_DIR_PATH` — plugin directory absolute path
- `ESSENTIAL_BLOCKS_DIR_URL` — plugin directory URL
- `ESSENTIAL_BLOCKS_BLOCK_DIR` — `'assets/blocks/'` (compiled block asset dir)
- `ESSENTIAL_BLOCKS_ADMIN_URL` — admin asset URL prefix
- `ESSENTIAL_BLOCKS_SITE_URL` — marketing site base URL (used in docs/demo links)
- `ESSENTIAL_BLOCKS_VERSION` — plugin version string
- `ESSENTIAL_BLOCKS_IS_PRO_ACTIVE` — bool (pro plugin detected?)

When investigating path issues, start by printing these constants in the
admin dashboard (they're localized to JS as `EssentialBlocksLocalize`).
