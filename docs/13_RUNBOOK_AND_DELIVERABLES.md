# 13 — Build, Run, Export and Release Runbook

## Status and assumptions
These documents specify a proposed generator. **The following commands and folder names are contracts to implement**, not proof a functioning package exists. The first engineering milestone is a runnable 100-person fixture that passes tests.

## Proposed repository layout
```text
hr-synthetic-data-generator/
├── README.md
├── docs/                                  # this Markdown knowledge pack
├── configs/
│   ├── smoke.yaml
│   ├── beginner.yaml
│   ├── intermediate.yaml
│   └── advanced_matching.yaml
├── schemas/
│   ├── canonical/                        # private truth model JSON/SQL schema
│   └── sources/                          # ATS, HRIS, payroll, etc. contracts
├── src/hr_generator/
│   ├── cli.py
│   ├── config.py
│   ├── seeds.py
│   ├── models.py
│   ├── reference/                        # org, calendar, position
│   ├── simulators/                       # application, employment, work, pay, LMS
│   ├── projections/                      # ATS, HRIS, Payroll, Attendance, etc.
│   ├── noise/                            # lag, errors, formatting, lineage
│   ├── validation/                       # config, truth, source, realism, release
│   ├── benchmarks/                       # optional match pair sampler
│   └── export/                           # CSV, Parquet, manifest, packaging
├── tests/                                # unit, integration, scenarios, replay
├── examples/                             # safe SQL/Power BI examples
└── output/<run_id>/                       # generated; normally gitignored
    ├── student/raw/                       # ten reference-compatible source CSVs
    ├── student/extended/                  # optional richer histories
    ├── instructor/truth/                 # canonical tables and events
    ├── instructor/benchmarks/            # withheld labels, maps
    ├── instructor/oracles/               # truth/source-as-of metrics
    └── metadata/                         # manifest, schemas, validation, profiles
```

## Planned CLI lifecycle
```bash
python -m hr_generator validate-config --config configs/smoke.yaml
python -m hr_generator generate --config configs/smoke.yaml --output output/smoke
python -m hr_generator validate --run output/smoke --strict
python -m hr_generator export --run output/smoke --profile reference_compatible
python -m hr_generator benchmark --run output/smoke --predicate SAME_PERSON
python -m hr_generator package --run output/smoke --audience student
```
These are **interface specifications**. Do not run or document them as working until implemented with tests.

## Practical step-by-step operational protocol
1. **Freeze scenario**: scenario ID, date window, seed, dependency versions, source schema versions and policies. Record input checksums.
2. **Validate feasibility**: expected vacancies, applicant outcomes, target joins, initial episodes, exits, active counts and per-period coverage.
3. **Generate 100-person smoke fixture**: ensure examples of rejection, joining, promotion/transfer and rehire; avoid output quotas that are impossible at small scale.
4. **Run G1/G2**: unique entities, event order, history, work/pay eligibility, manager graph and arithmetic.
5. **Project clean sources**: first ATS, HRIS, Payroll, Attendance and Exit; then Departments, Jobs, Leave, Training and Performance. Create internal provenance for every exported physical row instance.
6. **Apply observation rules**: source lag, staleness, formatting, missingness and imports; log intentional corruptions privately.
7. **Validate G3–G5**: source schema, as-of cutoffs, noise bounds, distributions, KPIs and privacy scanning.
8. **Optional benchmarks**: generate after stable source exports, with correctly defined predicates and identity-disjoint splits.
9. **Export**: ten legacy-compatible CSVs; optional extended datasets and Parquet; student and instructor paths are physically separate.
10. **Manifest and replay**: emit versioned schema dictionary, achieved counts, quality profiles, validation results, SHA-256 hashes and deterministic content fingerprints; repeat a run with same config/code/env to verify equivalence.
11. **Package**: publish only approved student artifacts; treat internal mappings, labeled pairs and oracles as restricted files.
12. **Scale**: only after smoke passes, run beginner then intermediate profiles; compare achieved patterns with tolerance targets and report exceptions.

## Expected final files (after implementation)
| Product | Representative files |
|---|---|
| Reference-compatible public | `Employees.csv`, `Departments.csv`, `Jobs.csv`, `Recruitment.csv`, `Attendance.csv`, `Payroll.csv`, `Leave.csv`, `Training.csv`, `Performance.csv`, `Exit.csv` |
| Extended public | `ATS_Stages.csv`, `ATS_Candidates.csv`, `HRIS_Employment_History.csv`, `HRIS_Assignment_History.csv`, `HRIS_Contract_History.csv`, `Payroll_Adjustments.csv`, `Source_Ingestion_Log.csv` |
| Restricted instructor | `gt_*.parquet`, `gt_source_record_map`, `pairs_labeled.csv`, `truth_kpi_oracles.csv`, `instructor_expected_anomalies.csv` |
| Safe metadata | `manifest.json`, `validation_summary.json`, `observed_quality_metrics.csv`, `data_dictionary.md`, `scenario_effective.yaml` |

## Run manifest fields
Record `run_id`, scenario ID, seed and seed derivation version, generator code/commit, dependency lockfile hash, config hash, source schema versions, policy/calendar versions, as-of date, row counts by source/stage, target-vs-achieved deviations, errors/warnings, output checksums, semantic fingerprint, and optionally wall-clock generation time. A timestamped manifest must not be expected to have byte-identical hash across replays unless generated time is frozen.

## Example ChatGPT implementation prompts
- **P1 smoke:** “Build the Python code for stages 0–6 with 100 fictional persons and five source exports. Run pytest and show real artifacts, failed checks, and row counts.”
- **P2 full sources:** “Implement the remaining five source adapters and extended history options; verify reference column names and as-of behavior.”
- **P3 realism:** “Implement conditional scenario distribution diagnostics and controllable source noise without mutating canonical truth.”
- **P4 release:** “Package ten CSVs, manifests, safe data dictionary and restricted private oracles; run leak scans and deterministic replay.”
- **P5 applications:** “Build SQL/BI labs and an optional matching benchmark from the validated source release.”

## Publication and security checklist
- [ ] No leaked `person_id`, `employment_id`, `application_id`, private mapping or labeled answer columns in public files.
- [ ] All source files satisfy the declared export profile and import into the target SQL stack.
- [ ] Counts and KPI cutoffs are documented; 2025 is partial through August in the default scenario.
- [ ] All fictional contact details use safe domains/non-operational formats.
- [ ] Canonical truth, source IDs and optional pair labels remain in restricted storage.
- [ ] Source QA and accepted anomalies have documented rule IDs and test evidence.
- [ ] File hashes, schema versions, seed and reproducibility evidence are included.
