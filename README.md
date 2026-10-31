# actions: shared reusable workflows

Reusable GitHub Actions workflows for `rxndb`, `pkft`, and `light_slides`.
Repos call these by tag (`@v1`) so fixes roll out without copy-paste drift.

## Workflows

- `python-test-release.yml` — 3-OS pytest + Codecov, then tag-gated
  TestPyPI → PyPI → GitHub release. Input: `package` (coverage flag).
  Secrets: `codecov-token`, `test-pypi-api-token`, `pypi-api-token`.
- `python-docs.yml` — Sphinx HTML build + short-lived artifact
  (`retention-days`, default 14; rebuildable from any tag).
- `validate-site.yml` — static-site checks (`site-dir`, default `docs`).
- `prune-runs.yml` — deletes old workflow runs (7d branch, 90d main/tag,
  overridable per dispatch). Callers keep the schedule trigger.

## Example caller

```yaml
jobs:
  test-and-release:
    uses: buchanankerswell/actions/.github/workflows/python-test-release.yml@v1
    with:
      package: rxnDB
    secrets:
      codecov-token: ${{ secrets.CODECOV_TOKEN }}
      test-pypi-api-token: ${{ secrets.TEST_PYPI_API_TOKEN }}
      pypi-api-token: ${{ secrets.PYPI_API_TOKEN }}
```

## Versioning

Pin callers to a major tag (`@v1`). Breaking changes to inputs/secrets get a
new major tag; fixes stay on the same tag (move it forward).
