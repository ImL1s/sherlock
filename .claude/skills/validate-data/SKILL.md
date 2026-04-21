---
name: validate-data
description: Validate sherlock_project/resources/data.json against its JSON schema. Use after any edit to the site manifest.
---

Run the offline-test gate, which includes the schema check:

```
uvx --from tox tox -e offline
```

This bootstraps `tox` via `uvx` (no global tox/poetry needed), installs the project + `pytest` + `jsonschema`, and runs every non-network test — including `tests/test_manifest.py::test_validate_manifest_against_local_schema` which validates `sherlock_project/resources/data.json` against `sherlock_project/resources/data.schema.json`.

If a schema violation is reported, fix the offending entry against `data.schema.json` (required fields, types, regex formats for `url`, `urlMain`, `errorType`, etc.) and re-run.

Do **not** run the full `-m validate_targets` suite locally — CI (`.github/workflows/validate_modified_targets.yml`) runs it per-PR against only the modified sites, which is the supported workflow.
