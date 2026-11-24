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
  when the caller has a `pyrightconfig.json`. Tool versions are pinned in
  the reusable (`ruff`, `pyright`) so CI is deterministic; bump them
  deliberately after verifying locally.
- `lint-web.yml` — `prettier --check` (`paths`, default `**/*.js **/*.css`).
- `markdownlint.yml` — Markdown style (`config`, default `.markdownlint.json`;
  `globs`, default `**/*.md`). Configs follow the canonical
  `kerswell_et_al_nonhydrostatic_benchmarks` variant (see Standards §6).
- `linkcheck.yml` — lychee over source files (`args`, default markdown globs
  with a 60s timeout; callers extend with their own excludes). Source-only
  by design: no builds, so it stays fast; run it on a weekly schedule for
  link rot.
- `typos.yml` — check-only spell check against the central allowlist
  (`config/typos.toml`, `canonical-ref` default `v1`). Per-repo `.typos.toml`
  files still apply on top (local discovery merges): keep them to
  `[files]` exclusions for generated/vendored/data paths, plus any word
  that genuinely belongs to one repo — with a comment saying why.
  Everything else goes central (see Typos allowlist below).
- `lint-r.yml` — `lintr::lint_dir` (`path`, default `R`; uses the caller's
  `.lintr` config, canonical copy in
  `kerswell_et_al_nonhydrostatic_benchmarks`).
- `prune-runs.yml` — deletes old workflow runs (`days` default 7,
  `main_days` default 90). Callers keep the schedule trigger and forward
  the dispatch inputs via `with:`.

`dependabot.yml` (weekly `github-actions`) is the copy-source for repos.

## Typos allowlist

`config/typos.toml` is the single source of truth for spelling exceptions.
When typos CI fails on a new word, the fix PR adds it here —
allowlist-by-default — unless the word genuinely belongs to one repo, in
which case it stays in that repo's `.typos.toml` with a comment saying
why. Never allowlist a real typo to silence CI; fix the source text
instead. The `typos.yml` reusable pins the check to this file at the
caller's `@v1`, so allowlist updates roll out with the tag like anything
else.

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
