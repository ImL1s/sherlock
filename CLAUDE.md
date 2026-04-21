# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Sherlock is a Python CLI that hunts social media accounts by username across 400+ sites (OSINT / reconnaissance). Python `^3.9`, managed with Poetry.

## Layout

- `sherlock_project/` — the installable package (NOT `sherlock/` — that's just the CLI entry-point name).
  - `sherlock.py` — `main()` is exposed as the `sherlock` console script.
  - `sites.py`, `result.py`, `notify.py` — core modules.
  - `resources/data.json` — the site manifest (JSON-schema-validated).
  - `resources/data.schema.json` — the schema.
- `tests/` — pytest suite.
- `devel/` — maintenance scripts (e.g., `site-list.py`). CI runs these; don't run them by hand.
- `docs/README.md` — the main README. `docs/pyproject/README.md` is the shorter variant used on PyPI.
- `.github/workflows/` — regression, modified-target validation, and site-list auto-sync.

## Commands

- `tox -e lint` — `ruff check` (the only style/lint gate).
- `tox -e offline` — `pytest -v -m "not online"` (fast, no network).
- `tox` — full matrix across py3.10–3.13 plus lint.
- `poetry run pytest` — respects `pytest.ini`.

No formatter is configured. There is no `ruff format` — only `ruff check`. Don't add Black / isort / `ruff format` without an RFC.

## Testing quirks

- `pytest.ini` pins `--strict-markers` and `-m "not validate_targets"` by default. `validate_targets` tests hit live social networks across the whole manifest — don't enable them locally; CI validates only the modified sites on PRs via `.github/workflows/validate_modified_targets.yml`.
- Registered markers: `online`, `validate_targets`, `validate_targets_fp`, `validate_targets_fn`. A new marker must be added to `pytest.ini` or strict-markers will fail the run.
- Fast local check for data.json edits: `poetry run pytest tests/test_manifest.py::test_validate_manifest_against_local_schema` (also wrapped as `/validate-data`).

## Data manifest workflow

- Site entries live in `sherlock_project/resources/data.json` and must conform to `resources/data.schema.json`.
- **Do not run `python devel/site-list.py` manually.** `.github/workflows/update-site-list.yml` regenerates and pushes the docs site automatically on merge to master.
- PRs that touch `data.json` auto-trigger per-site validation of only the modified entries — no need to run the full `validate_targets` suite locally.

## Repo quirks

- `poetry.lock` is intentionally gitignored (this is an end-user CLI; pinning is undesired).
- `*.txt`, `*.csv`, `*.xlsx` are gitignored — Sherlock writes result output to these formats. Don't commit sample outputs.
- Version is dynamic via `[tool.poetry-version-plugin] source = "init"` (read from `sherlock_project/__init__.py`). Don't hard-code the version anywhere else.

## CI / ownership

CI runs on push & PR to `master` and `release/**`: lint, test matrix (py3.10–3.14 on ubuntu/windows/macos), Docker build, modified-target validation, and site-list sync on merge.

`.github/CODEOWNERS`: `@ppfeister` owns `tox.ini`, `pytest.ini`, `tests/`, `pyproject.toml`. `@sdushantha` owns repo-level files. Get their review for changes in those areas.
