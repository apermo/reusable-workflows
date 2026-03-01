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

### `reusable-ci.yml`

PHP CI pipeline with configurable test matrix, PHPStan, and PHPCS.

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `php-versions` | string (JSON) | `["8.3", "8.4"]` | PHP versions to test |
| `php-version-coverage` | string | `8.4` | PHP version for coverage |
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
