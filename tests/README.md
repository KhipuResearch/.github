# tests/ — Not Applicable

## Status

**Not applicable.** This repository has no runnable test suite, and none is planned.

## Justification

Per `SPEC.md` §13.1:

> **Of the templates themselves** — None. There is no test for the YAML files in this repo, no `act`-based dry run, no schema validator config committed.

And per `SPEC.md` §1:

> There is no application code, no build, no runtime. Deliverable is documentation and YAML.

The artifacts in this repository are:

1. `profile/README.md` — Markdown rendered by GitHub as the org landing page. No executable logic. Testing it means human editorial review, not a programmatic suite.
2. `workflows/*.yml` — GitHub Actions YAML templates intended for consumers to copy (see DECISION [`0001-workflows-mislocated-at-root.md`](../DECISIONS/0001-workflows-mislocated-at-root.md)). They do not execute here (`SPEC.md` §2), and the places they *do* execute (consumer repos) are the appropriate locations for integration tests of those workflows.
3. `CI_CD_BEST_PRACTICES.md`, `CI_CD_FIXES_SUMMARY.md`, `SPEC.md`, `ARCHITECTURE.md`, `DECISIONS/*.md`, `policy/*.yaml` — documentation and control artifacts. No execution.

Adding a test suite for any of these would require inventing a new execution concern (e.g., a YAML schema checker, a Markdown-link checker, an `act`-based dry run) that is not presently scoped.

## What *would* be valuable future additions (tracked, not implemented)

See `CALIBRATION_REQUIRED.md` for the open remediation list. Candidates relevant to a future `tests/` directory:

- **YAML parse gate** for `workflows/*.yml` on every PR that modifies them.
- **`act`-based dry-run smoke** test that exercises each job under a synthetic checkout of a path-matching consumer tree.
- **Link-check** for `profile/README.md` external links (`krlabs.dev`, `checkout.krlabs.dev/...`, `info@krlabs.dev`, `support@krlabs.dev`, `github.com/KhipuResearch` per `SPEC.md` §16.1) — would need coordination with the "do not modify profile/README.md" operator instruction.
- **Template-drift detector** that compares consumer-repo copies of `workflows/*.yml` against the canonical versions here.

None of the above are implemented by this artifact pass. All are explicitly deferred and tracked.

## When to revisit this decision

Introduce a `tests/` suite if and only if one of the following happens:

1. DECISION [`0001-workflows-mislocated-at-root.md`](../DECISIONS/0001-workflows-mislocated-at-root.md) is reversed and `workflows/*.yml` begins executing against this repo. A YAML schema + `act` dry-run would then be justified.
2. A template-drift linter is prototyped and needs test fixtures.
3. A future ADR introduces a formal `workflow-templates/` migration (per DECISION `0001` reversal conditions), at which point `.properties.json` metadata would need validation.

Until then: no tests, no CI, no coverage threshold. The `tests/` directory exists only to host this justification so that future contributors do not assume its absence is an oversight.
