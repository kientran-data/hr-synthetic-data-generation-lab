# 08 — Source Observability, Latency, Noise and Data Quality Injection

## Placement in the pipeline
First generate correct business events, then project them into clean source records, then model observable differences. Introduced errors must be attributable to a source rule and must not change `person_id`/`employment_id`/event truth. This project deliberately generates *both normal business variation and observed data defects*; do not call a legitimate rehire or final payroll adjustment a data error.

## Observation and noise sequence
1. Compute event effective time and source availability time.
2. Materialize source records visible as of source extraction cutoff.
3. Apply source-local display conventions such as accents/date formats/case.
4. Apply controlled omissions, stale attributes and import duplicates using stable per-rule sub-seeds.
5. Emit instructor-only transformation trace with rule IDs, row-instance keys, before/after where permitted.
6. Re-profile exported data; compare achieved vs configured defect rates.

Support `clean`, `beginner`, `intermediate`, and `stress` levels. A clean run should still contain normal HR complexity: multiple applications, legitimate rehires and monthly time series.

## Principle
Introduce errors to **observations**, not to hidden canonical identity or lifecycle truth. A wrong HRIS date, stale department, or missing accent should remain correctable because the internal mapping and event history preserve what really happened.

## Noise taxonomy
| Noise category | ATS | HRIS | Payroll | Attendance | LMS/Performance | Validity rule |
|---|---|---|---|---|---|---|
| Formatting | unaccented names | capitalized surnames | UPPERCASE names | abbreviations | alternate transliterations | canonical person unchanged |
| Missing values | absent phone/email | missing address/middle name | null optional allowance | missing check-in | blank reviewer comments | no mandatory PK destruction by default |
| Stale attributes | old mobile | prior department | previous surname | old shift name | prior manager | tied to observable ingestion lag |
| Duplicates | duplicate application import | duplicate snapshot | corrected rerun | double punch import | repeat upload | intentional duplicates distinguishable from legitimate multiple events |
| Type differences | localized dates | uppercase country | month string `YYYY-MM` | `HH:MM` | categorical score labels | export schema parsable per contract |
| Cross-system identity | ATS-local candidate key | episode-specific employee key | payroll system's own key in advanced mode | device-specific badge key | LMS user alias | internal crosswalk tracks truth |
| Timing | applicant status backfill | late transfer | late adjustment | late check-out | delayed completion | valid actual event sequence survives |

## Controlled rule template
Each transformation needs a rule ID, scope, probability, seed/sub-seed, conditions, before/after transformation, impacted fields, expected QA outcome and whether it intentionally breaks source referential integrity.

Example:
```yaml
rule_id: name_strip_accents_ats
source: ATS
field: Candidate_Name
probability: 0.12
precondition: contains_vietnamese_diacritics
transform: unicode_remove_diacritics
instructor_reason: formatting_variant
changes_canonical_truth: false
```

For noisy emails, use reserved fictional domains such as `example.com`, `example.org`, or subdomains thereof and prevent any outbound messages. Avoid fabricated national identity card numbers that could collide with real identifiers. A single fictional person's DOB can be consistent in truth while an ATS observation has a deliberate plausible date typo.

## Distribution design
Noise should depend on source, historical era and operational workflow. For example, ATS may strip Vietnamese accents 15% of the time while payroll exports are more consistently uppercase. Use conditional probabilities rather than applying the same 5% random mutation to every column.

Set a base rate of rare hard negatives: two *different* people sharing surname, given-name initials, or DOB but having different canonical identities. Include truly identical observed names. Hard negatives must come from different persons, never be faked by changing hidden labels.

## Missingness mechanisms for teaching
- **MCAR-like:** uniformly lost optional field to teach basic missingness handling.
- **MAR-like:** missing mobile numbers concentrated in older ATS records because earlier forms did not require them.
- **MNAR-like scenario:** selectively unreported optional data driven by a simulated user decision; clearly mark as scenario assumptions, not empirical evidence.

## Data-quality scoring and expected anomalies
Track completeness, uniqueness, validity, consistency, timeliness and reconciliation rate. Separate:
1. Expected source defects deliberately injected by configuration.
2. True generator bugs that violate canonical integrity.
3. Legitimate business exceptions such as final payroll after exit.

Only category (1) belongs in an instructor-provided anomaly catalog. Do not use broad exceptions to mask category (2).

## Privacy and fairness
Use fictional names, fictional addresses or broad locales, reserved contact domains, and non-operational phone placeholders. Sensitive characteristics should be minimized. Do not generate actual ethnic, religious, medical or biometric claims. When age/gender variables are used for SQL education, frame them as synthetic analytical dimensions, not predictors or justifications for adverse hiring actions. Provide lessons on fairness, proxy variables, and data minimization without implying artificial patterns represent real population outcomes.

## Student/instructor isolation
Public rows may include `Recruitment_ID`, `Employee_ID`, salary, names, and observed timestamps. Restricted QA includes canonical IDs, exact corruption reason codes, and mappings. A `name_variant` field or hard-negative indicator in public data would leak benchmark answers and is therefore prohibited.

## Calibration and profile reports
Report defect rate by source, field and mechanism; missingness type; stale-record age; duplicate row-instance count vs duplicate business-event count; matching difficulty; and whether each intentional error preserves parseability and time semantics. No QA exemption may be used to conceal a broken world-model invariant.
