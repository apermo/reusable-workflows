# Reusable Workflows

[![PR Validation](https://github.com/apermo/reusable-workflows/actions/workflows/pr-validation.yml/badge.svg)](https://github.com/apermo/reusable-workflows/actions/workflows/pr-validation.yml)
[![Release](https://github.com/apermo/reusable-workflows/actions/workflows/release.yml/badge.svg)](https://github.com/apermo/reusable-workflows/actions/workflows/release.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Shared GitHub Actions reusable workflows for apermo repositories.

All workflow changes in this repo are linted automatically with
[actionlint](https://github.com/rhysd/actionlint) via
[reviewdog/action-actionlint](https://github.com/reviewdog/action-actionlint) on every pull request.
For local pre-push validation, install actionlint with `brew install actionlint` and run it from the
repo root.

## Branch protection

Matrix-based workflows expose a static summary check per workflow. Target these in branch protection
rather than the individual matrix jobs — their names evaluate cleanly when skipped and don't change when
the matrix shape changes.

| Workflow | Summary check name |
|----------|--------------------|
| `reusable-ci.yml` | `CI` |
| `reusable-wp-integration.yml` | `Integration` |
| `reusable-wp-e2e.yml` | `E2E` |
| `reusable-wp-visual-regression.yml` | `Visual Regression` |

The check path is `<caller-workflow-name> / <summary-job-id> / <summary-name>`, e.g. `CI / ci-summary / CI`.

## Workflows

### `reusable-wp-integration.yml`

WordPress integration test pipeline with real WP + MySQL. Supports PHP × WP version × multisite matrix.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `php-versions` | string (JSON) | `["8.3", "8.4"]` | PHP versions to test |
| `wp-versions` | string (JSON) | `["latest"]` | WP versions (`"latest"`, `"6.7"`, `"beta"`) |
| `multisite` | string | `"none"` | `"none"`, `"both"`, or `"only"` |
| `project-mode` | string | `"plugin"` | `"plugin"` or `"theme"` |
| `test-command` | string | `"test:integration"` | Composer script to run |
| `php-version-coverage` | string | `""` | PHP version for coverage (empty = none) |

| Secret | Required | Description |
|--------|----------|-------------|
| `codecov-token` | No | Codecov upload token |

```yaml
jobs:
  integration:
    uses: apermo/reusable-workflows/.github/workflows/reusable-wp-integration.yml@main
    with:
      php-versions: '["8.1", "8.4"]'
      wp-versions: '["latest", "6.6"]'
      multisite: 'both'
```

### `reusable-wp-e2e.yml`

WordPress E2E test pipeline with Playwright. Runs Chromium against a wp-env Docker environment.

The environment is available at `http://localhost:8888` with the default WordPress credentials (`admin` / `password`).

Caller repos must include a `.wp-env.json` in their root (see
[wp-env docs](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-env/)). For example, to test the
plugin in the current directory:

```json
{
  "plugins": ["."]
}
```

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `node-version` | string | `"22"` | Node.js version |
| `wp-versions` | string (JSON) | `["latest"]` | WP versions (`"latest"`, `"6.7"`) |
| `multisite` | string | `"none"` | `"none"`, `"both"`, or `"only"` |
| `mailpit` | boolean | `false` | Run Mailpit mail catcher (SMTP `:1025`, API `:8025`) |
| `a11y` | boolean | `false` | Install `@axe-core/playwright` for accessibility testing |
| `build-command` | string | `"build"` | npm script that compiles assets/blocks before wp-env starts (skipped if absent) |

Block-based plugins/themes are built before wp-env starts (default `npm run build`) so compiled blocks
are present when specs run. The step is skipped when no matching npm script exists.

Recommended `wp-versions` setup: test against `"latest"` and the minimum supported WP version (e.g. `'["latest", "6.4"]'`).

```yaml
jobs:
  e2e:
    uses: apermo/reusable-workflows/.github/workflows/reusable-wp-e2e.yml@main
    with:
      wp-versions: '["latest", "6.4"]'
```

### `reusable-lhci.yml`

Lighthouse CI audit pipeline. Runs accessibility, performance, SEO, and best-practices audits against one or more URLs.

The caller workflow is responsible for having a running site before invoking this workflow, unless `setup-wp-env`
is enabled. When `setup-wp-env` is `true`, the workflow starts a `wp-env` Docker environment at
`http://localhost:8888`.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `urls` | string (JSON) | `["http://localhost:8888"]` | URLs to audit |
| `node-version` | string | `"22"` | Node.js version |
| `runs` | number | `3` | Lighthouse runs per URL (median is used) |
| `setup-wp-env` | boolean | `false` | Start wp-env before running audits |
| `a11y-threshold` | number | `90` | Minimum accessibility score (0 to skip) |
| `performance-threshold` | number | `0` | Minimum performance score (0 to skip) |
| `seo-threshold` | number | `0` | Minimum SEO score (0 to skip) |
| `best-practices-threshold` | number | `0` | Minimum best practices score (0 to skip) |
| `config-path` | string | `""` | Path to `.lighthouserc.js` (overrides thresholds) |
| `artifact-name` | string | `"lighthouse-report"` | Name for uploaded Lighthouse report artifact |

```yaml
jobs:
  lighthouse:
    uses: apermo/reusable-workflows/.github/workflows/reusable-lhci.yml@main
    with:
      urls: '["http://localhost:8888", "http://localhost:8888/sample-page/"]'
      setup-wp-env: true
      a11y-threshold: 90
      performance-threshold: 50
```

### `reusable-wp-visual-regression.yml`

Playwright-based visual regression testing for WordPress. Captures screenshots and compares them against
committed baselines using Playwright's built-in `toHaveScreenshot()` API.

Caller repos must include a `.wp-env.json` in their root. The workflow starts a `wp-env` Docker environment at
`http://localhost:8888`. The consuming repo's `playwright.config.ts` should read the `VRT_THRESHOLD`,
`VRT_VIEWPORTS`, and `VRT_COLOR_SCHEMES` environment variables to configure test behavior. Baseline screenshots
are stored in the repo (consider Git LFS for large sets).

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `node-version` | string | `"22"` | Node.js version |
| `wp-versions` | string (JSON) | `["latest"]` | WP versions (`"latest"`, `"6.7"`, `"beta"`) |
| `multisite` | string | `"none"` | `"none"`, `"both"`, or `"only"` |
| `mailpit` | boolean | `false` | Run Mailpit mail catcher (SMTP `:1025`, API `:8025`) |
| `update-snapshots` | boolean | `false` | Regenerate baseline screenshots |
| `threshold` | string | `"0.2"` | Pixel diff tolerance (`maxDiffPixelRatio`) |
| `viewports` | string (JSON) | `["desktop", "mobile"]` | Viewport presets |
| `color-schemes` | string (JSON) | `["light"]` | Color scheme presets |

To accept a visual change, update the baseline by re-running the workflow with `update-snapshots: true`, then
download and commit the updated snapshots from the workflow artifacts.

```yaml
jobs:
  visual-regression:
    uses: apermo/reusable-workflows/.github/workflows/reusable-wp-visual-regression.yml@main
    with:
      wp-versions: '["latest"]'
      viewports: '["desktop", "mobile"]'
      color-schemes: '["light", "dark"]'
```

### `reusable-plugin-check.yml`

WordPress [Plugin Check](https://wordpress.org/plugins/plugin-check/) runner. Wraps
[`wordpress/plugin-check-action@v1`](https://github.com/WordPress/plugin-check-action) and reports guideline
violations as GitHub file annotations. Intended for plugins targeting the wordpress.org plugin directory.

A repo-provided `.wp-env.json` or `.wp-env.override.json` (required by `reusable-wp-e2e.yml`) is moved aside for
the run and restored afterwards, so `plugin-check-action` provisions its own environment instead of failing on it.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `wp-version` | string | `"latest"` | WordPress version (`"latest"` or `"trunk"`; upstream only special-cases `trunk`) |
| `build-dir` | string | `"./"` | Plugin build directory |
| `checks` | string | `""` | Only run specific checks (comma-separated) |
| `exclude-checks` | string | `""` | Checks to exclude (comma-separated) |
| `categories` | string | `""` | Limit checks to specific categories (comma-separated) |
| `exclude-files` | string | `""` | Files to exclude (comma-separated) |
| `exclude-directories` | string | `""` | Directories to exclude (comma-separated) |
| `ignore-codes` | string | `""` | Error codes to ignore (comma-separated) |
| `ignore-warnings` | boolean | `false` | Ignore warnings |
| `ignore-errors` | boolean | `false` | Ignore errors |
| `include-experimental` | boolean | `false` | Include experimental checks |

```yaml
jobs:
  plugin-check:
    uses: apermo/reusable-workflows/.github/workflows/reusable-plugin-check.yml@main
    with:
      wp-version: latest
```

### `reusable-wporg-deploy.yml`

Deploys a plugin or theme to the WordPress.org SVN repository. Wraps
[`10up/action-wordpress-plugin-deploy`](https://github.com/10up/action-wordpress-plugin-deploy). When a
`package.json` is present it runs `npm ci && npm run <build-command>` before deploy, so block-based plugins ship a
compiled `build/`. The build step is skipped when there is no `package.json` or the named npm script is absent.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `slug` | string | repo name | WordPress.org plugin/theme slug |
| `assets-dir` | string | `".wordpress-org"` | Path to WordPress.org assets (banners, icons, screenshots) |
| `build-dir` | string | `""` | Path to build directory (`false` to skip build copy) |
| `node-version` | string | `"22"` | Node.js version for the asset build |
| `build-command` | string | `"build"` | npm script that compiles assets/blocks before deploy (skipped if absent) |
| `generate-zip` | boolean | `false` | Generate a zip file as build artifact |
| `dry-run` | boolean | `false` | Run without committing to SVN |

| Secret | Required | Description |
|--------|----------|-------------|
| `svn-username` | Yes | WordPress.org SVN username |
| `svn-password` | Yes | WordPress.org SVN password |

```yaml
jobs:
  deploy:
    uses: apermo/reusable-workflows/.github/workflows/reusable-wporg-deploy.yml@main
    with:
      slug: my-plugin
    secrets:
      svn-username: ${{ secrets.WPORG_SVN_USERNAME }}
      svn-password: ${{ secrets.WPORG_SVN_PASSWORD }}
```

### `reusable-ci.yml`

PHP CI pipeline with configurable test matrix, PHPStan, and PHPCS.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `php-versions` | string (JSON) | `["8.3", "8.4"]` | PHP versions to test |
| `php-version-coverage` | string | `8.4` | PHP version for coverage |
| `test-command` | string | `"test"` | Composer script to run |
| `run-phpstan` | boolean | `true` | Run PHPStan analysis |
| `run-phpcs` | boolean | `true` | Run PHPCS check |
| `codecov-upload` | boolean | `false` | Upload to Codecov |

| Secret | Required | Description |
|--------|----------|-------------|
| `codecov-token` | No | Codecov upload token |

```yaml
jobs:
  ci:
    uses: apermo/reusable-workflows/.github/workflows/reusable-ci.yml@main
    with:
      php-versions: '["8.1", "8.2", "8.3", "8.4"]'
      codecov-upload: true
    secrets:
      codecov-token: ${{ secrets.CODECOV_TOKEN }}
```

### `reusable-release.yml`

CHANGELOG-driven release automation. Extracts version from CHANGELOG.md, creates GitHub releases, cleans up old prereleases.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `changelog-path` | string | `CHANGELOG.md` | Path to changelog |

| Secret | Required | Description |
|--------|----------|-------------|
| `release-token` | No | Override for `GITHUB_TOKEN`. Pass a PAT or GitHub App installation token when downstream `release` / `push: tags` workflows on the calling repo need to fire on the created release (GitHub's `GITHUB_TOKEN` loop-prevention suppresses them otherwise). |

```yaml
jobs:
  release:
    uses: apermo/reusable-workflows/.github/workflows/reusable-release.yml@main
```

### `reusable-pr-validation.yml`

Validates the top `CHANGELOG.md` version heading on pull requests. Merging is decoupled
from releasing: a PR only needs a valid heading when it intends to release.

| Top heading | Result | Behavior |
|-------------|--------|----------|
| Missing, `## [Unreleased]`, or non-version label | `no_version` | **Passes** — merge without release |
| Valid `## [X.Y.Z] - YYYY-MM-DD`, already tagged | `already_released` | **Passes** — merge without release |
| Valid `## [X.Y.Z] - YYYY-MM-DD`, untagged | `ok` | **Passes** — releases on merge |
| Looks like a version but isn't valid SemVer / missing date | `malformed` | **Fails** |

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `changelog-path` | string | `CHANGELOG.md` | Path to changelog |

```yaml
jobs:
  pr-validation:
    uses: apermo/reusable-workflows/.github/workflows/reusable-pr-validation.yml@main
```

**Migration note:** PRs without a version bump now merge without releasing. Add a new
`## [X.Y.Z] - YYYY-MM-DD` heading to cut a release on merge; the only hard failure is a
malformed version heading.

### `reusable-conventional-commits.yml`

Validates commit messages follow the Conventional Commits specification.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `max-length` | number | `50` | Max subject line length |
| `allowed-types` | string | `feat\|fix\|docs\|style\|refactor\|test\|chore\|perf\|ci\|build` | Allowed types |

```yaml
jobs:
  conventional-commits:
    uses: apermo/reusable-workflows/.github/workflows/reusable-conventional-commits.yml@main
```

### `reusable-prerelease.yml`

Manual prerelease creation from `release/*` branches.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `changelog-path` | string | `CHANGELOG.md` | Path to changelog |

```yaml
jobs:
  prerelease:
    uses: apermo/reusable-workflows/.github/workflows/reusable-prerelease.yml@main
```

### `reusable-stale.yml`

Auto-closes stale issues and PRs.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `days-before-issue-stale` | number | `60` | Days of inactivity before marking an issue as stale |
| `days-before-issue-close` | number | `30` | Days after stale before closing an issue |
| `days-before-pr-stale` | number | `30` | Days of inactivity before marking a PR as stale |
| `days-before-pr-close` | number | `30` | Days after stale before closing a PR |
| `exempt-labels` | string | `pinned,security,in-progress,approved,refined,keep` | Exempt labels |

```yaml
jobs:
  stale:
    uses: apermo/reusable-workflows/.github/workflows/reusable-stale.yml@main
```

## Composite actions

### `extract-changelog-version`

Selects and validates the release version from a Keep-a-Changelog `CHANGELOG.md` (first
non-`Unreleased` `## [...]` heading, strict SemVer 2.0.0 + ` - YYYY-MM-DD` date). It is the single
source of truth used by `reusable-pr-validation.yml`, `reusable-release.yml`, and
`reusable-prerelease.yml` so their version selection cannot diverge. It is a pure classifier — it
never fails the job; callers apply their own policy to the `result` output.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `changelog-path` | string | `"CHANGELOG.md"` | Path to the CHANGELOG file |

| Output | Description |
|--------|-------------|
| `result` | `no_version` \| `malformed` \| `already_released` \| `ok` |
| `version` | Full version (e.g. `1.2.3` or `1.2.3-beta.1`); empty unless `ok`/`already_released` |
| `base_version` | Version without prerelease/build suffix (e.g. `1.2.3`); empty unless `ok`/`already_released` |
| `tag` | Release tag (`v` + version); empty unless `ok`/`already_released` |
| `prerelease` | `true` when the version carries a prerelease suffix |

The reusable workflows in this repo reference it by relative path
(`./.github/actions/extract-changelog-version`), since GitHub resolves that against this repo and
rejects a same-repo `owner/repo/...@ref` reference from a reusable workflow. An external repo using
the action **directly** references the full path with a ref, as below. Either way, check out with
`fetch-depth: 0` so the `already_released` tag check works:

```yaml
steps:
  - uses: actions/checkout@v5
    with:
      fetch-depth: 0
  - id: version
    uses: apermo/reusable-workflows/.github/actions/extract-changelog-version@main
  # branch on steps.version.outputs.result
```

## License

MIT — see [LICENSE](LICENSE) for details.
