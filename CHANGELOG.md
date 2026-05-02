# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.6.2] - 2026-05-02

### Fixed

- `reusable-conventional-commits.yml` — skip the default message that `git revert` generates
  (`Revert "<original subject>"`) the same way merge commits are skipped. Previously the
  validator rejected reverts and forced contributors to hand-edit the message into the
  conventional schema, which dropped the `Revert "` prefix that release-please and similar
  tools rely on to group reverts. Hand-written `revert: …` / `revert(scope): …` subjects are
  still validated against `allowed-types`. (#33)

## [0.6.1] - 2026-05-01

### Fixed

- `reusable-conventional-commits.yml` — accept the `!` breaking-change marker in commit subjects
  (e.g. `feat(api)!: drop deprecated input`). The Conventional Commits 1.0 spec defines `!` and a
  `BREAKING CHANGE:` footer as equivalent ways to mark a breaking change; the validator regex
  previously only accepted the footer form, rejecting spec-compliant subjects like the one
  intended for the 0.6.0 release commit (which had to be rewritten to drop the `!`).

## [0.6.0] - 2026-05-01

### Changed

- `reusable-stale.yml` — **breaking**: replace generic `days-before-stale` / `days-before-close` inputs with four
  per-type inputs so issues and PRs can have different timeouts. New inputs:
  `days-before-issue-stale` (default 30), `days-before-issue-close` (default 14),
  `days-before-pr-stale` (default 60), `days-before-pr-close` (default 21). PR defaults are roughly twice the issue
  grace period since an open PR represents in-flight work. Callers that pass the old input names must rename them.

## [0.5.1] - 2026-04-20

### Fixed

- `reusable-wp-e2e.yml`, `reusable-wp-visual-regression.yml`, `reusable-lhci.yml` — remove the `docker-config.js`
  sed-patch shipped in 0.4.1–0.4.3. The upstream advisory metadata that blocked Composer 2.8 resolution of
  PHPUnit 11 was corrected in
  [FriendsOfPHP/security-advisories#762](https://github.com/FriendsOfPHP/security-advisories/pull/762)
  (merged 2026-04-18), which is the feed Composer consults. The workaround is no longer needed; see
  [WordPress/gutenberg#77472](https://github.com/WordPress/gutenberg/pull/77472) for the matching upstream revert.

## [0.5.0] - 2026-04-19

### Added

- `reusable-plugin-check.yml` — WordPress Plugin Check runner wrapping [`wordpress/plugin-check-action@v1`](https://github.com/WordPress/plugin-check-action). Exposes the 11 most commonly tuned inputs. (#27)

### Changed

- `reusable-ci.yml`, `reusable-wp-integration.yml`, `reusable-wp-e2e.yml`, `reusable-wp-visual-regression.yml`, `reusable-wp-theme-ci.yml`, `reusable-lhci.yml`, `reusable-plugin-check.yml` — declare explicit `permissions: contents: read` at the workflow level. Defense-in-depth: the reusables now scope their own `GITHUB_TOKEN` instead of inheriting whatever the caller grants. (#28)

## [0.4.3] - 2026-04-18

### Fixed

- `reusable-wp-e2e.yml`, `reusable-wp-visual-regression.yml`, `reusable-lhci.yml` — patch the wp-env JS source (`docker-config.js`) instead of searching for static Dockerfiles. wp-env generates Dockerfiles at runtime under `~/.wp-env/` as `*.Dockerfile`, so the 0.4.1/0.4.2 find-and-sed approach never matched. (#24, [WordPress/gutenberg#77470](https://github.com/WordPress/gutenberg/issues/77470))

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
