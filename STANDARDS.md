# Org standards (single source of truth)

Applies to Python web apps (FastAPI + dependency-free vanilla JS UIs) and
static sites. New repos start compliant via the template repos (see
Maintenance); existing repos converge through review against this file.

## 1. UI/UX

- Tokens: light default + `.dark-mode` override (`--bg/--panel/--ink/--muted/
  --link/--hairline/--radius`), Inter stack with tabular numerals,
  `2px` link focus ring. Structural styling uses tokens; only semantic
  accents (review, tick, plot series) keep custom colors.
- Components: sticky topbar (brand left, search/controls middle, action
  pills right), flat hairline pills for buttons and links, tabs with a
  sliding indicator for the two main views, centered modal dialogs for
  everything secondary.
- Help (`?`) and settings (`s`) are overlay popups sharing one dialog
  pattern; `Esc` and backdrop clicks dismiss; dialogs are mutually
  exclusive and take focus on open. Tabs are never removed; popups cover
  help, settings, and reference material.
- Shortcuts: `/` focuses search where a corpus exists, `?` help,
  `s` settings, `Esc` close. Single-letter keys are ignored inside inputs
  and editable text. Document only implemented keys.
- Main views stay focused on the primary task; configuration and reference
  live in popups. Search supports multi-word fuzzy matching with `-`
  exclusion; empty states explain and suggest.

## 2. Make pipeline

- Two files: root `Makefile` (targets) + `project.mk` (shared boilerplate:
  project root, persistent paths/vars, dated log files, safe-removal macro
  confined to the project root). Subdir Makefiles include it relatively.
- Bare `make` shows `help` (the default goal).
- Mandatory targets: `build` (reproducible from scratch: env + tests +
  docs where they exist; starts nothing, downloads nothing), `all`
  (`build` + `run`), `run`, `test`, `environment` (creates the conda env
  only if missing), `clean` (safe caches), `deep-clean` (`clean` plus logs
  and rebuildables — never source data or the env), `help`.
- Quiet recipes: one log line per removal, banners only on
  clean/deep-clean/help, file logging for slow leaf steps only (never on
  recursive `make`, which breaks `-n` dry runs).

## 3. FastAPI (app repos)

- Agent-facing JSON API; the human web UI is served at `/`. Same endpoints
  serve browsers and agents; the service is stateless.
- Common entrypoints: `GET /health` (`{"status": "ok"}`), `POST /validate`
  (echoes the data model), `POST /fit*` (computation), one download route
  (`/export` or equivalent), `GET /` plus one tab route serving the UI.
  Identical `main()` uvicorn launch. Data models stay domain-specific.
- Errors: `404` unknown identifier, `422` invalid payload, cause in the
  response body. Document only the codes each app actually returns.
- Agent docs (`api.rst`): intro, data model with a minimal valid payload,
  endpoint contract table, response-shape and error notes, curl examples.

## 4. Docs / README

- README skeleton, in order: title + badges + docs link + one-line
  description, then Prerequisite software → Installation Guide → Usage
  Guide → Citation → Acknowledgement. No other sections; coauthors and
  funders belong in Acknowledgement.
- Published apps document in Sphinx: `usage.rst` for humans, `api.rst` for
  agents, method and data-model detail lives there — never duplicated into
  the README. Static sites are exempt from Sphinx.
- Software citations are BibTeX with minted DOIs; pre-release software may
  carry `XXXXXXX` DOI boilerplate plus a `.zenodo.json` so the DOI can be
  minted on first release tag.

## 5. Software stack

- Minimal and modern: Python `>=3.13`, FastAPI/Uvicorn/Pydantic/NumPy for
  apps; dependency-free HTML/CSS/vanilla JS (no bundler); Sphinx for app
  docs; pytest/httpx/ruff/pyright for quality.
- Runtime deps stay unpinned ranges (library convention); reproducibility
  comes from the conda `environment.yml`, CI matrix, and short-lived
  artifacts — not lockfiles.
- Published packages use `setuptools-scm` + PyPI metadata; non-published
  code gets a metadata-only `pyproject.toml` (no build backend, so it can
  never be uploaded). Lint configs (ruff 150 cols, prettier) are identical
  everywhere; verify-only tools (e.g. headless browsers) stay ad-hoc and
  unpinned.

## 6. Build / release

- All CI lives in `buchanankerswell/actions` as reusable workflows pinned
  by major tag (`@v1`); repos hold thin callers only. Fixes roll forward
  on the same tag; breaking input changes get a new major tag.
- Python apps: 3-OS pytest + Codecov, tag-gated TestPyPI → PyPI → GitHub
  release. Docs build uploads HTML artifacts capped at 14 days
  (rebuildable from any tag; release dists stay on their tags). Sites:
  fixture-free validation on PRs, main, and tags; publishing is automatic
  from `main`.
- Workflow runs are pruned on a schedule (7 days branch, 90 days main/tag).
  The Codecov app is shared; coverage-free repos install nothing.

## Maintenance

- This file lives at the root of `buchanankerswell/actions`, versioned
  with the same tags as the workflows. Change it by PR there.
- Enforcement splits three ways: reusable workflows and CI checks enforce
  the automatable parts (tests, releases, pruning, artifact retention,
  UI drift checks); template repos (one per stack: FastAPI app, static
  site) bake the rest in so new repos start compliant; review against this
  file covers the remainder (README skeleton, API contract, UI norms).
- Keep the automatable and the documented in sync: any new enforced check
  gets a paragraph here, and any new paragraph names how it is checked.
