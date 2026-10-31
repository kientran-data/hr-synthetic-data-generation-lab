# 14 — End-to-End Implementation Roadmap and Definition of Done

## Phases, work packages and acceptance gates
| Phase | Priority | Scope | Done when |
|---|---|---|---|
| A0 | P0 | requirements, output modes, contracts, file naming | reference compatibility and data consumers documented |
| A1 | P0 | config validator, seed manager, feasibility planner | invalid configs fail; normalized config/hash stable |
| A2 | P0 | org, positions, calendar and fictional people | all IDs and reference constraints valid |
| A3 | P0 | recruiting + employment lifecycle | accepted+joined semantics, assignment, exit/rehire validated |
| A4 | P0 | work schedules, attendance, earned payroll, pay adjustments | no ineligible events; net pay arithmetic valid |
| A5 | P0 | ATS/HRIS/payroll/attendance/exit clean projections | source-grain and as-of rules validated |
| A6 | P0 | private source lineage and QA gates | all physical source rows traceable; no public truth leaks |
| A7 | P1 | departments/jobs/leave/training/performance and history adapters | ten source files plus optional histories validated |
| A8 | P1 | source observation lag, formatting and noisy imports | achieved defect metrics within configured bounds |
| A9 | P1 | distribution realism/calibration + KPI oracles | segment-specific quality report, reconciliation evidence |
| A10 | P1 | reproducible exports, packages, release metadata | documented checksums/semantic replay pass |
| A11 | P1 | optional entity matching benchmark and SQL/BI labs | label and answer-key isolation pass |
| A12 | P2 | scenario overlays, plugin-based future domains | adapters configurable without rewriting common core |

## MVP (must be actually runnable)
1. `configs/smoke.yaml` with feasible targets for 100 fictional persons.
2. Canonical org/person/application/employment/event/assignment/compensation state.
3. Several application outcomes; at least one transfer, exit and rehire; actual joins create episode IDs.
4. Five CSV outputs: `Recruitment.csv`, `Employees.csv`, `Payroll.csv`, `Attendance.csv`, `Exit.csv`.
5. Restricted source-row mapping; small private `SAME_PERSON`/`SAME_EMPLOYMENT` fixtures if benchmark mode enabled.
6. Unit tests for causal order, no overlaps, manager validity, pay arithmetic, and time eligibility.
7. A reproducible manifest, achieved counts, validation output and sample rows from **actual exported files**.

## Decisions to settle before code
- One or many companies; whether company IDs and department IDs can be reused across scenarios.
- Whether concurrent employment at distinct employers is enabled (default no same-employer overlap).
- Eligibility and visibility semantics for exited HRIS employees and post-exit payroll adjustments.
- Exact output schema version of the legacy-compatible CSVs and source primary-key rules.
- Calendar/policy ownership and fictional tax/pay formula; no legal compliance claims.
- Whether exact count targets are hard constraints or a probability tolerance.
- Which source errors are intentionally observable and which would be true generator bugs.
- How source row instances are keyed when deliberate duplicate imports repeat source record IDs.
- How canonical history/effective time differs from observed/ingested source snapshots.
- Whether the scenario supplies a student release, restricted instructor release, or both.

## Definition of done: end-to-end release
- [ ] A user can validate a config and generate a complete scenario with one documented CLI workflow.
- [ ] At least ten operational source CSVs are produced, correctly typed and importable.
- [ ] Every artifact has a known grain, PK/FK rules, date semantics and data dictionary.
- [ ] Business events and dependent operational records reconcile across source systems.
- [ ] Source lag, missingness and duplicates are documented and repeatable.
- [ ] Distribution diagnostics explain targets and achieved counts by meaningful segments.
- [ ] QA tests pass required blocking checks; exceptions are categorized and visible.
- [ ] Instructor-only truth, mappings, labels and oracles are excluded from public bundles.
- [ ] Output hashes, semantic fingerprints, code/config/env versions and scenario are recorded.
- [ ] Different seeds produce different *valid* outputs; same seed and pinned environment reproduces equivalent results.
- [ ] SQL/KPI/dashboard examples use explicit reporting periods and source-as-of cutoffs.

## Milestone proof requirements
Do not declare a milestone complete from a plan. Attach exact command(s) run, environment, files created, checks executed, PASS/FAIL summary and unresolved deviations. A specification-only repository should label all milestones `proposed` until implemented and tested.
