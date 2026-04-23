# ARCHITECTURE — `.github` (KRL org profile + shared workflow templates)

> **Status:** Control-plane architecture reference for the `KR-Labs/.github` meta-repository.
> **Scope:** Documentation-and-YAML only. No application runtime, no build system, no package manifest.
> **Source-of-truth companion:** [`SPEC.md`](./SPEC.md) — forensic specification (412 lines, commit `9995b3ff5b64fc8e9f1a3e77d1d23affa7f440b9`).

---

## 1. Purpose

The `.github` repository inside a GitHub organization plays two roles that are fused in a single tree:

1. **Organization profile surface** — the file `profile/README.md` is rendered by GitHub as the public landing page at `https://github.com/KR-Labs`. This is a GitHub rendering convention, not a CI behavior. See `SPEC.md` §16.1.
2. **Shared CI/CD workflow-template library** — the directory `workflows/` contains GitHub Actions YAML that downstream KRL consumer repos copy into their own `.github/workflows/` trees. These files do **not** execute against this repo because they are not located at `.github/workflows/` here. See `SPEC.md` §2 (architectural caveat) and §7 (reusable-workflow classification).

There is no third concern. No services are provisioned, no artifacts are produced, no tests are run in this repo.

---

## 2. Content topology

```
.github/                           # repo root (this is the GitHub "dot-github" meta-repo)
├── .gitignore                     # 10 bytes — ignores .DS_Store only
├── SPEC.md                        # 412 lines — forensic specification (source of truth)
├── ARCHITECTURE.md                # this file
├── CALIBRATION_REQUIRED.md        # deltas between SPEC and implemented artifacts
├── CI_CD_BEST_PRACTICES.md        # 389 lines — org standards doc (unchanged; informational)
├── CI_CD_FIXES_SUMMARY.md         # 326 lines — historical fix log, 2026-01-12 (unchanged)
├── DECISIONS/                     # Architecture Decision Records (7-section format)
│   ├── 0001-workflows-mislocated-at-root.md
│   ├── 0002-floating-branch-pin-on-trivy-action.md
│   └── 0003-hardcoded-monorepo-path-coupling.md
├── policy/                        # Tier U control artifacts (YAML)
│   ├── README.md                  # scope + NA justifications for dependency/ci-authority
│   ├── access-control.yaml        # who can merge, review, release
│   └── secrets.yaml               # secret inventory for the workflow-template surface
├── profile/                       # PUBLIC org landing page — DO NOT MODIFY
│   ├── README.md                  # 162 lines — GitHub-rendered org profile
│   ├── KRLabs_WebLogo.png
│   ├── banner.png                 # orphan asset per SPEC §15
│   └── krlabs-banner.png          # orphan asset per SPEC §15
├── tests/
│   └── README.md                  # NA justification — no runnable tests in this repo
├── .github/
│   └── workflows/
│       └── README.md              # NA justification — this repo has no self-CI
└── workflows/                     # CONSUMER TEMPLATES (copy-source; NOT auto-executed here)
    ├── pr-check.yml               # 172 lines — PR fast-feedback suite
    └── test.yml                   # 367 lines — full push/PR test suite
```

### 2.1 Three topological zones

| Zone | Directory | Executes here? | Consumers |
|---|---|---|---|
| Public profile | `profile/` | GitHub renders `README.md` on org page | Any visitor to `github.com/KR-Labs` |
| Consumer-copy templates | `workflows/` | **No** — off the `.github/workflows/` auto-exec path | KRL repos: `krl-dashboard`, `krl-premium-backend`, and others listed in `SPEC.md` §14.1 |
| Control / governance docs | `SPEC.md`, `ARCHITECTURE.md`, `DECISIONS/`, `policy/`, `CI_CD_*.md` | N/A — documentation | Humans and automated auditors |

### 2.2 Zones that would normally exist but do not

- `src/` — no application code (per `SPEC.md` §1).
- `tests/` as executable suites — no runtime to test (see `tests/README.md`).
- `.github/workflows/` with active CI — empty by deliberate scoping (see `.github/workflows/README.md`).
- `dependabot.yml` — absent per `SPEC.md` §4; not introduced by this control artifact stack.
- `LICENSE` at repo root — absent despite SPDX `Apache-2.0` headers in both workflow YAMLs (`SPEC.md` §14.4).

---

## 3. Artifact relationships

```
                        ┌────────────────────┐
                        │    profile/         │──► github.com/KR-Labs (org page)
                        │    README.md        │
                        └────────────────────┘
                                  ▲
                                  │  DO NOT MODIFY
                                  │  (per explicit operator instruction)
                                  │
┌──────────────┐    defines    ┌──┴────────┐    governs     ┌────────────────┐
│   SPEC.md    │──────────────►│ ARCHITEC- │───────────────►│  DECISIONS/    │
│  (forensic   │               │ TURE.md   │                │  (7-section    │
│   baseline)  │               │           │                │   ADRs)        │
└──────┬───────┘               └──┬────────┘                └────────┬───────┘
       │                          │                                  │
       │                          │                                  │
       │         constrains       ▼                                  │
       │                     ┌────────────┐                          │
       └────────────────────►│  policy/   │◄─────────────────────────┘
                             │  *.yaml    │    reconciles ADR outcomes
                             └──────┬─────┘    into enforceable controls
                                    │
                                    │  references
                                    ▼
                             ┌────────────┐
                             │ workflows/ │──► copied into consumer KRL repos'
                             │  *.yml     │    own .github/workflows/
                             └────────────┘
                                    ▲
                                    │
                             ┌──────┴───────────┐
                             │ CI_CD_BEST_PRAC- │
                             │ TICES.md         │  (informational — standards
                             │ CI_CD_FIXES_     │   that the YAMLs embody)
                             │ SUMMARY.md       │
                             └──────────────────┘
```

- `SPEC.md` is the forensic ground truth. Every other document in the control stack must be consistent with it or declare the drift in `CALIBRATION_REQUIRED.md`.
- `ARCHITECTURE.md` (this file) gives a stable mental model; `DECISIONS/` record individual tradeoffs.
- `policy/*.yaml` encodes enforceable controls where they apply; it explicitly marks N/A where they do not (dependency, CI authority).

---

## 4. Consumer contract (what this repo promises downstream)

1. **`profile/README.md` remains stable** except for intentional editorial updates. This artifact stack **does not** modify it.
2. **`workflows/*.yml` files are reference templates, not executing CI.** Consumers who copy them inherit three known constraints:
   - Hardcoded path coupling to `./Private IP/krl-dashboard/frontend`, `./Private IP/krl-dashboard/backend`, `./Private IP/krl-premium-backend` (see `SPEC.md` §7.2 and DECISION `0003`).
   - Floating-branch pin on `aquasecurity/trivy-action@master` (see `SPEC.md` §4 / §10.1 and DECISION `0002`).
   - No `workflow_call` or `workflow_dispatch` — event-triggered only (`SPEC.md` §7.3, §12.4).
3. **No secrets, no build artifacts, no release surface.** The secret inventory in `policy/secrets.yaml` is the full list of authentication material that touches this repo or its templates, bounded by `SPEC.md` §5.3.
4. **Mislocation of `workflows/` vs `.github/workflows/` is intentional** (DECISION `0001`). Consumers must copy, not fork CI execution.

---

## 5. Change control

Changes to items in this repo fall into three classes:

| Change class | Example | Gate |
|---|---|---|
| Profile content | Edit `profile/README.md` | Owner review; NOT touched by Tier U artifact generation |
| Workflow template | Bump an Action major version in `workflows/test.yml` | ADR update + policy reconciliation; downstream consumers must re-copy |
| Control artifacts | Update `policy/access-control.yaml` | New ADR if a 7-section decision is reversed; otherwise minor revision |

A template change that would break the consumer contract (e.g., removing the `./Private IP/` hardcoding) MUST be preceded by a new ADR superseding DECISION `0003`.

---

## 6. Known gaps (as of this writing)

Tracked in `CALIBRATION_REQUIRED.md`. High-level summary:

- No `LICENSE` file at repo root despite SPDX headers in YAMLs.
- No `dependabot.yml` — Action version bumps are manual.
- No `act`-based or YAML-schema CI for the templates themselves.
- Orphan PNG assets in `profile/` (`banner.png`, `krlabs-banner.png`) are not referenced by `profile/README.md` per `SPEC.md` §15.
- `workflows/pr-check.yml` soft-fails most substantive checks with `|| true` (`SPEC.md` §12.2) — policy posture not yet formalized.

---

*End of ARCHITECTURE.*
