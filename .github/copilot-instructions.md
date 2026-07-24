# Copilot Instructions for django-system-registry

## Quick Start

This is a reusable Django package for typed, registry-backed system settings.
Use `uv` for dependency management and `tox` as the canonical local validation
entry point. This repository currently targets Python 3.13 only.

```bash
uv sync --extra dev
```

`uv.lock` is committed. Update it when dependency metadata changes, and keep CI
compatible with `uv sync --frozen --extra dev`.

## Build, Test, and Lint Commands

### Setup and Packaging
```bash
# Install the development toolchain
uv sync --extra dev

# Build wheel and sdist artifacts
uv run python -m build
```

### Tox Entry Points
```bash
# Run the default locally available tox environments
uv run tox

# Run one environment explicitly
uv run tox -e py313
uv run tox -e lint
uv run tox -e mypy
uv run tox -e security

# Run a single test file or test function through tox
uv run tox -e py313 -- tests/test_models.py
uv run tox -e py313 -- tests/test_registry.py::test_get_registry_definitions_merges_provider_and_settings
```

### Direct Commands
```bash
# Run the full pytest suite
uv run pytest

# Run Ruff linting
uv run ruff check system_registry tests

# Run mypy with the repository config
uv run mypy system_registry tests

# Run security tooling directly
uv run bandit -q -r system_registry -x system_registry/migrations
uv run pip-audit
```

`tox` is the canonical entry point for local and CI checks. The configured
environments are `py313`, `lint`, `mypy`, and `security`, with optional `ruff`,
`bandit`, and `pip-audit` aliases for focused runs.

## High-Level Architecture

### Core Components

**Models** (`system_registry/models.py`)
- `SystemSetting` stores database overrides for typed registry-backed settings
- The model exposes read/write helpers that validate values against the
  registered definitions

**Registry and services** (`system_registry/registry.py`, `system_registry/services.py`)
- Definitions can come from Django settings or a dotted-path provider callable
- The service layer provides cached reads and writes for host applications

**Configuration and caching** (`system_registry/conf.py`, `system_registry/cache.py`)
- Package settings control cache prefixing, TTL behavior, and definition loading
- Cache invalidation keeps service-layer reads fresh after model changes

**Management commands** (`system_registry/management/`)
- `sync_system_settings` seeds or inspects the database from the active
  registry definitions

## Key Conventions

### Code Style
- Use `from __future__ import annotations` when forward references are needed
- Ruff is the linting tool; line length is 99 characters
- Migration files are exempt from the normal line-length and import-order rules

### Django Patterns
- Keep registry definitions host-owned; this package provides the framework and
  persistence layer
- Preserve the separation between raw model access and the cached service-layer
  helpers
- Keep the optional integration path with `django-editable-pages` indirect and
  settings-driven

### Versioning and Release Flow
- `pyproject.toml` drives release version selection, and the editable package
  metadata in `uv.lock` must stay aligned with that version
- Normal feature work should not bump the version manually
- Releases go through `.github/workflows/release.yml`, which creates a release
  bump PR, updates `pyproject.toml` plus the committed `uv.lock` metadata, then
  creates the tag and GitHub Release after merge
- The release flow is GitHub-only for now; do not add PyPI publishing steps

## Git Workflow

### Using Worktrees
- Create branch worktrees under the shared `../../worktrees/` directory (full
  path: `/home/bjorn/workspace/web_projects/worktrees/`).
- Do not work directly on `main`; create a branch worktree first:
  ```bash
  git worktree add ../../worktrees/django-system-registry-my-change -b my-change main
  cd ../../worktrees/django-system-registry-my-change
  ```
- Commit and push from that worktree branch as usual.
- After the branch has been pushed and you no longer need the local checkout, or
  after the related PR/release has been published, remove the worktree and then
  delete the local branch:
  ```bash
  cd /home/bjorn/workspace/web_projects/pkgs/django-system-registry
  git worktree remove ../../worktrees/django-system-registry-my-change
  git branch -d my-change
  ```
- If the worktree has uncommitted local-only files, clean or intentionally
  discard them before removing the worktree. Use `git worktree remove --force`
  only when you mean to throw that local state away.

## Important Notes

- Tests use `tests.settings`
- `uv.lock` should stay in sync with dependency metadata changes
- Keep workflow path filters aligned with this repo's package path
