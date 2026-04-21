---
name: verify
description: Verify local changes before claiming completion — runs the same lint + offline-test gates that CI runs. Use after making code edits to confirm nothing is broken.
---

Run both gates in order and report results:

1. Lint: `uvx --from tox tox -e lint` (runs `ruff check`).
2. Offline tests: `uvx --from tox tox -e offline` (runs `pytest -v -m "not online"`).

If either gate fails, surface the failing output and stop. Do **not** claim a task is complete while either gate is red.

`uvx --from tox` bootstraps `tox` into a cached ephemeral env — no global `tox`/`poetry` install required. If the repo already has `tox` on PATH (e.g., inside an active dev venv), prefer the plain `tox -e lint` / `tox -e offline` form for speed.

Do not run `-m validate_targets` tests — those hit live networks and are validated by CI per-PR on modified sites. Running them locally is slow and noisy.
