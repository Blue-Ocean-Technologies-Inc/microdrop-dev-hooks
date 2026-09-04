# microdrop-dev-hooks

Shared pre-commit hooks and lint baselines for the MicroDrop organisation's
repos (`Microdrop`, the heater/magnet/fluorescence plugins, the
`pixi-microdrop` launcher, …). This repo is the single home for the
convention-enforcement tooling described in each consumer's `AGENTS.md` —
consumers reference it by tag, so a fix or a new hook lands everywhere with
one version bump instead of being copy-pasted around.

## Hooks

### `stamp-import-sections`

Labels the module-level import block with the section headers required by
AGENTS.md's "Import Ordering" convention (Standard library / Third-party /
Enthought / Microdrop package / Microdrop style / Microdrop utils / Local /
Logger). Ruff's isort already sorts imports into the right groups via
`ruff.toml`'s `[lint.isort]` config; this hook adds the comment header ruff
has no mechanism to emit. It reads the consumer's own `ruff.toml` for
`section-order` and `[lint.isort.sections]`, plus `known-first-party` for
repos whose first-party package doesn't live at the repo root (e.g. a
plugin repo). Idempotent — run it after the ruff hooks.

### `forbid-scratch-files`

Rejects commits that add scratch/artifact paths (`.task-report.md`,
`.superpowers/`, `__pycache__/`, `.pixi/`) that should never be tracked.

## Using these hooks in a consumer repo

Add to the consumer's `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/Blue-Ocean-Technologies-Inc/microdrop-dev-hooks
    rev: v0.1.0
    hooks:
      - id: stamp-import-sections
      - id: forbid-scratch-files
```

`stamp-import-sections` requires a `ruff.toml` with `[lint.isort]` at the
consumer's repo root (or a parent of the file being stamped) — it walks
upward from each file to find one, the same way it would from a submodule
checkout.

## Shared lint baseline

`ruff-base.toml` and `copyright-header.txt` (added once the shared baseline
lands) are not consumed automatically — pre-commit hooks can't `extend` a
config file across repos the way ruff's own `extend` does within one repo.
A consumer copies them in and either points its own `ruff.toml` at
`extend = "ruff-base.toml"` (ruff *does* support that within a single
checkout, once the file is local) or diffs against them by hand when this
repo's copy changes.

## Releases

Tags are `vX.Y.Z`, cut by `commitizen` from Conventional Commit messages on
every push to `main` (`.github/workflows/release.yml`). A tag *is* the hook
`rev` consumers pin — bump the version here, then bump `rev` in each
consumer's `.pre-commit-config.yaml` to pick it up.
