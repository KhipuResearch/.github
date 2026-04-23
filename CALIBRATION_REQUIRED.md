# CALIBRATION_REQUIRED

Deltas between the forensic `SPEC.md` (source of truth, 412 lines, commit `9995b3ff5b64fc8e9f1a3e77d1d23affa7f440b9`) and the Tier U control-artifact stack now committed.

This document tracks items that either (a) require operator confirmation before they can be considered enforced, or (b) represent documented gaps the specification flagged that this artifact pass has **not** remediated.

---

## 1. Outstanding `<TOKEN_TBD>` placeholders

Every occurrence below requires operator input before the relevant artifact moves from "documented" to "enforced."

### 1.1 `policy/access-control.yaml`

- `<LAST_REVIEW_DATE_TBD>` — date of last access-control review
- `<ORG_OWNER_TEAM_TBD>` — GitHub team slug for KR-Labs org owners
- `<PLATFORM_MAINTAINERS_TEAM_TBD>` — GitHub team slug for CI/CD template maintainers
- `<PROFILE_EDITORS_TEAM_TBD>` — GitHub team slug for the public profile page curators
- `<SIGNED_COMMITS_POLICY_TBD>` — true/false on required signed commits for `main`
- `<LINEAR_HISTORY_TBD>` — true/false on required linear history for `main`
- `<ALLOW_REBASE_MERGE_TBD>` — true/false
- `<CODEOWNERS_FILE_TBD>` — path to the CODEOWNERS file (not yet created — see §3.3)
- `<AUDIT_LOG_LOCATION_TBD>` — where access-change audit logs land
- `<AUDIT_RETENTION_TBD>` — retention period

### 1.2 `policy/secrets.yaml`

- `<LAST_REVIEW_DATE_TBD>` — date of last secret-inventory review
- `<CODECOV_ROTATION_POLICY_TBD>` — policy for optional Codecov token rotation
- `<CODECOV_ORG_OWNER_TBD>` — org-level owner for the Codecov integration
- `<GITLEAKS_INTRODUCTION_DECISION_TBD>` — should this repo ship `.gitleaksignore` / `.gitleaks.toml` matching `SPEC.md` §10.2?
- `<REVIEW_FREQUENCY_TBD>` — cadence
- `<LAST_INVENTORY_DATE_TBD>` — most recent full secret inventory date
- `<NEXT_AUDIT_DUE_TBD>` — next inventory due date

### 1.3 DECISIONS

- `0001-workflows-mislocated-at-root.md` §7 — `<REPO_ROOT_TBD>`, `<CONSUMER_REPO_COUNT_TBD>`, `<DRIFT_THRESHOLD_TBD>`
- `0002-floating-branch-pin-on-trivy-action.md` §6–§7 — `<REPO_ROOT_TBD>`, `<REMEDIATION_DEADLINE_TBD>`, `<TRIVY_VERSION_TBD>`, `<TRIVY_SHA_TBD>`, `<UPSTREAM_ARCHIVE_SIGNAL_TBD>`
- `0003-hardcoded-monorepo-path-coupling.md` §6–§7 — `<REPO_ROOT_TBD>`, `<NEW_CONSUMER_TRIGGER_TBD>`

---

## 2. Artifacts intentionally NOT created

Per operator instruction, the following were not created or modified, despite being within the natural Tier U envelope:

- **Root `README.md`** — explicitly excluded. No repo-root README shipped.
- **`profile/README.md`** — unchanged. Declared read-only by operator instruction.
- **`profile/banner.png`** and **`profile/krlabs-banner.png`** — unreferenced orphan assets per `SPEC.md` §15. Not removed; retention is a curator decision for `<PROFILE_EDITORS_TEAM_TBD>`.

---

## 3. SPEC-flagged gaps NOT remediated by this pass

### 3.1 No Dependabot

`SPEC.md` §4 and §12.6 document the absence of `.github/dependabot.yml`. This artifact pass does not introduce one. Recommendation: add in a follow-up that also reconciles DECISION `0002` (Action-pin policy) with automated bumps.

### 3.2 No LICENSE at repo root

`SPEC.md` §14.4 notes both workflow YAMLs carry `SPDX-License-Identifier: Apache-2.0` headers but the repo has no `LICENSE` file. Not addressed here. Recommendation: copy the Apache-2.0 license text to `LICENSE` at repo root.

### 3.3 No CODEOWNERS

Referenced by `policy/access-control.yaml` as `<CODEOWNERS_FILE_TBD>`. Not created by this pass. Recommendation: once `<ORG_OWNER_TEAM_TBD>`, `<PLATFORM_MAINTAINERS_TEAM_TBD>`, `<PROFILE_EDITORS_TEAM_TBD>` are confirmed, author a CODEOWNERS at `.github/CODEOWNERS` (which *is* on the GitHub-recognized path for CODEOWNERS even though it overlaps structurally with the empty self-workflows directory).

### 3.4 No `.gitleaksignore` / `.gitleaks.toml`

`SPEC.md` §10.2:

> The workflows in this repo comply with the naming convention (`SECRET_KEY: test-secret-key-for-ci`) but this repo itself ships **no `.gitleaksignore` or `.gitleaks.toml`**.

Not remediated. Tracked under `<GITLEAKS_INTRODUCTION_DECISION_TBD>` in `policy/secrets.yaml`.

### 3.5 Workflow-template drift detection

DECISION `0003` §5 deferred a template-drift linter. No tooling to compare consumer-repo copies of `workflows/*.yml` against the canonical source exists yet.

### 3.6 `workflow-templates/` migration

DECISION `0001` deferred the GitHub-convention migration from `workflows/` to `workflow-templates/` with accompanying `.properties.json` metadata. No migration done.

### 3.7 No `act`-based dry run

`SPEC.md` §13.1 notes `CI_CD_BEST_PRACTICES.md` recommends `act` but no CI wiring exists. See `tests/README.md` and `.github/workflows/README.md` for future-candidate list.

### 3.8 Soft-fail audit

`SPEC.md` §12.2 documents the overuse of `|| true` in `pr-check.yml` (Prettier `:44`, tsc `:48`, `npm audit` `:100`, pip-audit `:112, 116`) and in `test.yml` (`type-check` `:195`; Codecov `:93, 165, 214`; SARIF upload `:331`). This pattern is noted in `ARCHITECTURE.md` §6 as a known gap. No ADR on soft-fail posture authored here — deferred.

### 3.9 `pr-size` empty-diff edge case

`SPEC.md` §12.5 documents a parsing bug in `pr-check.yml:134-137` (empty-diff case yields empty `$4`/`$6` and breaks `$((additions + deletions))`). Not patched. This is a bug in the template; fix would require modifying `workflows/pr-check.yml` itself — out of scope for this documentation-layer pass.

### 3.10 Coverage-method mismatch

`SPEC.md` §13.2 notes the dashboard backend uses `--cov=.` (`test.yml:153`) which includes test files in the denominator, while the premium backend uses `--cov=app` (`test.yml:81`). Methodological quirk; not resolved.

### 3.11 YAML-syntax errors in downstream repos

`SPEC.md` §14.3 / `CI_CD_FIXES_SUMMARY.md` §4 flags unresolved YAML errors in `krl-premium-backend/.github/workflows/api-compatibility.yml` and `schema-gate.yml`. These are in downstream consumer repos, not this `.github` repo. Out of scope but recorded here for operator awareness.

---

## 4. Validation performed for this pass

- `ls` of the repo pre- and post-write confirms file placement.
- YAML files (`policy/access-control.yaml`, `policy/secrets.yaml`) parse under standard YAML 1.2.
- Each ADR has 7 sections (`## 1. Title` → `## 7. Reversal Conditions`).
- `<TOKEN_TBD>` placeholder grammar uses angle brackets + SCREAMING_SNAKE_CASE throughout.
- No files under `profile/` were touched.
- No file named `README.md` exists at repo root (confirmed).

---

## 5. Open-item summary

| Item | Location | Owner | Priority |
|---|---|---|---|
| Resolve `<ORG_OWNER_TEAM_TBD>` et al. | `policy/access-control.yaml` | `<ORG_OWNER_TEAM_TBD>` | High |
| Pin `aquasecurity/trivy-action` to SHA | `workflows/test.yml:319` | `<PLATFORM_MAINTAINERS_TEAM_TBD>` | High |
| Introduce `dependabot.yml` | `.github/dependabot.yml` (absent) | `<PLATFORM_MAINTAINERS_TEAM_TBD>` | Medium |
| Add `LICENSE` at repo root | `/LICENSE` (absent) | `<ORG_OWNER_TEAM_TBD>` | Medium |
| Author `CODEOWNERS` | `.github/CODEOWNERS` (absent) | `<ORG_OWNER_TEAM_TBD>` | Medium |
| Decide on `.gitleaksignore` here | root | `<PLATFORM_MAINTAINERS_TEAM_TBD>` | Medium |
| ADR on soft-fail posture | `DECISIONS/0004-*.md` (not yet authored) | `<PLATFORM_MAINTAINERS_TEAM_TBD>` | Low |
| `workflow-templates/` migration | new tree | `<PLATFORM_MAINTAINERS_TEAM_TBD>` | Low |
| Template-drift linter | tooling; not yet designed | `<PLATFORM_MAINTAINERS_TEAM_TBD>` | Low |
| Remove or reference orphan PNGs | `profile/banner.png`, `profile/krlabs-banner.png` | `<PROFILE_EDITORS_TEAM_TBD>` | Low |

*End of CALIBRATION_REQUIRED.*
