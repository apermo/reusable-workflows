# Code Review Style Guide

## Project Context

Shared GitHub Actions reusable workflows for all apermo repositories. Each workflow is parameterized via
`workflow_call` inputs and can be called from any repo.

## Code Style

- Flag inline comments that merely restate what the code does instead of explaining intent or reasoning.
- Flag commented-out code.
- Flag new code that duplicates existing functionality in the repository.

## GitHub Actions

- All reusable workflows must use `workflow_call` trigger.
- Inputs should have sensible defaults matching the apermo standard.
- Secrets must be passed explicitly (never inherited).
- Action versions must be pinned to major: `@v1`, `@v2`, `@v4`, `@v5`.
- Flag workflows missing `fail-fast: false` on test matrices.

## File Operations

- Flag files that appear to be deleted and re-added as new files instead of being moved/renamed (losing git
  history).

## Build & Packaging

- Flag newly added files or directories that are missing from build/packaging configs (`.gitattributes`, `Makefile`,
  CI workflows, etc.).

## Documentation

- If a change affects user-facing behavior, flag missing updates to README, CHANGELOG, or inline docblocks.

## Commits

- This project uses Conventional Commits with a 50-char subject / 72-char body limit.
- Each commit should address a single concern.
