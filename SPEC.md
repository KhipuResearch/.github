# SPEC — .github (KRL org profile + shared workflows)

> **Status:** Forensic specification generated from repository evidence.
> **Repository type:** GitHub organization meta-repository. No runtime code.
> **Generated:** 2026-04-23 from commit `9995b3ff5b64fc8e9f1a3e77d1d23affa7f440b9` on branch `main`.

---

## 1. System Overview

This repository serves two distinct purposes for the `KR-Labs` GitHub organization:

1. **Organization profile** — Per GitHub convention, the `profile/README.md` in a repository named `.github` (owned by an organization) is rendered as the public landing page for the organization on github.com. See `profile/README.md:5` (`# Khipu Research Labs`) and the branding image reference at `profile/README.md:2` (`KRLabs_WebLogo.png`).
2. **Shared CI/CD workflow templates and documentation** — `workflows/*.yml` contains GitHub Actions workflow files used as reference or copy-source for consumer repos under the `KR-Labs` org. `CI_CD_BEST_PRACTICES.md` and `CI_CD_FIXES_SUMMARY.md` document the standards those workflows embody.

There is no application code, no build, no runtime. Deliverable is documentation and YAML.

**Scope evidence:**
- Root contents: `.gitignore`, `CI_CD_BEST_PRACTICES.md`, `CI_CD_FIXES_SUMMARY.md`, `profile/`, `workflows/` (directory listing captured 2026-04-23).
- No `package.json`, `pyproject.toml`, `Dockerfile`, `requirements.txt`, or source directories at repo root.

---

## 2. Architecture

Two-directory meta-repo:

```
.github/
├── .gitignore                    # 10 bytes — ignores .DS_Store only
├── CI_CD_BEST_PRACTICES.md       # 389 lines — standards doc
├── CI_CD_FIXES_SUMMARY.md        # 326 lines — historical fix log (2026-01-12)
├── profile/
│   ├── README.md                 # 162 lines — org landing page
│   ├── KRLabs_WebLogo.png        # 4.5 KB — logo
│   ├── banner.png                # 99.7 KB — banner asset
│   └── krlabs-banner.png         # 475 KB — alt banner
└── workflows/
    ├── pr-check.yml              # 172 lines — PR fast-feedback suite
    └── test.yml                  # 367 lines — full push/PR test suite
```

**Important architectural caveat:** The workflow files in `workflows/` are **NOT** in the GitHub-recognized location `.github/workflows/` (the conventional path where GitHub Actions auto-executes YAML). They are in the repository-root `workflows/` directory — this is the repo called `.github`, not a subdirectory of a consumer repo. That means:

- These workflows do **not** execute against this repository itself on push/PR.
- They function as **templates / reference implementations** that consumer repos are expected to copy into their own `.github/workflows/` paths.
- This interpretation is reinforced by `pr-check.yml:32` and `test.yml:66` which hardcode `working-directory: ./Private IP/krl-dashboard/frontend` and `./Private IP/krl-premium-backend` — paths that would only exist in a monorepo (the parent `KRL/` working copy), not in this standalone `.github` repo.

---

## 3. Tech Stack

- **GitHub Actions YAML** — the only configuration language used. `pr-check.yml` and `test.yml`.
- **Markdown** — for the org profile and CI docs.
- **PNG assets** — branding.

Languages/frameworks referenced by the workflows (not vendored here, but required in consumer repos):

| Tool | Version | Evidence |
|---|---|---|
| Python | 3.11 | `test.yml:19` (`PYTHON_VERSION: "3.11"`), `pr-check.yml:105` |
| Node.js | 20 | `test.yml:20` (`NODE_VERSION: "20"`), `pr-check.yml:30` |
| PostgreSQL | 16-alpine | `test.yml:32, 104, 226` |
| Redis | 7-alpine | `test.yml:46, 118, 240` |
| pytest + pytest-cov + pytest-asyncio + httpx | unpinned | `test.yml:70, 142` |
| Playwright | chromium only | `test.yml:272` (`npx playwright install --with-deps chromium`) |
| Trivy | `@master` (unpinned) | `test.yml:319` |
| pip-audit | unpinned | `pr-check.yml:108` |

---

## 4. Dependency Map — External GitHub Actions

All Actions references found in `workflows/*.yml`. Pinning style: **major-version tag only** (e.g., `@v4`). None are pinned to commit SHA. One is pinned to `@master` (floating branch — highest supply-chain risk).

| Action | Version | Used in | Pin Style |
|---|---|---|---|
| `actions/checkout` | `@v4` | `pr-check.yml:25, 59, 91, 127`; `test.yml:57, 129, 176, 251, 316, 343` | Major tag |
| `actions/setup-node` | `@v4` | `pr-check.yml:28, 94`; `test.yml:179, 254, 346` | Major tag |
| `actions/setup-python` | `@v5` | `pr-check.yml:103`; `test.yml:60, 131, 261` | Major tag |
| `actions/github-script` | `@v7` | `pr-check.yml:160` | Major tag |
| `actions/upload-artifact` | `@v4` | `test.yml:300` | Major tag |
| `codecov/codecov-action` | `@v4` | `test.yml:88, 160, 209` | Major tag |
| `aquasecurity/trivy-action` | `@master` | `test.yml:319` | **Floating branch — RISK** |
| `github/codeql-action/upload-sarif` | `@v3` | `test.yml:328` | Major tag |

**Dependabot config:** None present. No `dependabot.yml` at `.github/dependabot.yml` or `.github/workflows/` — consumer repos receive no automated Action version bumps from this repo.

---

## 5. Configuration Model

### 5.1 Workflow-level env

`test.yml:18-20`:
```yaml
env:
  PYTHON_VERSION: "3.11"
  NODE_VERSION: "20"
```

### 5.2 Per-job environment variables (test credentials — non-secret)

Backend test jobs inject the following into their step environment (`test.yml:73-78, 146-150, 282-286`):

| Variable | Value | Notes |
|---|---|---|
| `DATABASE_URL` | `postgresql://test:test@localhost:5432/<db>` | Test-only. DB name varies per job: `krl_test`, `krl_dashboard_test`, `krl_e2e_test`. |
| `REDIS_URL` | `redis://localhost:6379/0` | Test-only. |
| `SECRET_KEY` | `test-secret-key-for-ci` | Matches the safe-credential pattern from `CI_CD_BEST_PRACTICES.md:160`. |
| `ENVIRONMENT` | `test` | |
| `PLAYWRIGHT_BASE_URL` | `http://localhost:3000` | E2E only (`test.yml:294`). |
| `API_BASE_URL` | `http://localhost:8000` | E2E only. |
| `CI` | `true` | E2E only. |

### 5.3 Secrets

**No explicit `secrets.*` references in either workflow file.** Verified by scan of both files — no lines containing `${{ secrets.` syntax.

- Codecov is wrapped with `continue-on-error: true` (`test.yml:93, 165, 214`), so it degrades gracefully without a `CODECOV_TOKEN` in the absence of org-provided token.
- The GitHub-provided `GITHUB_TOKEN` is implicitly available to `actions/github-script` (`pr-check.yml:160`) and `upload-sarif` (`test.yml:328`) without explicit reference.

### 5.4 Inputs

Neither workflow declares `workflow_call` or `workflow_dispatch` inputs. They are **event-triggered only**, not reusable in the GitHub Actions technical sense (see §7).

---

## 6. Build & Run

**Not applicable.** This repo has no build system, no runtime, no package manifest. The workflows under `workflows/` are themselves not auto-executed because they live outside `.github/workflows/` (see §2).

---

## 7. Deployment / Reusable Workflows

Neither workflow is a **reusable workflow** in the GitHub Actions sense (no `on: workflow_call` trigger, no `inputs:`, no `secrets:` block). They are **event-triggered workflow templates** intended to be copy-pasted (or reference-only) by consumer repos.

### 7.1 `workflows/pr-check.yml` — PR Checks

**Trigger** (`pr-check.yml:11-13`):
```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened]
```

**Jobs** (5 total, all on `ubuntu-latest`):

| Job | Name | Purpose | Evidence |
|---|---|---|---|
| `quick-checks` | Quick Checks | Install Node 20, lint (`npm run lint`), Prettier check (soft-fail), `tsc --noEmit` (soft-fail). | `pr-check.yml:19-48` |
| `commit-check` | Commit Message Check | Iterate each commit in the PR; warn if first line >72 chars. | `pr-check.yml:53-80` |
| `dependency-audit` | Dependency Audit | `npm audit --audit-level=high` + `pip-audit` on premium + dashboard backends, with `--ignore-vuln GHSA-v6rh-hp5x-86rv`. All soft-fail (`\|\| true`). | `pr-check.yml:85-116` |
| `pr-size` | PR Size Check | Diff stats; warn if additions+deletions >1000. | `pr-check.yml:121-149` |
| `label-check` | Label Check | Inline JS (github-script@v7) requiring one of: `bug`, `enhancement`, `documentation`, `maintenance`, `breaking`. Warning only, not a hard gate. | `pr-check.yml:154-172` |

**Soft-fail pattern:** Most substantive checks end with `|| true` (Prettier, tsc, npm audit, pip-audit). Only `npm run lint` (`pr-check.yml:40`) is a hard gate.

### 7.2 `workflows/test.yml` — Test Suite

**Trigger** (`test.yml:12-16`):
```yaml
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
```

**Jobs** (6 total, all on `ubuntu-latest`):

| Job | Needs | Services | Coverage gate | Evidence |
|---|---|---|---|---|
| `backend-premium-tests` | — | postgres:16, redis:7 | `--cov-fail-under=80` against `app` | `test.yml:26-93` |
| `backend-dashboard-tests` | — | postgres:16, redis:7 | `--cov-fail-under=80` against `.` | `test.yml:98-165` |
| `frontend-tests` | — | — | lines/functions/statements ≥80, branches ≥70 | `test.yml:170-214` |
| `e2e-tests` | `frontend-tests` | postgres:16, redis:7 | N/A — artifact only | `test.yml:219-305` |
| `security-scan` | — | — | Trivy CRITICAL+HIGH, SARIF upload soft-fail | `test.yml:310-331` |
| `build-check` | `frontend-tests` | — | `npm run build` + `.next/` presence check | `test.yml:336-367` |

**Hardcoded monorepo paths** (a critical consumer-repo constraint): every job uses `working-directory: ./Private IP/<subpackage>` (e.g., `test.yml:66, 138, 186, 267, 275, 281, 292, 353`). These paths assume the consumer checkout exposes a `Private IP/` directory containing `krl-dashboard/frontend`, `krl-dashboard/backend`, and `krl-premium-backend`. That matches the KRL monorepo layout noted in project memory, but would break for any repo that copies the file without that exact layout.

### 7.3 Reusable vs callable vs event-triggered

| Class | Present? |
|---|---|
| `workflow_call` (true reusable) | **No.** Neither workflow declares it. |
| `workflow_dispatch` (manually callable) | **No.** Neither workflow declares it — despite `CI_CD_FIXES_SUMMARY.md:147-153` recommending it for other workflows in consumer repos. |
| Event-triggered (`push`, `pull_request`) | **Yes.** Both. |

---

## 8. API Contracts

**Not applicable in the traditional sense** (no HTTP or RPC surface).

The "API" exposed by this repo is the **workflow template contract** consumer repos inherit when they copy these YAMLs:

- **Inputs:** None — no `inputs:` block on either workflow.
- **Secrets:** None declared (Codecov token is optional due to `continue-on-error`).
- **Environment assumptions (implicit contract):**
  - A directory tree rooted at `Private IP/krl-dashboard/frontend`, `Private IP/krl-dashboard/backend`, `Private IP/krl-premium-backend` relative to the checkout root.
  - `package.json` with scripts: `lint`, `type-check`, `test:coverage`, `build` in the frontend.
  - `requirements.txt` in each backend directory.
  - A FastAPI entrypoint at `main:app` for the dashboard backend (`test.yml:288`).
  - `.next/` build output directory as success signal (`test.yml:363`).

Any consumer repo copying these without that layout must refactor the paths.

---

## 9. Data Model

**Not applicable.** No schemas, migrations, or persisted data belong to this repository.

---

## 10. Security Posture

### 10.1 Action pinning

All GitHub Actions are pinned to **major-version tags** (`@v4`, `@v5`, `@v7`, `@v3`), not commit SHAs. This is the GitHub-recommended minimum but not the gold-standard SHA pinning required under SLSA L3. One outlier:

- `aquasecurity/trivy-action@master` (`test.yml:319`) — pinned to a **floating branch**, which means any push to the action's `master` branch is automatically consumed. **Highest supply-chain risk in the repo.**

### 10.2 Secret hygiene (documented vs implemented)

`CI_CD_BEST_PRACTICES.md` §3 (`CI_CD_BEST_PRACTICES.md:108-162`) and `CI_CD_FIXES_SUMMARY.md` §2 (`CI_CD_FIXES_SUMMARY.md:55-92`) prescribe:

- `.gitleaksignore` at repo level
- `.gitleaks.toml` allowlist
- Safe test-credential prefixes: `test-`, `mock-`, `dummy-`, `fake-`

The workflows in this repo comply with the naming convention (`SECRET_KEY: test-secret-key-for-ci` — `test.yml:77`) but this repo itself ships **no `.gitleaksignore` or `.gitleaks.toml`** (only a 10-byte `.gitignore` containing `.DS_Store`). The remediation from the fix summary was applied to `Private IP/krl-data-connectors`, not here.

### 10.3 Secret scanning in CI

`test.yml:310-331` — Trivy filesystem scan, SARIF uploaded to GitHub Security tab. `continue-on-error: true` on the upload step (`test.yml:331`) means SARIF upload failures never fail the workflow.

### 10.4 Dependency audit

`pr-check.yml:85-116` — `npm audit --audit-level=high` (soft-fail) + `pip-audit` with an explicit ignore for `GHSA-v6rh-hp5x-86rv` on both backends (`pr-check.yml:112, 116`). No rationale is inline-documented for the ignore.

### 10.5 No secrets committed

Grep for `${{ secrets.` across both YAMLs returns nothing. Consumer repos relying on these templates must wire their own secrets.

---

## 11. Observability

- **Workflow summary artifacts:** Playwright HTML report uploaded as `playwright-report` artifact with 30-day retention (`test.yml:299-305`). Triggered `if: always()` — survives test failure.
- **Coverage reporting:** Codecov uploads with per-job flags (`backend-premium`, `backend-dashboard`, `frontend` — `test.yml:91, 163, 212`). All three wrapped in `continue-on-error: true`.
- **SARIF upload:** Trivy results pushed to GitHub code-scanning Security tab (`test.yml:327-331`). Also `continue-on-error`.
- **Summary prints:** `pr-size` job emits PR stats to stdout (`pr-check.yml:139-148`); `commit-check` prints per-commit `✓` marks (`pr-check.yml:79`).

No structured logging, no metrics export, no Sentry wiring in these workflow files themselves.

---

## 12. Failure Modes

### 12.1 Template drift

Consumer repos copying these workflows are pinned to the **hardcoded `./Private IP/...` paths** (`test.yml:66, 138, 186, 267`). Any repo that reorganizes its directory layout silently breaks unless those paths are patched. There is no linter or check that flags this drift.

### 12.2 Soft-fail overuse

`pr-check.yml` uses `|| true` on Prettier (`:44`), tsc (`:48`), `npm audit` (`:100`), and pip-audit (`:112, 116`). Frontend `type-check` in `test.yml:195` is also soft-failed (`# Type check optional for now`). These will never block a PR on the issues they nominally guard.

### 12.3 Floating-branch Action pin

`aquasecurity/trivy-action@master` (`test.yml:319`) — a compromised or breaking change to the `master` branch of that action is consumed on the next workflow run with no notice.

### 12.4 Missing `workflow_dispatch`

Neither file declares `workflow_dispatch`. A consumer repo adopting these verbatim cannot manually trigger them from the Actions UI — only pushes and PRs to `main`/`develop` will run `test.yml`. `CI_CD_BEST_PRACTICES.md:82` explicitly recommends adding `workflow_dispatch` as a best practice; the templates here do not follow their own guidance.

### 12.5 Empty-diff edge case in `pr-size`

`pr-check.yml:134-137` parses `git diff --stat | tail -1 | awk '{print $4}'` to extract additions. When the PR has zero file changes the summary line format differs, and `$4`/`$6` yield empty strings. The subsequent `$((additions + deletions))` arithmetic then fails. No guard present.

### 12.6 No dependabot

No `dependabot.yml` — Action version bumps require manual PRs.

---

## 13. Testing Strategy

### 13.1 Of the templates themselves

None. There is no test for the YAML files in this repo, no `act`-based dry run, no schema validator config committed. `CI_CD_BEST_PRACTICES.md:305-312` recommends using `act` locally but does not wire it into CI.

### 13.2 Prescribed for consumer repos

The templates **are** the test strategy consumer repos are meant to adopt. Summarized:

- Unit + integration coverage ≥80% (`test.yml:84, 156, 203-206`).
- Branch coverage ≥70% for frontend (`test.yml:205`).
- Playwright E2E gated on `needs: [frontend-tests]` (`test.yml:222`).
- Security scan (Trivy) every push/PR.

This aligns with the common `rules/common/testing.md` requirement of 80% minimum coverage, though the dashboard backend uses `--cov=.` (`test.yml:153`) which includes test files in the coverage denominator — a methodological quirk vs. `--cov=app` in the premium backend (`test.yml:81`).

---

## 14. Operational Notes

### 14.1 Inferred consumer repos

From the hardcoded paths and references, the templates target these KRL monorepo subprojects:

- `Private IP/krl-dashboard/frontend` (Next.js — `.next/` output) — `pr-check.yml:32`, `test.yml:183`
- `Private IP/krl-dashboard/backend` (FastAPI — `uvicorn main:app`) — `pr-check.yml:115`, `test.yml:138`, `test.yml:288`
- `Private IP/krl-premium-backend` (FastAPI) — `pr-check.yml:111`, `test.yml:66`

Project memory additionally lists `krl-data-connectors`, `krl-model-zoo`, `krl-frameworks`, `krl-causal-policy-toolkit`, `krl-geospatial-tools`, `krl-network-analysis` as KRL org repos (see `CI_CD_FIXES_SUMMARY.md:168-193`, §5), but the templates in this repo do **not** cover them — those repos maintain their own `.github/workflows/` configurations independently.

### 14.2 Documented remediation history

`CI_CD_FIXES_SUMMARY.md:1-326` records the 2026-01-12 fix pass across 4 repos:
- `krl-premium-backend` — dependency additions (`numpy`, `geoalchemy2`, `pytest-timeout`) — `:15-51`
- `krl-data-connectors` — `.gitleaksignore` + `.gitleaks.toml` allowlist — `:55-92`
- `krl-model-zoo` — disk-space cleanup in docs workflow — `:96-131`
- `krl-frameworks` — disk-space cleanup + `--no-cache-dir` in package-governance workflow — `:167-191`

### 14.3 Known open items per fix log

`CI_CD_FIXES_SUMMARY.md:211-248` flags unresolved items as of 2026-01-12:
- YAML syntax errors in `krl-premium-backend/.github/workflows/api-compatibility.yml` and `schema-gate.yml` — `:220-224`
- License enforcement test failures across multiple repos — `:226-229`
- Security-workflow 0s-duration failures — `:232-235`
- Long test suites (6–8 min) — `:237-240`

### 14.4 Copyright & license

Both workflow YAMLs carry:
```
# Copyright 2025 KR-Labs. All rights reserved.
# SPDX-License-Identifier: Apache-2.0
```
(`pr-check.yml:2-5`, `test.yml:2-5`). No top-level LICENSE file in this `.github` repo.

---

## 15. Module Inventory

| Path | Lines / Size | Purpose |
|---|---|---|
| `/.gitignore` | 10 B (1 line) | Ignores `.DS_Store`. |
| `/CI_CD_BEST_PRACTICES.md` | 389 lines | Org-wide CI/CD standards doc: dependency management, workflow config, security scanning, disk-space, testing. Last updated 2026-01-12 (`:389`). |
| `/CI_CD_FIXES_SUMMARY.md` | 326 lines | Historical record of fixes applied 2026-01-12 across 4 KRL repos; lists remaining issues and expected outcomes. |
| `/profile/README.md` | 162 lines | Public org landing page. Opens with centered KRL logo (`:1-3`), frames KRL as causal-inference infrastructure, lists featured KASS notebooks (NB07, NB14, NB15, NB20, NB22 — `:29-33`), methodological citations (`:87-93`), package inventory (`:103-107`), and contact info. |
| `/profile/KRLabs_WebLogo.png` | 4.5 KB | Org logo (referenced from raw.githubusercontent.com URL at `profile/README.md:2`). |
| `/profile/banner.png` | 99.7 KB | Banner asset (not referenced by README — orphan?). |
| `/profile/krlabs-banner.png` | 475 KB | Larger banner asset (not referenced by README — orphan?). |
| `/workflows/pr-check.yml` | 172 lines | PR fast-feedback: lint, prettier, tsc, commit-msg length, dep audit, PR size, label check. 5 jobs. |
| `/workflows/test.yml` | 367 lines | Full test suite: premium backend + dashboard backend + frontend + E2E + security scan + build check. 6 jobs, 2 with `needs:` dependencies. |

**Orphan assets:** `profile/banner.png` and `profile/krlabs-banner.png` are not referenced by `profile/README.md`. They may be manually used on external sites (the README links to `krlabs.dev` at `:78, 132, 134, 142, 150`).

---

## 16. Repository Artifacts

### 16.1 `profile/README.md` — the org-page content

Rendered at `https://github.com/KR-Labs` as the organization landing page per GitHub's `.github`-repo convention.

**Structure** (162 lines):
1. Centered logo image (`:1-3`)
2. `# Khipu Research Labs` title + tagline (`:5-16`)
3. **KASS** notebook overview (`:19-53`) — featured table lists NB07, NB14, NB15, NB20, NB22 with method + domain + type columns (`:29-33`).
4. Audience sections — analysts, researchers, students, government (`:57-65`).
5. **What's Free** (`:69-81`) — FRED Basic, BLS Basic, Custom CSV; rest gated to Pro/Enterprise.
6. **Methodological Standards** (`:85-95`) — Imbens & Lemieux (2008), Abadie/Diamond/Hainmueller (2010, 2015), Cinelli & Hazlett (2020), Cattaneo/Idrobo/Titiunik (2020), OMB Circular A-4.
7. **KRL Suite** package table (`:99-109`) — `krl-open-core`, `krl-data-connectors` (public PyPI); `krl-causal-policy-toolkit` (platform gated).
8. Contributing, Go Further, What This Is Not, Connect sections (`:113-154`).

Key URLs cited: `krlabs.dev`, `checkout.krlabs.dev/b/6oUdRa4Rp3lhbZv8939sk02` (Community tier Stripe checkout — `:130`), `info@krlabs.dev`, `support@krlabs.dev`, `github.com/KhipuResearch`.

### 16.2 `CI_CD_BEST_PRACTICES.md` — standards

Five sections (`:1-10` TOC):
1. **Dependency Management** (`:14-56`) — `pyproject.toml` declarations; `pip install -e ".[test]"` pattern.
2. **Workflow Configuration** (`:60-104`) — avoid trigger mismatches; validate YAML; avoid multiline-string colon pitfalls.
3. **Security Scanning** (`:108-162`) — `.gitleaksignore` + `.gitleaks.toml` allowlist; safe credential prefixes.
4. **Disk Space Management** (`:166-215`) — remove `/usr/share/dotnet`, `/opt/ghc`, `/usr/local/share/boost`, `$AGENT_TOOLSDIRECTORY`; `--no-cache-dir` pip.
5. **Testing Best Practices** (`:219-284`) — pytest markers, service wiring, timeouts.

Appendices: checklist for new workflows (`:288-299`), quick-fix commands (`:303-337`), repo-specific fixes applied (`:342-360`), monitoring cadence (`:363-377`).

### 16.3 `CI_CD_FIXES_SUMMARY.md` — historical fix log

Dated 2026-01-12 (`:1, 324`). Reports 4 repos fixed, 7 issue categories addressed, 90%+ expected reduction in CI/CD failures. Categories: missing Python deps (`:15-51`), 1,149 gitleaks false positives (`:55-92`), disk space exhaustion (`:96-131`), workflow-trigger misconfig (`:135-163`), package-governance cascade failures (`:167-191`). Lists remaining work by priority (`:211-248`).

---

## Evidence gaps / caveats

- **Workflow activation location is atypical.** Files under `/workflows/` (not `/.github/workflows/`) mean they do not execute for this repo; interpretation as "templates" is inferred from content, not declared in a README.
- **No inline README** in `/` or `/workflows/` explains the template relationship; only `CI_CD_BEST_PRACTICES.md` and `CI_CD_FIXES_SUMMARY.md` contextualize the workflows indirectly.
- **Orphan banner assets** (`profile/banner.png`, `profile/krlabs-banner.png`) — unreferenced; usage external to this repo is inferred, not evidenced.
- **Dependabot absence** — no automated Action version bumps configured for this repo.
- **No LICENSE file** at repo root despite SPDX headers declaring Apache-2.0 in each YAML.
- **No `workflow_dispatch` or `workflow_call`** — workflows are not reusable in the GitHub Actions technical sense despite the "shared templates" framing.

---

*End of SPEC.*
