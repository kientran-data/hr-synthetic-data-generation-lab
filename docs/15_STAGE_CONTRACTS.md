# 15 — Stage Contracts, Inputs, Outputs and Generation Algorithms

## Standard stage contract
Every generator stage defines: `stage_id`, immutable input references, output schema, invariant checks, expected record count formula, per-stage sub-seed, downstream dependencies, metrics, error types and version. Stage outputs should be persistable/replayable to support debugging and regeneration.

## Stage details

### S0 — Scenario validation and planning
- **Inputs:** YAML/JSON scenario, source schema versions, policy/calendar versions.
- **Outputs:** normalized immutable config, `plan.json`, feasibility diagnostics, deterministic seed registry.
- **Algorithm:** parse/normalize → validate limits → derive candidate/episode capacity → confirm exact quotas/tolerances → compute content hash.
- **Checks:** dates ordered; ratios in [0,1]; no impossible joins relative to applications/capacity; 100-person smoke uses its own scaled targets.

### S1 — Organization, departments, jobs, positions and vacancies
- **Inputs:** company count, org depth, job bands, location/calendar, growth plan.
- **Outputs:** companies/departments/jobs/positions and date-effective vacancies.
- **Algorithm:** create DAG hierarchy → assign grade/pay bands → allocate positions by planned headcount → open vacancies by budget/season.
- **Checks:** no organization cycles; valid parent interval; positions reference valid orgs; no logically impossible grade/salary.

### S2 — Fictional person/applicant pool
- **Inputs:** population target, locale and name policy, safe contact policy, seed.
- **Outputs:** private `gt_person_master`, optional contact-history events.
- **Algorithm:** sample locale/name from fictional templates and distributions; assign opaque stable identity; intentionally allow homonyms.
- **Checks:** unique canonical keys; DOB range according to scenario; no real identifiers or deliverable contact endpoints.

### S3 — Recruitment and hiring lifecycle
- **Inputs:** applicant pool, vacancies, recruiting rates, role/season factors.
- **Outputs:** applications, stage events, offers, withdrawals, no-shows and joined applications.
- **Algorithm:** assign applicants to eligible openings → draw stage transitions conditioned on role/time → schedule outcomes → admit actual joins within capacity.
- **Checks:** optional stage timing monotonic; joined application has an actual start; accepted offer alone not a hire.

### S4 — Employment, contracts, assignments, managers, compensation, exit/rehire
- **Inputs:** joined applications, baseline episode quota, policies, existing positions.
- **Outputs:** employment episodes, contract/assignment/pay history, HR events and exits.
- **Algorithm:** create baseline active episodes → create episodes from joins → assign positions/managers/pay → simulate changes at eligible dates → sample exits → rehire creates new episode after an allowed gap.
- **Checks:** no overlapping primary assignments; manager acyclic; exclusive employment end date; explicit source application or baseline origin.

### S5 — Operational truth: work, leave, payroll, learning and reviews
- **Inputs:** employment/assignment/pay intervals, calendar, attendance/leave/training/review rules.
- **Outputs:** scheduled workdays, punches, leave requests/approvals, monthly pay calculations, LMS enrollments and performance reviews.
- **Algorithm:** materialize eligible day/period grids → draw conditional exceptions → calculate earned pay and eligible work hours → emit distinct operational events.
- **Checks:** work inside active episode; approved leave affects work; final-pay adjustments identified; `net_pay` reconciles.

### S6 — Clean source projection and lineage
- **Inputs:** canonical events and dates, source contracts, export profile, as-of cutoffs.
- **Outputs:** ATS/HRIS/Payroll/etc. clean records + restricted crosswalk for every physical row.
- **Algorithm:** select relevant events → apply source-specific grain/ID policy → render status/date fields → assign observed/ingested times → materialize as-of export snapshot.
- **Checks:** clean PK/FK contracts; source row count formula; crosswalk completeness; no canonical key in public schema.

### S7 — Observation timing, noise and corruption
- **Inputs:** clean records, source-specific rule graph, observation cutoffs, per-rule sub-seed.
- **Outputs:** visible source files with formatting differences, missingness, staleness, optional duplicate row instances; private anomaly log.
- **Algorithm:** visibility lag → source formats → selective missingness/staleness → optional import duplicates/adjustments → profile achieved defect rates.
- **Checks:** no underlying event mutation; expected QA category; proper source-row-instance mapping even for duplicated visible IDs.

### S8 — Validation and analytics calibration
- **Inputs:** canonical data, clean/noisy sources, expected count formulas, config.
- **Outputs:** `validation_summary.json`, achieved-vs-target metrics, KPI/reconciliation results, privacy/lineage scans.
- **Algorithm:** hard truth checks → clean projection checks → corruption bounds → segment distribution tests → oracle comparisons → release checks.
- **Checks:** all blocking failures prevent publication; warnings/expected defects enumerated.

### S9 — Optional matching benchmarks and analytic views
- **Inputs:** public source rows, private mapping, predicate-specific pair policy.
- **Outputs:** unlabeled student pairs, restricted labels/splits; optional curated analytic views.
- **Algorithm:** split persons first → sample positives/easy/hard negatives within splits → label via internal keys → redact/hide answers.
- **Checks:** predicate applicability, uniqueness, split isolation, no label leakage.

### S10 — Export, manifest and release
- **Inputs:** accepted run artifacts, audience policy, formats.
- **Outputs:** public/teacher bundles, data dictionary, manifest, SHA-256 and semantic replay fingerprints.
- **Algorithm:** schema-aware serialize → deterministic sort → scan leaks → compute hashes → write manifest → package distinct audiences.
- **Checks:** artifact existence, schema type, checksum, restricted-file absence in public package, repeat-run equivalence.

## Count planning formulas (illustrative)
- `episode_count = baseline_episodes + joined_applications` (if no other start origins are allowed).
- `active_episodes(as_of) = episodes_started_on_or_before(as_of) - episodes_exited_by(as_of)` with explicit exclusive-end rules and no double-counting.
- `application_count = count(submitted applications)`, not distinct candidates.
- `payroll_base_rows = count(eligible episode × pay period intersections)` before separately coded amendments.
- `attendance_base_rows = count(eligible schedule/attendance observations under source emission policy)`.
- `review_rows = count(eligible review events actually scheduled)`.

## Failure handling and stage reruns
Never turn an impossible state transition into a silent null or a row deletion. Fail with `stage_id`, offending key, rule ID and diagnostic context. On repair, rerun the stage and its downstream dependency graph from frozen upstream inputs. Record whether the new run ID is a new release or a reproducible rerun.

## Artifact confidentiality as part of stage design
Truth tables and complete crosswalks are internal S2–S10 objects. Public adapters must support explicit schema allowlists, including file metadata, preview rows and derived labels. A training-facing archive must not accidentally include the parent run directory containing `instructor/`.
