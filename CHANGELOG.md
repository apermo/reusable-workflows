# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.2] - 2026-04-18

### Fixed

- `reusable-wp-e2e.yml`, `reusable-wp-visual-regression.yml`, `reusable-lhci.yml` — use `composer config audit.block-insecure false` to actually disable Composer 2.8 resolution-time advisory block (`--no-audit` in 0.4.1 only skipped post-install audit) (#24)

## [0.4.1] - 2026-04-18

### Fixed

- `reusable-wp-e2e.yml`, `reusable-wp-visual-regression.yml`, `reusable-lhci.yml` — patch wp-env Dockerfile to disable Composer 2.8 advisory blocking on PHPUnit install ([WordPress/gutenberg#77470](https://github.com/WordPress/gutenberg/issues/77470))

## [0.4.0] - 2026-04-08

### Added

- `reusable-wp-e2e.yml` — optional `@axe-core/playwright` install via `a11y` input (#16)
- `reusable-lhci.yml` — Lighthouse CI workflow with configurable score thresholds (#17)
- `reusable-wp-visual-regression.yml` — visual regression testing with Playwright (#18)

### Fixed

- `reusable-pr-validation.yml` — reject CHANGELOG versions that already have a git tag (#13)
- `reusable-pr-validation.yml`, `reusable-conventional-commits.yml` — remove `name:` keys so branch protection matches the job ID (#12)

### Changed

- Document actionlint CI integration in README and CLAUDE.md (#8)
- Add summary job to matrix-based workflows (`ci`, `integration`, `e2e`, `visual-regression`) for stable branch protection checks (#11)

## [0.3.0] - 2026-04-06

### Added

- `reusable-wp-e2e.yml` — optional Mailpit mail catcher via `mailpit` input

### Changed

- `reusable-wp-e2e.yml` — replaced PHP built-in server and DDEV with wp-env (#14)

### Removed

- `reusable-wp-e2e.yml` — removed `php-version`, `project-mode`, and `use-ddev` inputs

## [0.2.0] - 2026-03-21

### Added

- WP theme CI workflow with ESLint, Stylelint, and version consistency check (#2)
- DDEV-based E2E testing via `use-ddev` input in WP E2E workflow (#6)
- Labeled WP versions support in integration matrix via `{version, name}` objects (#3)

### Changed

- Rename integration workflow from "Reusable WP Integration" to "WP Integration"

## [0.1.1] - 2026-03-19

### Fixed

- Integration workflow: generate `coverage.xml` for coverage PHP version

## [0.1.0] - 2026-03-01

### Added

- Reusable CI workflow with configurable PHP versions and coverage
- Reusable release workflow with CHANGELOG-driven version detection
- Reusable PR validation workflow for CHANGELOG entries
- Reusable conventional commits validation workflow
- Reusable prerelease workflow for release branches
- Reusable stale issues/PRs workflow
- Reusable WordPress.org SVN deploy workflow
- Reusable WP integration test workflow with PHP × WP × multisite matrix
- Reusable WP E2E test workflow with Playwright
- Self-referencing CI as built-in integration test

### Fixed

- Stale and prerelease caller workflows missing permissions (caused startup_failure)

### Changed

- WP integration workflow uses `wp-phpunit/wp-phpunit` Composer package instead of SVN checkout
- Upgrade `actions/stale` from v9 to v10
