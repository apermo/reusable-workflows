# Reusable Workflows

## Project Overview

Shared GitHub Actions reusable workflows for all apermo repositories. Each workflow is parameterized via `workflow_call` inputs and can be called from any repo.

## Architecture

All reusable workflows live in `.github/workflows/` with the `reusable-` prefix. This repo's own CI calls these workflows via relative path (`./.github/workflows/reusable-*.yml`), serving as a built-in integration test.

### Workflow inventory

| File | Purpose |
|------|---------|
| `reusable-ci.yml` | PHP CI (test matrix, PHPStan, PHPCS) |
| `reusable-wp-integration.yml` | WP integration tests (real WP + MySQL matrix) |
| `reusable-wp-e2e.yml` | WP E2E tests (Playwright + PHP built-in server) |
| `reusable-release.yml` | CHANGELOG-driven release creation |
| `reusable-pr-validation.yml` | CHANGELOG entry validation |
| `reusable-conventional-commits.yml` | Commit message format validation |
| `reusable-prerelease.yml` | Manual prerelease from release branches |
| `reusable-stale.yml` | Auto-close stale issues/PRs |
| `reusable-wp-theme-ci.yml` | WP theme CI (ESLint, Stylelint, version check) |
| `reusable-wporg-deploy.yml` | Deploy plugin/theme to WordPress.org SVN |

### Self-referencing CI

The repo's own workflows (`pr-validation.yml`, `release.yml`, etc.) call the reusable versions via relative path. This validates that the reusable workflows work correctly on every change.

## Conventions

- All reusable workflows use `workflow_call` trigger
- Inputs have sensible defaults matching the apermo standard
- Secrets are passed explicitly (never inherited)
- Action versions are pinned to major: `@v4`, `@v5`, `@v7`, `@v9`
- CHANGELOG.md follows Keep a Changelog format
- Releases are automated via the reusable release workflow
