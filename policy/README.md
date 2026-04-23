# policy/ — Tier U control artifacts for `.github`

This directory contains YAML control artifacts for the `KR-Labs/.github` meta-repository.

## Contents

| File | Status | Rationale |
|---|---|---|
| `access-control.yaml` | **Active** | Who can merge, who can publish a new workflow template, who can edit `profile/README.md`. |
| `secrets.yaml` | **Active** | Secret inventory. This repo ships no explicit `${{ secrets.* }}` references (`SPEC.md` §5.3) but the consumer contract for `workflows/*.yml` surfaces implicit secret assumptions. |
| `dependency.yaml` | **Not Applicable** | Justified below. |
| `ci-authority.yaml` | **Not Applicable** | Justified below. |

## `dependency.yaml` — Not Applicable

Rationale: This repository ships no dependency manifest. Per `SPEC.md` §1:

> There is no application code, no build, no runtime.

There is no `package.json`, no `pyproject.toml`, no `requirements.txt`, no `go.mod`, no `Gemfile`, no `Cargo.toml`, and no `Dockerfile` in the repository tree. The only things that could be "dependencies" are the GitHub Actions referenced inside `workflows/*.yml`, and the policy governing those pins is captured by DECISION [`0002-floating-branch-pin-on-trivy-action.md`](../DECISIONS/0002-floating-branch-pin-on-trivy-action.md), not by a `dependency.yaml`.

If this repo ever acquires a dependency manifest (e.g., a local `pyproject.toml` for lint tooling or a `package.json` for YAML validation), re-scope this file.

## `ci-authority.yaml` — Not Applicable

Rationale: The workflow files under `workflows/*.yml` do not execute against this repository. Per `SPEC.md` §2:

> These workflows do **not** execute against this repository itself on push/PR.
> They function as templates / reference implementations...

Per `SPEC.md` §7.3, neither file declares `workflow_call` or `workflow_dispatch`, so they are not reusable workflows in the GitHub Actions technical sense.

Because this repo exposes no executing CI surface, there is no "CI authority" to govern here. The templates' downstream authority in each consumer repo (what jobs can merge-block, what jobs are soft-fail) is a property of that consumer's `.github/workflows/` copy, not of this repo.

DECISION [`0001-workflows-mislocated-at-root.md`](../DECISIONS/0001-workflows-mislocated-at-root.md) formalizes the absence of self-CI authority here.

If DECISION `0001` is reversed and `workflows/` migrates to `.github/workflows/` with `workflow_call` triggers, introduce `ci-authority.yaml` at that time.

## Format notes

Both active YAML files parse under standard PyYAML. The `<TOKEN_TBD>` placeholder grammar uses angle brackets and SCREAMING_SNAKE_CASE (e.g., `<ORG_OWNER_TEAM_TBD>`, `<CODEOWNER_ROTATION_TBD>`) for fields that require operator confirmation before this policy is considered enforced.
