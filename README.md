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

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `wp-version` | string | `"latest"` | WordPress version to test against |
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

```yaml
jobs:
  release:
    uses: apermo/reusable-workflows/.github/workflows/reusable-release.yml@main
```

### `reusable-pr-validation.yml`

Validates CHANGELOG entries on pull requests.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `changelog-path` | string | `CHANGELOG.md` | Path to changelog |

```yaml
jobs:
  pr-validation:
    uses: apermo/reusable-workflows/.github/workflows/reusable-pr-validation.yml@main
```

### `reusable-conventional-commits.yml`

Validates commit messages follow the Conventional Commits specification.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `max-length` | number | `72` | Max subject line length |
| `allowed-types` | string | `feat\|fix\|docs\|style\|refactor\|test\|chore\|perf` | Allowed types |

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
| `days-before-stale` | number | `30` | Days before marking stale |
| `days-before-close` | number | `14` | Days before closing |
| `exempt-labels` | string | `pinned,security,in-progress` | Exempt labels |

```yaml
jobs:
  stale:
    uses: apermo/reusable-workflows/.github/workflows/reusable-stale.yml@main
```

## License

MIT — see [LICENSE](LICENSE) for details.
