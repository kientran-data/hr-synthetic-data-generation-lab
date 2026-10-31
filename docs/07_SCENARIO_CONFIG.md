# 07 — End-to-End Scenario Configuration, Distributions and Targets

## Config is the control plane for the whole pipeline
The YAML below determines **data generation**, not simply truth-table size: entities, time horizons, event probabilities, source views, quality defects, matching scenarios and file outputs. It is an example contract and must not be presented as generated data or tested software.

### Config lifecycle
1. Parse and validate types/enums/units; normalize dates, locales, rates and output modes.
2. Check feasibility of exact counts and conditional rates (vacancy capacity, eligibility windows, active-headcount accounting).
3. Create a stable, ordered scenario snapshot and content hash; derive sub-seeds by module/stable key.
4. Resolve effective business policies and freeze them for the run.
5. Generate components in dependency order, collecting achieved counts/diagnostics after each stage.
6. Export both intended targets and realized values; never claim a percentage target was satisfied without tolerance checks.

### Parameter ownership
| Parameter group | Owner | Affects |
|---|---|---|
| `period`, `population` | scenario | population, episodes, truncation |
| `recruitment`, `employment` | domain simulation | application/stage/join/exit transitions |
| `attendance`, `payroll`, `training`, `performance` | operational simulators | work schedules, compensation, learning, reviews |
| `observation` | source projection | as-of visibility and source delays |
| `noise` | quality injection | exported defects and missingness |
| `benchmark` | measurement | pair sampling, splits and labels |
| `exports` | delivery | CSV/Parquet, manifests and release boundaries |

## Approach
Every run accepts a declarative configuration. The YAML below is a **proposed generator contract**; it is not a claim that software or data already exists. Ratios are simulation targets, and implemented generators must report achieved counts and tolerance rather than silently forcing invalid events.

```yaml
project:
  name: hr-synthetic-data-generator
  scenario: baseline_reference_inspired
  seed: 42
  schema_version: "1.0.0"
  generator_version: "0.1.0-proposed"
  locale: vi_VN
  currency: VND
  timezone: Asia/Ho_Chi_Minh

period:
  start_date: 2020-01-01
  as_of_date: 2025-08-31
  end_exclusive: 2025-09-01

population:
  person_pool: 8000
  baseline_active_employment_episodes: 1081
  job_applications: 9000
  joined_applications: 1919
  exited_employment_episodes_by_as_of: 929
  target_total_employment_episodes: 3000
  target_active_episodes_as_of: 2071
  rehire_person_probability: 0.04
  multiple_application_person_probability: 0.30
  company_count: 2
  department_count: 12
  job_title_count: 45
  manager_ratio_target: 0.12
  concurrent_same_employer_episodes: false

recruitment:
  outcome_targets:
    joined: 1919
    rejected: 4535
    on_hold: 1388
    withdrawn: 1158
  minimum_days_application_to_join: 7
  allow_accepted_offer_no_show: true

employment:
  annual_transfer_probability: 0.08
  annual_promotion_probability: 0.07
  annual_salary_review_probability: 0.55
  annual_contract_renewal_probability: 0.22
  minimum_rehire_gap_days: 30
  baseline_start_before_window: true

attendance:
  workweek: [Mon, Tue, Wed, Thu, Fri]
  public_holiday_calendar: fictional_vi_training_calendar
  nominal_daily_hours: 8.0
  monthly_absence_probability: 0.025
  overtime_day_probability: 0.10

payroll:
  frequency: monthly
  effective_pay_proration: calendar_or_policy_based
  allow_final_payment_after_exit: true
  fictional_tax_rule: training_flat_band_v1

training:
  annual_enrollment_rate: 0.65
  completion_rate: 0.78

performance:
  review_frequency_months: 12
  first_review_minimum_tenure_days: 90
  target_reviewed_episodes: 2501

observation:
  export_mode: reference_compatible
  extra_extended_exports: true
  source_lag_days:
    ATS: {min: 0, max: 2}
    HRIS: {min: 0, max: 14}
    Payroll: {min: 0, max: 7}
    Attendance: {min: 0, max: 3}
    LMS: {min: 1, max: 21}
    Performance: {min: 1, max: 30}

noise:
  profile: intermediate
  record_missingness_probability: 0.02
  name_variant_probability: 0.12
  missing_middle_name_probability: 0.08
  email_change_probability: 0.05
  source_record_duplicate_probability: 0.01
  stale_profile_probability: 0.07

benchmark:
  predicates: [SAME_PERSON, SAME_EMPLOYMENT, SAME_APPLICATION, SAME_COMPANY]
  seed: 1729
  train_fraction: 0.70
  validation_fraction: 0.15
  test_fraction: 0.15
  positive_pairs_target: 12000
  negative_pairs_target: 12000
  hard_negative_fraction: 0.50
  publish_labels_to_students: false

exports:
  formats: [csv, parquet]
  restricted_truth: true
  include_manifest: true
  include_validation_report: true
```

## Count arithmetic and constraints
The baseline targets satisfy `1081 + 1919 = 3000` employment episodes and, **assuming exactly 929 episode exits**, `3000 - 929 = 2071` active episodes. This is an accounting identity under the configured event window, **not** proof of generated output. The 9,000 application outcomes total 9,000. A joined application creates exactly one new episode. Rehires are counted as *new episodes* from the 1,919 joins, not as another 1,919 distinct people.

`target_reviewed_episodes` may be infeasible under some timings or early-tenure requirements; validation must report and either adjust a documented target or reject impossible configs. Targets for other event counts must be computed from eligibility, never random fixed multipliers independent of employment histories.

## Suggested profiles
| Profile | Persons | Special purpose | Noise |
|---|---:|---|---|
| `smoke` | 100 | fast deterministic QA, lesson demos | low |
| `beginner` | 800 | simple SQL, clean keys, basic dashboard | low |
| `intermediate` | 8,000 | reference-inspired dashboards, changes and rehiring | medium |
| `advanced_matching` | 20,000 | difficult name collisions, late updates, changed contacts | high but controlled |
| `temporal_stress` | 5,000 | rehire, contract overlaps, backdated changes and corrections | targeted |

Size numbers indicate canonical person pools, not employment record counts. For small profiles, derive feasible application and employment count targets rather than reuse baseline absolute targets.

## Generator command interface (planned)
```bash
python -m hr_generator validate-config --config configs/smoke.yaml
python -m hr_generator generate --config configs/smoke.yaml --output output/smoke
python -m hr_generator validate --run output/smoke --strict
python -m hr_generator export --run output/smoke --profile reference_compatible
python -m hr_generator benchmark --run output/smoke --predicate SAME_PERSON
```

These commands are **desired CLI specifications**, not commands tested in an existing implementation. If building the generator, implement commands and automated tests before presenting them as executable.

## Scenario-level controls
Allow teacher-friendly scenario overrides: turnover by department and tenure, hiring seasonality, compensation by grade, manager-to-worker ratios, remote-work attendance, unpaid leave, hiring freeze, sudden staffing expansion, LMS participation, fairness auditing, and source ingestion failures. Record all overrides in the manifest.

## Statistical realism and calibration
Configure conditional distributions, not just global ratios. At minimum, examine joins by vacancy/role/season; exits by tenure/department; salary by grade/currency; overtime by shift; leave by calendar; training eligibility/completion by tenure. Compare sampled vs target values by segment; document tolerance, minimum cell size and rare-event behavior. Never fit exact headline totals by creating an illegal employment event.

## Suggested scenario overlays
`hiring_freeze`, `rapid_growth`, `high_turnover`, `payroll_adjustments`, `late_hris_updates`, `advanced_matching`, and `temporal_stress` are optional overlays, not mandatory concrete YAML files until implemented. Each override must be recorded in the run manifest and have at least one scenario-specific acceptance test.
