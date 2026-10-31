# 01 — Project Scope, Objectives and Deliverables

## Positioning
**HR Synthetic Data Generation Lab** is an engineering project that specifies and eventually implements a **complete, repeatable data-generation pipeline**, not solely canonical ground-truth creation or data matching. Its outputs are synthetic multi-source HR datasets suitable for data engineering, SQL, analytics, dashboards and entity-resolution learning. Ground truth is an implementation mechanism for consistency, lineage and evaluation.

**End-to-end contract:** `requirements → scenario + distributions → schemas + configuration → organization + population → lifecycle simulation → operational event generation → source projections → lag/noise → QA/calibration → exports/benchmarks → versioned release`.

## Goals and measurable success
| Objective | Expected result | Evidence required |
|---|---|---|
| End-to-end generation | coherent ATS, HRIS, Payroll, Attendance, Leave, Training, Performance and Exit data | generated files with schema/row counts |
| Business realism | plausible conditional outcomes across time, department, tenure and role | distribution, temporal and reconciliation reports |
| Configurability | sizes, periods, policies, rates, noise, output schemas and seeds controllable | validated YAML, scenario snapshots |
| Reproducibility | same config+code+environment+seed gives identical semantic outputs | deterministic tests, canonical hashes |
| Observability | explicit data quality imperfections with private lineage | source profiles, expected anomaly records |
| Analytical usability | documented grains, relational joins, useful HR metrics | dictionary, QA SQL, KPI oracle |
| Evaluation utility | difficult record linkage tasks with reliable hidden answers | benchmark pairs, holdouts, labels |
| Safe distribution | wholly fictional data, audience isolation | privacy and leakage checks |

## What is being generated
1. **Canonical simulation model (private):** fictional people, organizations, jobs, vacancies, applications, contracts, employment episodes, assignment/compensation history, and event ledger.
2. **Operational HR system observations (public/student):** ATS recruitment, HRIS employees, departments and jobs, attendance, payroll, leave, training/LMS, performance, exits.
3. **Optional richer source histories:** stage history, candidate profiles, HRIS assignment/contract history, payroll corrections and ingestion logs.
4. **Supporting artifacts:** schemas, profiles, summary statistics, QA findings, run manifest, scenario assumptions, benchmark tasks, private mapping/labels and KPI oracles.
5. **Learning assets:** SQL/Power BI/KPI examples and instructor assessments (downstream application, not a prerequisite for generating data).

## Default fictional scenario
- Vietnam-localized fictional employer group; `vi_VN`, VND and `Asia/Ho_Chi_Minh` are default assumptions, not mandatory settings.
- Reporting window: `2020-01-01` to `2025-08-31` inclusive; any current/partial-year comparison must acknowledge the cutoff.
- Reference-inspired **targets**: 8,000 fictional persons in the pool; 9,000 applications; 1,081 employment episodes already active at the window's start; 1,919 joins during the period; 929 exits by the cutoff; 3,000 total episodes and 2,071 active episodes **only if** the configured feasibility and event accounting hold.
- Multiple applications per person, transfer/promotion, rehire and manager changes should occur. A rehire creates a new `employment_id` for the same `person_id`.
- Target counts are not evidence of produced data. An engine must report achieved counts, violations and deviations rather than silently modifying reality to fit totals.

## Audience and deliverable separation
| Area | Audience | Examples | Must not contain |
|---|---|---|---|
| `output/<run_id>/student/raw/` | analysts/students | ten source CSVs | canonical IDs, labels |
| `output/<run_id>/student/extended/` | advanced learners | source histories, optional candidate pairs | answer keys |
| `output/<run_id>/instructor/truth/` | restricted | canonical Parquet/event history | real personal data |
| `output/<run_id>/instructor/benchmarks/` | restricted | mapping and labeled matching tasks | public exposure |
| `output/<run_id>/metadata/` | controlled | run manifest, profiles, schema docs, QA | instructor secrets in shared bundle |

## In scope / outside scope
**In scope:** design-to-export simulation, HR workflow coherence, versioned source formats, configurable controlled source imperfections, target tuning, quality assurance, release packaging, and analytics-oriented evaluation.

**Out of scope initially:** live HR APIs, legally compliant payroll taxation, labor-law advice, actual employee data, genuine performance prediction, credit/medical/biometric attributes, production identity verification, and real-world predictive claims.

## Delivery sequence
- **D0 — specification:** this Markdown pack and explicitly documented assumptions.
- **D1 — executable smoke:** 100-person deterministic fixture with five source CSVs, hidden mapping, tests and manifest.
- **D2 — full HR generation:** configurable simulation with 10 reference-compatible files and extended histories; measured QA and KPI oracles.
- **D3 — scenario/benchmark release:** multiple difficulty profiles, matching pairs, instructor release, documentation and grading utilities.

Read `03_END_TO_END_PIPELINE.md`, `07_SCENARIO_CONFIG.md`, `14_IMPLEMENTATION_ROADMAP.md` and `15_STAGE_CONTRACTS.md` before implementation.
