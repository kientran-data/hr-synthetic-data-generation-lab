# 09 — End-to-End Validation, Quality Gates and Realism Checks

## Validate the complete pipeline, not only hidden ground truth
Run gate checks at config parse, entity generation, event simulation, clean projections, noisy observations, benchmark generation and release packaging. Every check must specify severity, denominator, allowed tolerance and evidence file. QA also includes **realism checks** on conditional patterns and file-level usability.

| Stage gate | Essential assertions | Outcome |
|---|---|---|
| G0 Config | schema, units, feasibility, version, seed | reject invalid scenario |
| G1 Base population | unique IDs, org structure, sensible age/role/grade policy | reject incoherent world |
| G2 Lifecycle | joins, exits, eligibility, rehire, interval integrity | hard stop |
| G3 Clean sources | schema, primary/foreign keys, crosswalk, as-of cuts | hard stop |
| G4 Noisy sources | targeted defects/lag, no unauthorized truth changes | tolerate documented expected defects only |
| G5 Analytics | KPI reconciliation, appropriate denominators and cohorts | reject contradictory outputs |
| G6 Benchmarks | labels, valid predicates and identity-disjoint splits | hard stop on leakage |
| G7 Release | all ten expected files, documentation, schemas, hashes, role isolation | do not publish until passed |

## Additional test types
- **Statistical:** row counts by source and time, stratified event distributions, pay-band/tenure associations, seasonality and missingness rates with configured tolerances.
- **Property-based:** random valid config seeds preserve core invariants; edge cases at interval boundaries, leap days, no-shows, zero tenure, rehire and manager changes.
- **Cross-artifact:** source files load in DuckDB/SQLite; historical dimensions and columns match the documented reference export profile.
- **Privacy/lineage:** no realistic contact endpoints, no canonical IDs or label hints in student deliverables, every exported row instance privately traceable.
- **Replay:** two identical runs have matching **content hashes** after excluding intentional volatile metadata (e.g., wall-clock `generated_at`); manifest may include a timestamp and therefore need a separate reproducible content fingerprint.

## Philosophy
Validate the **truth layer first**, then noiseless source projections, then source-specific noisy exports, then benchmark labels. These are different contracts: noise may intentionally cause source inconsistency, but must never mutate canonical identities or event causal order.

## Critical acceptance checks
| ID | Invariant | Severity | What to assert |
|---|---|---|---|
| GT-01 | Person IDs unique | blocker | `count(person_id)=count(distinct person_id)` |
| GT-02 | All foreign keys resolved | blocker | no orphan applications, contracts, assignments, events |
| GT-03 | Valid employment intervals | blocker | episode `end_date > start_date` or open-ended |
| GT-04 | Valid state machine | blocker | no join without eligible successful application; baseline exceptions explicit |
| GT-05 | No duplicate primary assignments at same time | blocker | intervals do not overlap within an episode |
| GT-06 | Manager relationship valid | blocker | manager active and no reporting cycles as-of date |
| GT-07 | Rehire creates a new episode | blocker | distinct `employment_id` for each rejoin |
| GT-08 | Schedules and attendance eligible | blocker | work dates within episode and matching calendar rules |
| GT-09 | Payroll period eligible | blocker | period overlaps episode; post-exit adjustments specially coded |
| GT-10 | Payroll arithmetic correct | blocker | `net = earned base + allowances + bonus - tax - deductions` within rounding tolerance |
| GT-11 | No ordinary training/performance before work starts | blocker | actual event dates valid; exceptions explicit |
| SRC-01 | Source PK unique except declared duplicate scenarios | error | duplicate rate matches configured scope |
| SRC-02 | Source map total coverage | blocker | every exported physical row instance has a distinct private map entry |
| SRC-03 | No hidden truth leakage | blocker | scan all student exports for prohibited columns and labels |
| SRC-04 | Timing lag matches allowed policy | error | source observed/ingested timestamps within configured bounds |
| BM-01 | Pair labels agree with truth mapping | blocker | labels recomputed from endpoints and requested predicate |
| BM-02 | No cross-split identity leakage | blocker | each canonical person belongs to only one split |
| RUN-01 | Reproducibility | blocker | same config, seed and version yield identical sorted checksums |
| RUN-02 | Manifest complete | blocker | counts, SHA-256 hashes, seed, version, schema and validation present |

## Suggested test pseudocode
```python
assert len(person_ids) == len(set(person_ids))
assert all(row.person_id in persons for row in employments)
assert all(row.start_date < row.end_date for row in employments if row.end_date)

for ep in employments:
    assignments = sorted(primary_assignments(ep), key=lambda x: x.valid_from)
    assert all(a.valid_to <= b.valid_from
               for a, b in zip(assignments, assignments[1:]))

for row in attendance_truth:
    ep = employment_by_id[row.employment_id]
    assert ep.start_date <= row.work_date
    assert ep.end_date is None or row.work_date < ep.end_date

for row in payroll_truth:
    ep = employment_by_id[row.employment_id]
    assert row.period_start < row.period_end
    assert intervals_overlap((row.period_start, row.period_end),
                             (ep.start_date, ep.end_date or DATE_MAX)) \
           or row.payment_kind in APPROVED_AFTER_EXIT_ADJUSTMENT_KINDS
    assert money_close(row.net_pay, row.base_pay + row.allowances +
                       row.bonus - row.tax - row.deductions)
```

The above is **design pseudocode**; implement concrete types, code paths, and tests before executing.

## Matching quality verification
For every pair, obtain left and right `person_id`, `employment_id`, `application_id`, and employer from the internal mapping and recompute the label. Verify the intended positive/negative counts by predicate, pair uniqueness under canonical order, hard-negative proportion and disjoint entity splits. Do not sample negatives from impossible cross-split endpoints.

## Analytical oracles
Instructors should get gold-standard headcount snapshots, hires, exits, payroll totals and attendance aggregates computed directly from canonical truth. Student SQL is graded against the **appropriate source-as-of definition**, not always against omniscient truth: deliberately stale HRIS snapshots may produce legitimately different reported numbers.

## Reconciliation reports
Each run emits:
- `validation_summary.json` with executed checks, pass/fail, severity and counts.
- `observed_quality_metrics.csv` with missingness, duplicates, staleness and field validity by source.
- `truth_kpi_oracles.csv` with cohort and period metadata.
- `manifest.json` with output hashes and generator provenance.
- `instructor_expected_anomalies.csv` with injected noise reason and record keys.

## Two kinds of fail-fast
- **Hard stop:** impossible truth lifecycle, missing mapping, hidden truth leakage, invalid deterministic reproduction.
- **Accept as teaching challenge:** deliberately stale department, unaccented names, noncritical missing fields, observed duplicates with a documented rule.

## Data realism review
Assess distributions **conditional on meaningful factors**, e.g. hire rate by position, exit hazards by tenure, pay band by grade, overtime by shift. Avoid presenting unsupported demographic correlations as real-world facts. Include time-window completeness notes, especially for partial-year 2025, and test truncated-period comparisons.

## Release decision
Any blocker (`GT-*`, missing lineage, leakage, invalid exported schemas, or failed semantic replay) prevents publication. Expected source defects are acceptable only when the active noise profile explicitly authorizes them and the private anomaly log explains the rule. Give users both a machine-readable validation summary and a short human-readable QA report.
