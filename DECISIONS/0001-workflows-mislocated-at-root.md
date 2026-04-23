# ADR 0001 — `workflows/` lives at repo root, not `.github/workflows/`

## 1. Title

Accept the root-level `workflows/` directory as a deliberate consumer-template location; do not promote to `.github/workflows/`.

## 2. Context

Per `SPEC.md` §2, the YAML files `workflows/pr-check.yml` (172 lines) and `workflows/test.yml` (367 lines) sit at the repository root under `workflows/`, **not** at the GitHub-recognized auto-execute path `.github/workflows/`. Because this repository is itself named `.github` (the org-meta repo for `KR-Labs`), the conventional auto-exec path would be `.github/.github/workflows/`, which is empty.

Consequences of the current placement, per `SPEC.md` §2 and §7.3:

- GitHub Actions does **not** schedule these workflows on push or pull-request events against this repo.
- They function purely as **copy-source templates** for downstream KRL consumer repos (`krl-dashboard`, `krl-premium-backend`, etc., per `SPEC.md` §14.1).
- No README or inline doc states this explicitly; interpretation is inferred from content (the hardcoded `./Private IP/` paths) and absence of self-CI.

Two alternative postures are possible:

- **(A) Codify template status** — keep the location, document it explicitly as "consumer-copy templates, not active CI."
- **(B) Remediate** — either move the files to `.github/workflows/` (which would cause them to auto-execute against a repo with no source code to test, immediately failing on the `./Private IP/` working-directory lines), or relocate them under `workflow-templates/` (the GitHub-recognized location for org-level sharable templates).

Option (B-move-to-active) is actively hostile: the hardcoded monorepo paths (`SPEC.md` §7.2) guarantee failure. Option (B-workflow-templates) is attractive long-term but requires author-metadata conversion (icon, properties, `.properties.json`) that is out of scope for this Tier U artifact pass.

## 3. Decision

Retain `workflows/` at the repository root. Treat it as the **canonical consumer-copy template library** until a follow-up ADR migrates it to the GitHub `workflow-templates/` convention.

The placement is now formally accepted as intentional and documented in `ARCHITECTURE.md` §2.1 and this ADR.

## 4. Consequences

Positive:

- Zero downside from auto-execution failure, because no execution happens.
- Clear separation between "this repo's self-CI" (absent by design, per `<NOT_APPLICABLE_TOKEN_TBD>`) and "templates consumers copy."

Negative:

- Non-obvious to new contributors — the absence of a README alongside `workflows/` means the template relationship must be learned from `SPEC.md` and this ADR.
- Not discoverable via the GitHub "New workflow" UI (which sources `workflow-templates/` not `workflows/`).
- Changes to templates do not produce immediate signal; drift between template intent and consumer copies is not automatically detected.

## 5. Alternatives Considered

- **Move to `.github/workflows/`** — rejected. Would auto-execute against a repo with no application code, causing the `./Private IP/<subpackage>` `working-directory` lines in `test.yml:66, 138, 186, 267` to fail on every run. See DECISION `0003` for the path-coupling analysis.
- **Move to `workflow-templates/`** — deferred. This is the GitHub-recognized org-template location and would improve discoverability, but requires an accompanying `.properties.json` metadata file per template (`filename.properties.json`) and iconography. Track as follow-up in `CALIBRATION_REQUIRED.md`.
- **Delete and require consumers to maintain independently** — rejected. Loses the central-standardization benefit documented in `CI_CD_BEST_PRACTICES.md`.

## 6. Affected Artifacts

- `<REPO_ROOT_TBD>/workflows/pr-check.yml`
- `<REPO_ROOT_TBD>/workflows/test.yml`
- `<REPO_ROOT_TBD>/ARCHITECTURE.md` (declares the template zone)
- `<REPO_ROOT_TBD>/CALIBRATION_REQUIRED.md` (tracks the deferred `workflow-templates/` migration)
- `<REPO_ROOT_TBD>/policy/ci-authority.yaml` — marked **Not Applicable** in `policy/README.md` because no workflow here has authority over this repo's own execution surface.

## 7. Reversal Conditions

This decision MUST be reversed (via a superseding ADR) if any of the following occur:

- A consumer repo explicitly requests reusable workflows via `on: workflow_call` (`SPEC.md` §7.3). In that case, `workflows/*.yml` would need `workflow_call` triggers, declared `inputs:`, and `secrets:` blocks, and migration to either `.github/workflows/` (for same-repo reuse) or remain in a consumer-accessible path with input abstraction from DECISION `0003`.
- GitHub deprecates the "copy-source template" pattern in favor of `workflow-templates/`-only sharing.
- Auto-discovery of template drift becomes a requirement (e.g., a periodic audit shows consumer copies have diverged >3 major versions from the templates here). In that case, either move to `.github/workflows/` with reusable-workflow semantics, or adopt a separate tooling pipeline (e.g., Renovate presets) to enforce synchronization.
- The `<CONSUMER_REPO_COUNT_TBD>` of consumer repos copying these templates grows beyond the `<DRIFT_THRESHOLD_TBD>` where manual synchronization becomes infeasible.
