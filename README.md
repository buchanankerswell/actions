# actions: shared reusable workflows

Reusable GitHub Actions workflows shared across repositories. Callers pin a
major tag (`@v1`) so fixes roll out without copy-paste drift.

## Workflows

- `python-test-release.yml` — multi-OS pytest + Codecov, then tag-gated
  TestPyPI → PyPI → GitHub release. Input: `package` (coverage flag).
  Secrets: `codecov-token`, `test-pypi-api-token`, `pypi-api-token`.
- `python-docs.yml` — Sphinx HTML build + short-lived artifact
  (`retention-days`, default 14; rebuildable from any tag).
- `validate-site.yml` — static-site checks (`site-dir`, default `docs`;
  runs `pipeline/check.py` + `pipeline/check_media.py`, which must stay
  stdlib-only so no install step is needed).
- `design-drift.yml` — diffs vendored design files against canonical
  `design/` (`static-dir`, required; `files`, defaults to the three
  canonical files; `canonical-ref`, default `v1`).
- `lint-python.yml` — `ruff check` + `ruff format --check` (`paths`,
  default `.`; `check-format`, default true) + `pyright`, which runs only
  when the caller has a `pyrightconfig.json`. Lint tools float by design
  (same convention as `npx prettier` in the Makefiles).
- `lint-web.yml` — `prettier --check` (`paths`, default `**/*.js **/*.css`).
- `markdownlint.yml` — Markdown style (`config`, default `.markdownlint.json`;
  `globs`, default `**/*.md`). Configs follow the canonical
  `kerswell_et_al_nonhydrostatic_benchmarks` variant (see Standards §6).
- `linkcheck.yml` — lychee over source files (`args`, default markdown globs
  with a 60s timeout; callers extend with their own excludes). Source-only
  by design: no builds, so it stays fast; run it on a weekly schedule for
  link rot.
- `typos.yml` — check-only spell check. Per-repo allowlists live in the
  caller's `.typos.toml`; `aspect/.typos.toml` is the house-style example
  (data/bib/vendored exclusions plus domain words).
- `lint-r.yml` — `lintr::lint_dir` (`path`, default `R`; uses the caller's
  `.lintr` config, canonical copy in
  `kerswell_et_al_nonhydrostatic_benchmarks`).
- `prune-runs.yml` — deletes old workflow runs (`days` default 7,
  `main_days` default 90). Callers keep the schedule trigger and forward
  the dispatch inputs via `with:`.

`dependabot.yml` (weekly `github-actions`) is the copy-source for repos.

## Example caller

```yaml
jobs:
  lint-python:
    uses: buchanankerswell/actions/.github/workflows/lint-python.yml@v1
  test-and-release:
    uses: buchanankerswell/actions/.github/workflows/python-test-release.yml@v1
    with:
      package: mypackage
    secrets:
      codecov-token: ${{ secrets.CODECOV_TOKEN }}
      test-pypi-api-token: ${{ secrets.TEST_PYPI_API_TOKEN }}
      pypi-api-token: ${{ secrets.PYPI_API_TOKEN }}
```

## Caller conventions

- One file per concern: `ci.yml` (fast checks: lint, style, spelling),
  `docs.yml` (Sphinx build), `release.yml` (test + publish),
  `prune-runs.yml` (run pruning). Every caller starts with `---`, sets
  `permissions: contents: read` at the top level (plus `contents: write`
  only for releases), and sets `concurrency: cancel-in-progress: true`
  where reruns are safe.
- Triggers: `push` to `main` (+ tags `v*` for publishables), `pull_request`,
  and a weekly `schedule` for link-heavy checks. Runners are pinned to
  `ubuntu-24.04`; reusable actions are pinned (`checkout/setup-python` at
  v6, third-party actions at their major tags).

## Versioning

Pin callers to a major tag (`@v1`). Breaking changes to inputs/secrets get a
new major tag; fixes stay on the same tag (move it forward).
