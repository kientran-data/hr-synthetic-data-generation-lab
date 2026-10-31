# 02 — Reference Repository, Input Assumptions and Compatibility

## Role of external reference in the end-to-end project
The repository below informs desired table names, SQL/Power BI compatibility, and example reporting scope. **It is not a requirements authority for random seeds, synthetic distributions, canonical data, noise rules, or expected generated counts.** Those originate from this lab's own scenario/configuration and must be validated at execution.

### Input/assumption inventory
| Input | Origin | Used by stage | Verification |
|---|---|---|---|
| SQL reference table names and column examples | public sample repository | export adapter | compare actual headers/SQL types |
| Fictional Vietnamese locale and reporting range | local scenario | people/calendar/business events | parse dates and safe locales |
| HR probabilities and outcomes | `07_SCENARIO_CONFIG.md` | event simulators | feasible eligibility + achieved distribution |
| Job/grade/pay bands | generated org catalog | position/compensation | valid bounds and dependencies |
| Source-specific lag/noise | scenario rules | projection and corruption | traceable transformation log |

Maintain a **compatibility matrix** rather than forcing the internal simulation schema to inherit reference-system modeling limitations.

## Verified reference
Repository: https://github.com/vikashGiri27/HR-Analytics-end-to-end-project

Checked against its public `README.md`, root `Database/` directory, and `Database/HR_Database_Setup.sql` on the default `main` branch on 2026-10-31. The reference uses MySQL, SQL/advanced SQL, Power BI, DAX, and Power Query. Its four dashboard pages are overview, recruitment/hiring, retention/attrition, and performance. It describes 2025 reporting as partial through August.

**This is a reference analytics repository, not our generator implementation.** Its existing CSV files do not establish a canonical identity or employee-lifecycle ground truth. The proposed tables prefixed with `gt_` in this kit are new.

## Existing table contracts from reference MySQL setup
| Reference table | Important fields (subset, exact spelling follows SQL setup) | Proposed observation grain |
|---|---|---|
| `employees` | `Employee_ID`, `First_Name`, `Last_Name`, `Hire_Date`, `Department_ID`, `Job_ID`, `Manager_ID`, `Employment_Status`, `Salary` | one HRIS employee/episode snapshot or versioned record |
| `departments` | `Department_ID`, `Department_Name`, `Location`, `Budget`, `Head_Employee_ID` | department snapshot |
| `jobs` | `Job_ID`, `Job_Title`, `Grade`, `Minimum_Salary`, `Maximum_Salary` | job reference |
| `recruitment` | `Recruitment_ID`, `Candidate_ID`, `Candidate_Name`, `Position`, `Department_ID`, `Application_Date`, `Interview_Date`, `Recruiter_ID`, `Hiring_Status`, `Joining_Date` | one application |
| `attendance` | `Attendance_ID`, `Employee_ID`, `Attendance_Date`, `Check_In`, `Check_Out`, `Working_Hours`, `Overtime_Hours`, `Attendance_Status`, `Shift` | employee × scheduled date |
| `payroll` | `Payroll_ID`, `Employee_ID`, `Payroll_Month`, `Basic_Salary`, `Allowances`, `Bonus`, `Tax`, `Deductions`, `Net_Salary`, `Payment_Date` | employment × payroll period |
| `leave_data` | `Leave_ID`, `Employee_ID`, `Leave_Type`, `Start_Date`, `End_Date`, `Total_Days`, `Approval_Status` | leave request |
| `training` | `Training_ID`, `Employee_ID`, `Training_Name`, `Provider`, `Start_Date`, `End_Date`, `Training_Cost`, `Certification_Status` | training enrollment |
| `performance` | `Performance_ID`, `Employee_ID`, `Review_Date`, `Reviewer_ID`, `KPI_Score`, `Productivity_Score`, `Attendance_Score`, `Teamwork_Score`, `Overall_Rating`, `Promotion_Recommendation` | performance review |
| `exit_data` | `Exit_ID`, `Employee_ID`, `Exit_Date`, `Exit_Reason`, `Exit_Type`, `Notice_Period`, `Final_Rating` | exit event |

The SQL setup has `exit_data.Employee_ID` marked unique, which limits it to one exit per *observed employee ID*. For rehiring in the new generator, either assign a new source `Employee_ID` per employment episode or use an extended versioned schema. A person can therefore have several HRIS IDs/episodes.

## Two output modes
### Mode A — `reference_compatible`
Export ten CSV files using the reference naming, column names, and MySQL-friendly types. Preserve the relationships expected by its SQL/DAX approach. Do not silently append `person_id`, `employment_id`, or ground-truth labels. Source IDs remain source-specific. Keep schemas compatible with the reference SQL where practical.

### Mode B — `extended_multisource`
Add ATS candidate profiles, interview stages, HRIS history, contract changes, org moves, source ingestion timestamps, as-of snapshots, source provenance and versioned public records. Extra files are additive, not assumed to exist in the GitHub repository.

## Important analytical differences
- The reference README describes `3,000` workforce records, `2,071` active, `929` exits, and `9,000` applications. Treat them as reference dashboard figures, not hard-coded statistical truths of a newly simulated world.
- Its `31%` attrition dashboard value appears to be a ratio of exits to the stated workforce count; for the course use a period-specific **attrition rate** with a properly defined denominator.
- `Recruitment_ID`, `Candidate_ID`, and `Employee_ID` must not be interpreted as canonical person identifiers.
- `Joining_Date` must correspond to an eligible joined application and employment episode; an offer or acceptance without a start is not a hire in a join-based metric.
- A current-state `employees` table can conceal past team/manager changes. Teach history through canonical assignment events and an optional versioned HRIS source.
- The README and SQL file are sufficient to align schemas, but no claim is made that original CSV distributions or embedded PBIX calculations were fully audited.

## License and attribution
Do not copy or redistribute the repository's original data, images, or SQL wholesale without checking its license and permissions. The synthetic generator should create new, independent fictional records and reference the project for educational inspiration.

## Implementation decision record
Choose compatibility mode explicitly for each release. Never put new private fields into the ten reference CSV files. Unsupported reference assumptions (such as uniqueness of source Employee_ID across rehire) must be solved through source episode IDs and additive history files, not by modifying historical truth.
