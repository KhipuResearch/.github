# `.github/workflows/` — Not Applicable for Self-CI

## Status

**Intentionally empty.** This directory (`.github/workflows/` **inside** the `KR-Labs/.github` meta-repository) has no executing workflows and none are planned.

Do not confuse this directory with the root-level `workflows/` directory one level up. See the disambiguation below.

## Disambiguation

There are **two** `workflows` paths in this repo:

| Path | Purpose | Auto-executes here? |
|---|---|---|
| `/workflows/` (repo root) | Consumer-copy templates for KRL downstream repos. | **No.** Off the GitHub-recognized auto-exec path. See DECISION [`0001-workflows-mislocated-at-root.md`](../../DECISIONS/0001-workflows-mislocated-at-root.md). |
| `/.github/workflows/` (this directory) | The GitHub-recognized auto-exec path for a repo's own CI. | **Would**, if it contained any YAML. It does not. |

The first path is the one SPEC.md §2 identifies as "mislocated" — i.e., GitHub auto-exec expects the second path. The current repo state:

- `workflows/pr-check.yml` and `workflows/test.yml` live at the first path (root-level).
- This directory (the second path) contains only this README.

## Justification for keeping this directory empty

Per `SPEC.md` §1:

> This repository serves two distinct purposes ... 1. Organization profile ... 2. Shared CI/CD workflow templates and documentation. There is no application code, no build, no runtime.

Per `SPEC.md` §6:

> **Not applicable.** This repo has no build system, no runtime, no package manifest.

There is nothing for a self-hosted CI to do:

- No code to lint.
- No tests to run (`tests/` is marked Not Applicable — see `tests/README.md`).
- No package to publish.
- No container to build.
- No deployment target.

The workflow templates at `/workflows/*.yml` assume a `./Private IP/<subpackage>` directory tree that does not exist in this repo (see DECISION [`0003-hardcoded-monorepo-path-coupling.md`](../../DECISIONS/0003-hardcoded-monorepo-path-coupling.md)). If they were moved into this directory they would execute, and they would fail on every run on the `working-directory: ./Private IP/...` lines.

## Candidates for future self-CI (tracked, not implemented)

Tracked in `CALIBRATION_REQUIRED.md`. A future set of this-repo self-CI workflows could include:

1. **YAML schema validation** for `workflows/*.yml` on PR.
2. **Markdown lint** for `SPEC.md`, `ARCHITECTURE.md`, `DECISIONS/*.md`, `CI_CD_*.md`, and `policy/README.md`.
3. **`policy/*.yaml` parse gate** — confirm all YAML here continues to parse.
4. **ADR format linter** — verify 7-section structure and frontmatter on PRs that touch `DECISIONS/`.
5. **Link-check** — limited scope, avoiding `profile/README.md` per operator instruction.
6. **`<TOKEN_TBD>` placeholder sweep** — count and surface outstanding `<*_TBD>` tokens as a CALIBRATION_REQUIRED signal.

None are implemented in this artifact pass.

## Reversal condition

This directory MUST receive YAML if and only if one of the following occurs:

- DECISION `0001` is reversed and `workflows/*.yml` is moved here (and rewritten to be reusable or self-applicable).
- Any of the tracked candidates above is formally adopted via a new ADR.

Until then: empty by design. Future contributors should not assume the absence is an oversight.
