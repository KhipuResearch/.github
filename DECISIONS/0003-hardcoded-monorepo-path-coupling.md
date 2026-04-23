# ADR 0003 — Hardcoded `./Private IP/` monorepo paths in `workflows/*.yml`

## 1. Title

Accept the hardcoded `./Private IP/<subpackage>` coupling in workflow templates; do not abstract via `workflow_call` inputs at this time.

## 2. Context

`SPEC.md` §7.2 documents that every job in `workflows/test.yml` and each applicable step in `workflows/pr-check.yml` uses a literal `working-directory: ./Private IP/<subpackage>`:

> Every job uses `working-directory: ./Private IP/<subpackage>` (e.g., `test.yml:66, 138, 186, 267, 275, 281, 292, 353`). These paths assume the consumer checkout exposes a `Private IP/` directory containing `krl-dashboard/frontend`, `krl-dashboard/backend`, and `krl-premium-backend`. — `SPEC.md` §7.2

`SPEC.md` §8 restates this as the templates' implicit API contract:

> A directory tree rooted at `Private IP/krl-dashboard/frontend`, `Private IP/krl-dashboard/backend`, `Private IP/krl-premium-backend` relative to the checkout root.

`SPEC.md` §12.1 flags this as a failure mode:

> Consumer repos copying these workflows are pinned to the hardcoded `./Private IP/...` paths. Any repo that reorganizes its directory layout silently breaks unless those paths are patched. There is no linter or check that flags this drift.

The path fragment `Private IP/` is the actual working-tree directory name used in the KRL parent monorepo (documented in project memory and confirmed by the companion consumer repos `krl-dashboard/` and `krl-premium-backend/` under that root). It is not a secret or a redaction marker — it is the literal directory name that happens to be somewhat sensitive in appearance.

Three postures are possible:

- **(A) Accept the coupling.** Leave templates as-is. Consumers who mirror the monorepo layout work out-of-the-box; others patch the paths during copy.
- **(B) Abstract via `workflow_call` inputs.** Convert templates to reusable workflows with inputs such as `frontend-dir`, `premium-backend-dir`, `dashboard-backend-dir`. Requires also addressing the no-reusable-workflow finding from DECISION `0001`.
- **(C) Abstract via environment variables only.** Keep `on: pull_request` / `on: push`, but hoist paths into top-level `env:` block. Consumers override by editing once instead of ~8 times per file.

`SPEC.md` §7.3 notes neither workflow declares `workflow_call` or `workflow_dispatch`. `SPEC.md` §12.4 flags the `workflow_dispatch` gap separately. `CI_CD_BEST_PRACTICES.md:82` recommends adding `workflow_dispatch` as a best practice; the templates do not currently follow that guidance.

## 3. Decision

**Adopt posture (A) — accept the coupling — with explicit documentation.**

Reasoning:

1. Every current consumer of these templates is either (a) the same monorepo this `.github` repo sits alongside, or (b) a satellite repo that already has its own fully-independent `.github/workflows/` (enumerated in `SPEC.md` §14.1: `krl-data-connectors`, `krl-model-zoo`, `krl-frameworks`, `krl-causal-policy-toolkit`, `krl-geospatial-tools`, `krl-network-analysis`).
2. Per `SPEC.md` §14.1, those satellite repos maintain their CI independently — they do **not** copy `workflows/test.yml` from here. So the only live consumers that benefit from abstraction are the three `Private IP/<subpackage>` targets, which by construction share the exact layout the paths assume.
3. Posture (B) requires addressing the `workflow_call` absence (DECISION `0001` reversal conditions) and introduces input-schema maintenance. Not justified for a 3-consumer surface.
4. Posture (C) is a small improvement but adds no real portability — a consumer who must edit `env:` once vs. the 8 working-directory lines once is saving manual work that does not move the needle.

Under posture (A), the templates document (via `SPEC.md`, this ADR, and `ARCHITECTURE.md` §4) that copying requires either (i) the monorepo layout or (ii) a one-time patch pass.

## 4. Consequences

Positive:

- Zero additional surface in this repo. `workflows/pr-check.yml` and `workflows/test.yml` remain as captured by `SPEC.md`.
- Matches the actual consumer topology (in-monorepo use).

Negative:

- Any **new** consumer repo that does not fit the `Private IP/<subpackage>` layout must be warned (via `CONTRIBUTING.md` or `ARCHITECTURE.md` §4) to patch during copy.
- The "silent breakage" mode flagged in `SPEC.md` §12.1 is accepted — no lint or drift detection is introduced. A consumer that renames their subpackage after copying the templates will not get a warning until workflows fail.
- Incompatible with a future state where multiple non-monorepo KRL consumer repos might want to adopt these templates. In that state, reverse this ADR and adopt posture (B).

## 5. Alternatives Considered

- **Posture (B) — `workflow_call` reusable workflows with path inputs** — deferred. Requires also addressing `workflow_call` absence from DECISION `0001`; scope-creep for the current artifact pass. A consolidated "reusable-workflows migration" ADR is a cleaner vehicle.
- **Posture (C) — top-level `env:` abstraction** — rejected as minor-improvement-with-no-portability-gain.
- **Rename `Private IP/` upstream** — out of scope for this repo; affects the KRL monorepo root, not `.github/`.
- **Introduce a template-drift linter** — deferred; a `scripts/check-consumer-paths.sh` that consumer repos can opt into is a reasonable future addition but does not belong in the template set itself.

## 6. Affected Artifacts

- `<REPO_ROOT_TBD>/workflows/pr-check.yml` — working-directory references at `pr-check.yml:32, 111, 115` per `SPEC.md` §14.1.
- `<REPO_ROOT_TBD>/workflows/test.yml` — working-directory references at `test.yml:66, 138, 186, 267, 275, 281, 292, 353` per `SPEC.md` §7.2.
- `<REPO_ROOT_TBD>/ARCHITECTURE.md` §4 — restates the consumer contract.
- `<REPO_ROOT_TBD>/CALIBRATION_REQUIRED.md` — tracks `<NEW_CONSUMER_TRIGGER_TBD>` (the signal that would reverse this ADR).
- Downstream consumer repos: any copy of either YAML is transitively affected.

## 7. Reversal Conditions

This decision MUST be reversed (via a superseding ADR) if:

- A fourth or later consumer repo outside the `Private IP/<subpackage>` layout formally requests template adoption without hand-patching. Threshold is `<NEW_CONSUMER_TRIGGER_TBD>` (recommended: 1 such request — the minimum that proves the coupling has become a friction point).
- The KRL monorepo restructures away from `Private IP/` as a root-level directory name. At that point the existing paths break even for the in-monorepo consumers.
- DECISION `0001` is reversed (templates promoted to `workflow_call` reusable workflows). At that point input abstraction becomes cheap and posture (B) becomes the default.
- A template-drift linter is introduced org-wide (e.g., via a dedicated `workflow-templates/` migration from DECISION `0001`'s reversal conditions). At that point abstraction would be coupled with automated drift warnings and posture (B) is again the right call.
