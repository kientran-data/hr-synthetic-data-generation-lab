# 06 — Multi-Source HR Data Generation and Export Contracts

## Why source generation is a distinct phase
A source adapter converts one coherent simulated HR world into an **operational system's view**, including its PKs, status vocabulary, snapshot date and ingestion behavior. It must never generate unrelated rows from scratch. Every adapter declares source-specific schema, input event families, primary grain, record-ID policy, observation cutoff and allowed absence/duplicate rules.

## End-to-end adapter API (conceptual)
```python
# Interface proposal, not executable code from an existing package.
class SourceAdapter:
    source_name: str
    schema_version: str
    def project(self, truth, as_of, config, rng): ...  # noiseless source rows + lineage
    def validate_clean(self, rows, crosswalk): ...
    def export(self, rows, output_mode): ...
```

## Generate count expectations from grain
For monthly payroll, expected base volume is the number of **eligible employment-episode × pay periods**, not `n_employees * fixed_months`. For attendance, expected volume comes from generated schedules, holidays, leave and active dates. For performance, use tenure/review eligibility. Record row counts per stage: ideal event count → clean projected count → noisy exported count.

## Fundamental rule
Each operational dataset is a **projection** of the canonical world at a specified observation time. The student's records use source-specific IDs. The generator maintains a separate, inaccessible mapping to canonical IDs. Granularity is explicit and stable.

## Source contracts
| Source | Student-facing table / file | Grain and observed primary key | Generated from | Key projection behavior |
|---|---|---|---|---|
| ATS | `Recruitment.csv` | application / `Recruitment_ID` | applications and application events | multiple applications per person, candidate name variants, withdrawn/rejected/on-hold states |
| HRIS | `Employees.csv` | episode snapshot / `Employee_ID` | employment, assignment, contracts, personal identity | one new Employee_ID per episode; current department/job/manager as-of snapshot; stale updates possible |
| Organization | `Departments.csv` | department / `Department_ID` | `gt_organization` | names, heads, budget, location; valid at snapshot |
| Job catalog | `Jobs.csv` | job / `Job_ID` | `gt_position`/job codes | grade and salary range references |
| Attendance | `Attendance.csv` | scheduled episode × date / `Attendance_ID` | schedules and attendance events | check-in/out, absences, overtime; weekends depend on calendar |
| Payroll | `Payroll.csv` | episode × pay period / `Payroll_ID` | compensation, eligibility, payroll events | correct prorating, earnings and deductions, payment timing |
| Leave | `Leave.csv` | leave request / `Leave_ID` | leave events | requested vs approved leave and holiday-aware chargeable duration |
| LMS | `Training.csv` | course enrollment / `Training_ID` | learning events | start/end, certification, cost and completion |
| Performance | `Performance.csv` | performance review / `Performance_ID` | performance events | reviewer relation, scores, recommendation, review timing |
| Exit | `Exit.csv` | exit event / `Exit_ID` | exit events | termination per episode, reason/type/notice period |

The above ten legacy-compatible tables mirror the **table set** in the reference repository; the generator's canonical model and matching files are proposed additions.

## Compatibility field rules
For `reference_compatible`, use **exact column names** from `02_REFERENCE_AND_INPUTS.md` and preserve appropriate column order when exporting to reference-shaped CSV. Some references use `Exit.csv` while their SQL table is `exit_data`, and `Leave.csv` maps to `leave_data`: document these translations in a machine-readable export manifest.

Critical mappings:
- `ATS.Recruitment_ID` → internal `application_id`; `Candidate_ID` is an **ATS-local** candidate profile key and can recur across that candidate's applications. Never assume Candidate_ID identifies the same person across systems or across ATS migrations.
- `HRIS.Employee_ID` → internal `employment_id` via mapping; two episodes of one person receive distinct IDs for compatibility.
- `Payroll.Employee_ID`, `Attendance.Employee_ID`, `Training.Employee_ID`, etc. should reference the **observed HRIS Employee_ID** valid for the episode when the source has a trusted internal integration. For harder matching exercises, configure some sources with their own employee identifiers and a secure crosswalk instead.
- `HRIS.Manager_ID` maps to the source Employee_ID of a manager whose **assignment is effective as-of the source snapshot**. If the manager record lags, simulate a bounded mismatch.
- `Employee.Hire_Date` represents episode start, not the person's first-ever hire date.
- `Employees.Salary` represents the designated as-of contractual rate; `Payroll.Basic_Salary` represents earned base pay for a specific period, which may be prorated.
- Use `Recruitment.Hiring_Status = 'Hired'` in compatibility mode **only after actual joining**, not merely offer acceptance.

## Recommended extended exports
For richer teaching, generate separate additive files rather than quietly changing the legacy schema:
- `ATS_Candidates.csv`: candidate profile versions with updated contact details.
- `ATS_Stages.csv`: stage history and timestamps per application.
- `HRIS_Employment_History.csv`: episode starts/exits, employee IDs, effective and recorded times.
- `HRIS_Assignment_History.csv`: department/job/manager as-of intervals.
- `HRIS_Contract_History.csv`: contract versions, types and effective dates.
- `Payroll_Adjustments.csv`: final pay, retroactive corrections, bonuses after exit.
- `Source_Ingestion_Log.csv`: source update delay, batches and incomplete transmissions.

## Example source observations (illustrative; not generated records)
| Source | Observed ID | Observed name | Internal truth link (instructor only) |
|---|---|---|---|
| ATS | APP0098 | Nguyen Thi Minh Anh | `person=P00042`, `application=A0098` |
| HRIS | EMP0031 | Nguyễn Thị Minh Anh | `person=P00042`, `employment=E002` |
| Payroll | PAY0081 | NGUYEN THI MINH ANH | `person=P00042`, `employment=E002` |
| Attendance | ATT0152 | Nguyễn T. M. Anh | `person=P00042`, `employment=E002` |

**Never include the right-hand truth-link column in student exports.** The table exists only to explain provenance to instructors and implementers.

## Row-count rules
- `count(recruitment)` = number of applications, **not** unique candidates.
- `count(employees)` = count of episode snapshots in legacy mode, **not** distinct people when rehires occur.
- `count(payroll)` ≈ number of eligible episode-months, plus separately classified adjustments if exported.
- `count(attendance)` = scheduled episode-days that emit observed records, not number of people or calendar days.
- `count(training)` = learning enrollments, including repeats by different employees.
- `count(exit_data)` = terminated episodes visible by the as-of date.

## Visibility filters
Export source snapshots using `source_observed_at` and `ingested_at` cutoffs. A canonical transfer on June 1 may not appear in an HRIS export taken on June 3 if its update is ingested on June 8. Do not rewrite canonical June history to imitate this delay.

## Cross-source consistency and intentional errors
Keep the noiseless projections referentially and temporally valid. Apply controlled source-specific errors afterward and record intentional errors with reason codes. A source mismatch should normally remain plausible and traceable; an arbitrary impossible birthdate or random employee ID is not a useful realism shortcut.

## Public row visibility after noise
All source outputs must be produced in this order: (1) canonical eligible events → (2) source-schema projection → (3) source IDs/crosswalk → (4) visible ingestion snapshot → (5) allowed formatting/staleness/missingness/duplicate rules → (6) source QA and role-safe release. A truncated snapshot is not the same as random record deletion; report both separately.
