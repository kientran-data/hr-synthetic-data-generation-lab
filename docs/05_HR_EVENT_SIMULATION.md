# 05 — HR Event Simulation and Business Rules

## Generation dependency, not just the state machine
This file defines the **HR business simulator** in phases 4–5 of the end-to-end pipeline: positions/vacancies must exist first; applications may lead to joins; employment creates eligibility for work, payroll, leave, reviews and exits; source exports come afterward. State machines protect causal consistency; distributions in `07_SCENARIO_CONFIG.md` determine *how often* valid transitions occur.

## Conditional simulation knobs
| Process | Draw depends on | Examples of constraints |
|---|---|---|
| Vacancy creation | org growth, approved positions, role and season | capacity/grade consistency |
| Applicant arrival | vacancy age, role, recruiting seasonality | application timestamp in window |
| Stage progression | role, candidate pool, configured stage rates | order of stages and no impossible joins |
| Offer/no-show | candidate stage, open position, start-date availability | accepted offer need not lead to join |
| Transfer/promotion | tenure, assignment, org eligibility | close old assignment, keep episode |
| Exit / rehire | tenure, company, season, policy | new episode on rehire, no same-employer overlap by default |
| Schedule/attendance | work calendar, leave, start/end, shift | no ordinary schedule outside active episode |
| Payroll | earned work, compensation intervals, period policy | prorating and reconciled net pay |
| Training/reviews | eligibility, tenure, job, team | review period and reviewer validity |

All probabilities are synthetic design choices, not evidence about real worker behavior or protected populations.

## State machines
### Candidate/application
`SUBMITTED → SCREENING → INTERVIEW → OFFERED → ACCEPTED → JOINED`, with allowed terminal alternatives `REJECTED`, `WITHDRAWN`, `CANCELLED`, or `NO_SHOW`. `ON_HOLD` is an intermediate state that can later resume. **Only `JOINED` creates a started employment episode.** The presence of an offer or accepted offer alone is not sufficient.

### Employment episode
`PLANNED → ACTIVE → EXITED`, with optional pre-start `CANCELLED`. Rehire is **not** `EXITED → ACTIVE` on the same `employment_id`; create a new episode associated with the same `person_id` after a new join event. Moves, promotion, probation completion, parental leave and temporary absence change attributes or availability without creating a new episode.

### Contract and assignment
An active episode can contain multiple sequential contract versions, compensation intervals and primary job assignments. A transfer closes the previous assignment interval and opens the next; it does not rewrite an old department in historical records.

## Allowed causal dependencies
| Event | Required precondition | Required effect |
|---|---|---|
| `application_submitted` | candidate and open position exist | application record created |
| `application_joined` | successful candidate process, actual start date | `gt_employment` episode created |
| `employment_started` | effective start, eligible assignment/contract | payroll and work schedule eligibility begins |
| `contract_renewed` | existing valid episode | new contractual interval, previous version closed |
| `promoted` / `transferred` | episode active | new assignment, optionally new compensation interval |
| `manager_changed` | both relevant episodes active and valid | new manager assignment interval |
| `leave_approved` | eligible active episode, leave interval valid | schedule/absence state reflects leave |
| `employment_ended` | episode active | episode closes; subsequent routine schedules stop |
| `rehired` | a genuine earlier exit, new start | new employment ID linked to same person |

## Mandatory temporal invariants
1. `application.submitted_at <= interview_at <= offer_at <= joined_at` for stages that occur; do not require skipped optional stages.
2. For a joined in-window application, `employment.start_date = actual_join_date`, with separate planned vs actual dates.
3. `employment.end_date > employment.start_date` using exclusive-end convention.
4. Contracts and assignments must not create a primary work relationship outside the episode. Pre-start signing may occur before start, but validity starts no earlier than the employment start unless explicitly designed otherwise.
5. The combination `(employment_id, work_date)` should have at most one primary schedule; no ordinary scheduled work after episode end.
6. Overtime is conditional on attendance and approved rules; absence is not simultaneously coded as normal onsite presence except a documented split-day case.
7. Monthly payroll is produced only for periods overlapping an eligible episode; prorate starts and ends. Final payment or adjustment after exit is allowed if `payment_kind` makes the relationship explicit.
8. A review's `reviewed_at` must not precede the work/assessment period, and its reviewer must be valid as of the relevant event; allow documented external or late review corrections separately.
9. A manager relationship cannot create a management cycle, including via transfers and manager rehires.
10. Concurrent employment is disabled by default for a single employer, though multi-company concurrent episodes can be introduced as a controlled scenario.
11. Source ingestion can lag behind the effective event; this creates **staleness**, not a change in canonical truth.

## Worked fictional lifecycle
| Date | Canonical event | Entity effect |
|---|---|---|
| 2022-01-01 | Starts Company A | `P00042`, episode `E001` |
| 2024-06-30 | Final active workday at Company A | `E001` has exclusive `end_date=2024-07-01` |
| 2025-03-10 | Applies for Data Analyst, Company B | application `A0098` |
| 2025-04-15 | Accepted offer | application pending start |
| 2025-05-01 | Starts Company B | new episode `E002`, linked to `P00042` |
| 2025-08-10 | SQL course completed | learning event belongs to `E002` |
| 2025-08-31 | As-of reporting date | `E002` active |

**Identity predicates:** `SAME_PERSON(E001,E002)=1`; `SAME_EMPLOYMENT(E001,E002)=0`; `SAME_COMPANY(E001,E002)=0` if these episodes belong to different companies.

## Realism scenarios worth simulating
- Candidate applies three times; only the third application leads to a join.
- Employee transfers from Sales to Finance in July; June payroll attributes costs to Sales under the selected accounting rule.
- Promotion changes grade/salary effective on the first day of a month, but HRIS updates 10 days later.
- Employee resigns then rejoins the same employer after six months with a new HRIS ID.
- A rejected applicant has a highly similar name to a different successful employee.
- Payroll amendment arrives after termination; it is not a new period of employment.
- Leave overlaps a public holiday; count chargeable leave days using the applicable work calendar, not just date subtraction.

## Scenario-specific law and policy
Contract categories, holidays, tax parameters, maximum working hours, termination notices, and overtime formulas vary by geography and date. The generator's default rules are **fictional training assumptions** and must be labeled as such. Do not claim legal compliance with Vietnam or any other jurisdiction without independent legal validation.

## Simulator design contract
For each event family, implement `eligible(state, date)`, `sample_transition(context, rng)`, `apply(event, state)`, `validate(event, before, after)` and `emit_truth_rows(event)`. Support deterministic replay and correction events without silently editing already-exported source facts. Provide tests for skipped optional recruiting stages, final-pay adjustments, backdated changes and partially observed reporting windows.
