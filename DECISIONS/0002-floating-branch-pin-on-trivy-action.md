# ADR 0002 — Floating-branch pin on `aquasecurity/trivy-action@master`

## 1. Title

Formalize the action-pinning posture for `workflows/test.yml` and schedule remediation of the single floating-branch reference.

## 2. Context

`SPEC.md` §4 documents that every external GitHub Action referenced in `workflows/pr-check.yml` and `workflows/test.yml` is pinned to a **major-version tag** (e.g., `actions/checkout@v4`, `actions/setup-node@v4`, `actions/setup-python@v5`, `actions/github-script@v7`, `actions/upload-artifact@v4`, `codecov/codecov-action@v4`, `github/codeql-action/upload-sarif@v3`) — except one:

> `aquasecurity/trivy-action@master` (`test.yml:319`) — pinned to a **floating branch**, which means any push to the action's `master` branch is automatically consumed. Highest supply-chain risk in the repo. — `SPEC.md` §10.1

This is also the only pinning style in the repo where a compromise or breaking change to the upstream action is silently consumed on the next workflow run. Everywhere else, a malicious `@v4.0.1` tag retag would be needed; at `@master`, only a branch push is needed.

Additional relevant facts from `SPEC.md`:

- Trivy runs a filesystem scan and uploads SARIF to the GitHub Security tab (`test.yml:319-331`).
- The SARIF upload step is wrapped in `continue-on-error: true` (`test.yml:331`), so scan failures never fail the workflow (`SPEC.md` §10.3, §11).
- No Dependabot configuration exists in this repo (`SPEC.md` §4, §12.6), so there is no automated bump for any action — including this one — to graduate to a tag pin.

Three postures are possible:

- **(A) Pin to a major tag** (`@v0` / `@0` / `@master` → some `@v<N>`). Consumes Dependabot/Renovate-style updates with ecosystem-typical cadence. Matches the rest of the repo.
- **(B) Pin to a specific commit SHA** (e.g., `@a11a6e508aa00e3c8f0b27a3097bb49b1c5b48f2`). Gold-standard SLSA L3 posture. Requires a process to update.
- **(C) Keep `@master`** and accept the risk.

## 3. Decision

**Adopt posture (A) as the binding norm for this repo and schedule the Trivy reference to move from `@master` to the current latest major tag.**

Specifically:

1. The org-wide posture for all Actions referenced by templates in `workflows/` is: **major-version tag pinning minimum, commit-SHA pinning preferred for security-sensitive actions.**
2. `aquasecurity/trivy-action@master` at `workflows/test.yml:319` is reclassified as a **security-sensitive action** (it reads the entire filesystem and emits attestations consumed by GitHub code-scanning). It MUST move to commit-SHA pinning — posture (B) — rather than merely major-tag pinning — posture (A).
3. Until remediated, the existing `@master` reference is tolerated under a `<REMEDIATION_DEADLINE_TBD>` window tracked in `CALIBRATION_REQUIRED.md`.

## 4. Consequences

Positive:

- Eliminates the single highest-impact supply-chain risk documented in `SPEC.md` §10.1.
- Aligns the repo's declared standards (`CI_CD_BEST_PRACTICES.md` §3 — safe credential patterns, `continue-on-error` audit trail) with its dependency pinning discipline.
- Gives downstream KRL consumers who copy `workflows/test.yml` a safer starting point.

Negative:

- Requires a `<TRIVY_VERSION_TBD>` reference sweep at remediation time. If the action's `master` has introduced breaking changes since the last release, pinning to a specific SHA may surface behavior drift.
- Without Dependabot configured (see DECISION `0003` side-note on automation gaps, and `SPEC.md` §4), bumps become manual. A commit-SHA pin will require periodic security review to avoid staleness of the pinned SHA in the face of CVEs in the action itself.
- `continue-on-error: true` on the SARIF upload (`test.yml:331`) means even a correct pin does not produce enforcement — this ADR does **not** address the soft-fail posture. Track separately.

## 5. Alternatives Considered

- **Posture (A) — major tag only, no SHA** — rejected as insufficient for the security-sensitive class this action falls into. Kept as the baseline for non-security-sensitive actions.
- **Posture (C) — keep `@master`** — rejected. Violates the documented intent of `CI_CD_BEST_PRACTICES.md` and leaves the largest attack surface unaddressed.
- **Remove Trivy entirely and rely on CodeQL** — rejected; `github/codeql-action/upload-sarif@v3` at `test.yml:328` is a SARIF **consumer**, not a source. The filesystem scan at `test.yml:319` is the only producer.
- **Add Dependabot** — deferred; addressed in `CALIBRATION_REQUIRED.md`, not this ADR.

## 6. Affected Artifacts

- `<REPO_ROOT_TBD>/workflows/test.yml` — line `319` (the `@master` reference) and surrounding Trivy step (`:310-331`).
- `<REPO_ROOT_TBD>/policy/secrets.yaml` — documents that this action handles no secret but has read access to the entire filesystem and write access to GitHub Security findings via `GITHUB_TOKEN`.
- `<REPO_ROOT_TBD>/policy/dependency.yaml` — marked **Not Applicable** in `policy/README.md` because this repo ships no dependency manifest (no `package.json`, no `pyproject.toml`). The Action pin is governed by this ADR, not by a dependency-policy file.
- `<REPO_ROOT_TBD>/CALIBRATION_REQUIRED.md` — records the open remediation, the target commit SHA (`<TRIVY_SHA_TBD>`), and the deadline.
- Every downstream consumer repo that copies `workflows/test.yml` is transitively affected.

## 7. Reversal Conditions

This decision MUST be reversed (via a superseding ADR) if:

- The `aquasecurity/trivy-action` project publishes a policy of force-pushing `master` with breaking changes weekly such that commit-SHA pinning produces excessive stale-pin churn. (Unlikely; documented here for completeness.)
- An upstream governance change makes the action unmaintained (`<UPSTREAM_ARCHIVE_SIGNAL_TBD>`). In that case, both the pin and the action itself must be reconsidered — migrate to a maintained equivalent.
- A superior scanning mechanism (e.g., native GitHub Advanced Security filesystem scan) becomes available at the tier this org subscribes to, obviating the need for Trivy.
- The remediation target — commit-SHA pinning for security-sensitive actions — is formally downgraded org-wide (e.g., because Renovate with `pinDigests: true` is adopted and automates the SHA churn). In that case, the floor moves from "SHA pin" to "tag pin with digest via Renovate," which is materially equivalent and compatible with posture (B).
