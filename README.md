# Reusable Workflows

[![PR Validation](https://github.com/apermo/reusable-workflows/actions/workflows/pr-validation.yml/badge.svg)](https://github.com/apermo/reusable-workflows/actions/workflows/pr-validation.yml)
[![Release](https://github.com/apermo/reusable-workflows/actions/workflows/release.yml/badge.svg)](https://github.com/apermo/reusable-workflows/actions/workflows/release.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Shared GitHub Actions reusable workflows for apermo repositories.

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

### `reusable-wp-visual-regression.yml`

Playwright-based visual regression testing for WordPress. Captures screenshots and compares them against
committed baselines using Playwright's built-in `toHaveScreenshot()` API.

Caller repos must include a `.wp-env.json` in their root. The consuming repo's `playwright.config.ts` should
read the `VRT_THRESHOLD`, `VRT_VIEWPORTS`, and `VRT_COLOR_SCHEMES` environment variables to configure test
behavior. Baseline screenshots are stored in the repo (consider Git LFS for large sets).

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `node-version` | string | `"22"` | Node.js version |
| `wp-versions` | string (JSON) | `["latest"]` | WP versions (`"latest"`, `"6.7"`) |
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
