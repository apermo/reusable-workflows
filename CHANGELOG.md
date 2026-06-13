# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Fixed

- `reusable-pr-validation.yml`, `reusable-release.yml`, and `reusable-prerelease.yml` now check
  `apermo/reusable-workflows` out into a subdir and reference the `extract-changelog-version` action
  locally from there, instead of a bare `./` path. The relative path resolved against the
  *consumer's* checkout (which has no `.github/actions/`), so every external caller failed with
  "Can't find action.yml"; a same-repo `owner/repo/...@ref` ref doesn't resolve from a reusable
  workflow either. Regression from 0.11.0. (#43)

### Changed

- The action checkout in `reusable-pr-validation.yml`, `reusable-release.yml`, and
  `reusable-prerelease.yml` is pinned to `${{ github.job_workflow_sha }}` (the reusable workflow's
  own commit) instead of `main`, so consumers that pin the workflow to a version get the matching
  action, and a PR's self-test exercises the PR's action rather than `main`'s. The checkout also
  sets `persist-credentials: false`. (#43)

## [0.11.0] - 2026-06-13

### Added

- `extract-changelog-version` composite action — single source of truth for selecting and
  validating the release version from `CHANGELOG.md` (first non-`Unreleased` heading, strict
  SemVer 2.0.0 + ` - YYYY-MM-DD` date). Emits `result`/`version`/`base_version`/`tag`/`prerelease`.
  (#43)

### Changed

- `reusable-pr-validation.yml`, `reusable-release.yml`, and `reusable-prerelease.yml` now extract
  the CHANGELOG version via the shared `extract-changelog-version` action instead of three
  divergent inline implementations, so their version selection can no longer diverge (the
  `## [Yanked]`-over-an-untagged-version case now resolves consistently to "no release"). (#43)
- **BREAKING:** `reusable-prerelease.yml` now requires a strict `## [X.Y.Z] - YYYY-MM-DD` heading;
  it previously accepted an undated `## [X.Y.Z]`. A prerelease cut from a release branch whose top
  entry is undated will now fail — add the date, or open an issue to relax the prerelease path.
  Relatedly, `reusable-release.yml` now fails loudly on a malformed top heading instead of silently
  skipping (pr-validation should catch it pre-merge). (#43)
- **BREAKING:** `reusable-conventional-commits.yml` — default `max-length` lowered from
  `72` to `50` to match the org-wide commit-message standard (50-char subject, 72-char
  body wrap). Caller repos that relied on the implicit 72 limit and have subjects longer
  than 50 chars will start failing; pass `with: max-length: 72` to keep the previous
  behavior. The `template-wordpress` `.husky/commit-msg` `MAX` should be lowered to 50 in
  lock-step (tracked separately). (#38)

## [0.10.0] - 2026-06-08

### Changed

- Bump GitHub Actions to their first Node 24 runtimes across all reusable
  workflows: `actions/checkout` v4→v5, `actions/github-script` v7→v8,
  `actions/setup-node` v4→v5, `actions/upload-artifact` v4→v5, and
  `actions/cache` v4→v5. GitHub forces JavaScript actions off the
  deprecated Node 20 runtime starting 2026-06-16.

## [0.9.0] - 2026-06-07

### Added

- `reusable-wp-e2e.yml` — `build-command` input (default `build`). Compiles
  Gutenberg blocks/assets via `npm run <build-command>` after `npm ci` and before
  wp-env starts, so block E2E specs see compiled blocks. Skipped when the named
  npm script is absent, so non-block plugins/themes are unaffected. (#44)
- `reusable-wporg-deploy.yml` — `node-version` (default `22`) and `build-command`
  (default `build`) inputs. Runs `npm ci && npm run <build-command>` before deploy
  so the shipped plugin includes a compiled `build/`. Skipped when there is no
  `package.json` or the named script is absent. (#44)

### Fixed

- `reusable-plugin-check.yml` — no longer fails on repos that commit a `.wp-env.json`
  (required by `reusable-wp-e2e.yml`). `plugin-check-action` only provisions its own
  wp-env when none exists; a repo one made it skip setup and die with "Environment not
  initialized". The repo `.wp-env.json`/`.wp-env.override.json` are now moved aside for
  the run and restored afterwards. (#44)

## [0.8.0] - 2026-06-07

### Changed

- `reusable-pr-validation.yml` — decouple merging from releasing. A missing,
  `## [Unreleased]`, or already-tagged top heading now **passes** (merge without
  release) instead of hard-failing; only a malformed version heading (looks like a
  version but isn't valid SemVer, or missing the ` - YYYY-MM-DD` date) fails. A new
  untagged `## [X.Y.Z] - YYYY-MM-DD` still triggers a release on merge.
  `reusable-release.yml` is unchanged — its existing tag-existence skip already gates
  releases on a version bump. (#39)

## [0.7.0] - 2026-05-24

### Added

- `reusable-release.yml` — optional `release-token` secret. Defaults to
  `GITHUB_TOKEN`; pass a PAT or GitHub App installation token so downstream
  `release` / `push: tags` workflows on the calling repo fire on the created
  release (GitHub's `GITHUB_TOKEN` loop-prevention suppresses them otherwise).
  Backwards-compatible — callers that don't pass the secret are unaffected. (#35)

### Changed

- `reusable-stale.yml` — rebalance default day thresholds and broaden exempt labels.
  - Issues now go stale after 60 days (was 30) and close 30 days later (was 14)
    — total 90-day idle window before closure (was 44).
  - PRs go stale faster (30 days, was 60) but get a longer grace period before
    closing (30 days, was 21) — total 60-day window before closure (was 81).
    Shifts the bias from "let PRs linger forever" to "ping authors sooner, then
    give them a full month to respond".
  - `exempt-labels` default extended with `approved`, `refined`, `keep` so
    triaged items don't get culled.

  All four day-count inputs and the `exempt-labels` input remain overridable
  per-repo via `with:` if a project needs different settings.

### Security

- `reusable-ci.yml`, `reusable-wp-e2e.yml`, `reusable-wp-integration.yml`,
  `reusable-wp-theme-ci.yml`, `reusable-wp-visual-regression.yml`,
  `reusable-wporg-deploy.yml` — pin `tools: composer:^2.9.8` on every
  `shivammathur/setup-php@v2` step. Defends against future regressions of
  [GHSA-f9f8-rm49-7jv2](https://github.com/advisories/GHSA-f9f8-rm49-7jv2)
  (Composer ≤ 2.9.7 leaked GitHub Actions tokens to stderr when validating
  new-format tokens containing hyphens). No functional change today —
  `setup-php@v2` already installs latest stable Composer; the pin makes the
  minimum explicit. (#36)

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

[0.11.0]: https://github.com/apermo/reusable-workflows/compare/v0.10.0...v0.11.0
[0.10.0]: https://github.com/apermo/reusable-workflows/compare/v0.9.0...v0.10.0
[0.9.0]: https://github.com/apermo/reusable-workflows/compare/v0.8.0...v0.9.0
[0.8.0]: https://github.com/apermo/reusable-workflows/compare/v0.7.0...v0.8.0
[0.7.0]: https://github.com/apermo/reusable-workflows/compare/v0.6.2...v0.7.0
