# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - Unreleased

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

### Changed

- WP integration workflow uses `wp-phpunit/wp-phpunit` Composer package instead of SVN checkout
