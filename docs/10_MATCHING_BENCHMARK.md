# 10 — Optional Entity Matching Benchmark and Hidden Evaluation

## Place within the full project
Record matching is an **optional downstream product** generated after source observations and private lineage have been validated. It is not the core purpose of the HR generator and must not dictate public source keys or business event logic. Baseline dataset generation should work with benchmarks disabled.

### Benchmark inputs and outputs
Input: observed source rows + restricted source-to-entity mapping + pairing policy + independently derived matching seed. Output: unlabeled public pair tasks, restricted labeled answer key and reproducible evaluation splits. Public source data remains usable for SQL/BI even when no pairing tasks are exported.

## Purpose
Build benchmark tasks where students infer record linkage from noisy ATS/HRIS/payroll/attendance/LMS records. A benchmark can be correct only if the intended matching predicate is specified. Similar-looking data is not automatically the same entity.

## Required predicates
| Predicate | Definition | Example: same person with two jobs |
|---|---|---|
| `SAME_PERSON` | same canonical `person_id` | positive (1) |
| `SAME_EMPLOYMENT` | same non-null canonical `employment_id` | negative (0) when episodes differ |
| `SAME_APPLICATION` | same non-null canonical `application_id` | negative (0) when submissions differ |
| `SAME_COMPANY` | same employer organization identity in relevant context | negative (0) if companies differ |

**Important:** if either endpoint has no applicable entity (e.g. ATS-only application without employment), then `SAME_EMPLOYMENT` is **not applicable**, not automatically negative. Filter these pairs or use a separate `label_status=UNDEFINED` outside binary training sets. Define company at the appropriate hierarchy level, not “same department name.”

## Private mapping contract
`gt_source_record_map` holds one row for each observed record, keyed for physical row instances by `(run_id, source_name, source_record_id, source_row_instance_id)`, with nullable canonical IDs and source observation time. Every benchmark endpoint must be resolvable to exactly one physical source row instance (including import duplicates). The map stays outside student datasets.

### Example labels (instructor only)
| Left | Right | SAME_PERSON | SAME_EMPLOYMENT | Explanation |
|---|---|---:|---:|---|
| `ATS:APP0098` | `HRIS:EMP0031` | 1 | 1 | application led to episode E002; SAME_EMPLOYMENT applies here only because the application maps to joined episode E002 |
| `HRIS:EMP0010` | `HRIS:EMP0031` | 1 | 0 | same person rehired, episodes E001 and E002 |
| `HRIS:EMP0031` | `HRIS:EMP0440` | 0 | 0 | two different people with similar names |
| `Payroll:PAY0081` | `Attendance:ATT0152` | 1 | 1 | observations from same episode |

## Pair-generation algorithm
1. Create canonical groups for each predicate from internal mappings.
2. Generate positives from multiple systems, within-system duplicates, and cross-year observations; cap pairs per high-frequency employee so one long-tenure employee does not dominate.
3. Generate easy negatives by deterministic random sampling across different people.
4. Generate **hard negatives** using observed name-token overlap, DOB similarity, matching department, plausible contact collisions and confusing dates; verify canonical non-match.
5. Add explicit `same person / different episode` negatives for `SAME_EMPLOYMENT`.
6. Assign identity-disjoint train/validation/test splits *before sampling final pairs*. For binary pairs, both endpoints must be from persons allocated to the same split; do not let a person appear on both sides of different splits.
7. Deduplicate `(predicate, left, right)` using canonical endpoint ordering.
8. Generate a withheld labeled answer key; publish only input records and unlabeled pair tasks to students.

## Difficulty tiers
- **Easy:** exact normalized email and full name; different person negatives with distant names.
- **Medium:** missing middle name, accents, abbreviations, renamed email, stale department.
- **Hard:** high-similarity names, same DOB, source lag, rehire, identical initials and missing contacts.
- **Expert:** temporal predicates and multiple companies, application unjoined vs joined, overlapping source versions.

## Evaluation protocol
Report precision, recall, F1, exact-match accuracy for balanced instructional sets, and PR-AUC when scored probabilities are available. On realistic imbalanced candidate pairs, prioritize precision-recall metrics and calibration. For identity graph clustering, add cluster-level measures; pairwise metrics alone can hide transitive-closure errors. Report scores separately by predicate, source-pair type, difficulty and rehire status.

Never evaluate on matching pairs built from records used in training for the same underlying person. In addition to entity-disjoint splits, create an optional temporal holdout while documenting the difference between future-observation generalization and unseen-person generalization.

## Leakage checklist
Reject public fields derived from canonical IDs, hidden group hashes, truth labels, internal event IDs, split hints, or deterministic sequential keys that directly reveal links. Even filenames and metadata should not contain the withheld pair labels. Only the instructor package should include `gt_source_record_map` and `gt_match_pair_labels`.

## Student assignment
Build a pair scorer with normalized Vietnamese names, stable source keys, and timestamp compatibility; compare exact matching, fuzzy matching and an optional ML model. Include an error analysis on false positives arising from two different people with similar names and false negatives arising from legitimate rehire or renamed contact details.

## Key design constraint
For duplicate visible source-record IDs, resolve endpoint ambiguity using a benchmark-visible stable row-instance locator only if it does **not** leak private identity. Use a distinct instructor key for each physical source row instance. Report which identifier is used in pairing files and guarantee uniqueness/traceability in the public pair task.
