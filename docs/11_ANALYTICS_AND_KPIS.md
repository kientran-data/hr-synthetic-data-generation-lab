# 11 — HR Analytics, KPI Oracles and Reconciliation

## Analytics is a validation and consumption stage
These metrics serve both (a) private oracle checks of the data-generation pipeline and (b) later SQL/BI learning. They do not define generation order. Every KPI calculation should be available as **canonical truth**, **clean source-as-of**, and **noisy observed source-as-of** when the distinction matters; differences are expected only under documented coverage/lag/noise rules.

## KPI check catalog
| Family | Generate from | Required consistency |
|---|---|---|
| Workforce | employment episodes + as-of effective time | starts - exits reconcile to active episodes |
| Recruitment | application events + joins | no offer-only records counted as hires |
| Payroll | eligible episodes + compensation + adjustments | arithmetic and pay period policies |
| Attendance | schedules + approved absences + punches | denominators exclude invalid non-workdays |
| Training/performance | eligible reviews/enrollments | temporal eligibility and distinct grains |

Metrics must show reporting period and `data_available_through`; partial-year 2025 is not a full-year comparison.

## General conventions
Every KPI must declare **grain**, **period**, **population**, **time zone**, **as-of cutoff**, **numerator**, **denominator**, and whether it uses canonical truth or observed source exports. Do not compare full-year 2024 with partial-year 2025 without aligned months or a clearly stated adjustment.

| KPI | Definition | Caution |
|---|---|---|
| Active headcount (episodes) | number of episodes satisfying `start_date <= as_of_date` and (`end_date` null or `as_of_date < end_date`) | rehires still count once as active episode if only one active episode allowed per person |
| Distinct active persons | distinct `person_id` among active episodes | may differ with parallel valid employment contracts |
| Applications | number of submitted application events in period | not unique applicants |
| Unique applicants | distinct canonical persons among applications in period | not directly known in student sources without linkage |
| Joined hires | number of new episodes with actual start date in period | offers and acceptances are not starts |
| Hiring conversion | joined applications ÷ submitted applications for a fully mature cohort | recent applications require censoring/decision-window handling |
| Offer acceptance | accepted offers ÷ offers issued | does not imply actual joining |
| Time to join | actual start date − application date for joined applications | distinguish time-to-fill (vacancy-based) |
| Exits | number of episodes whose last active day (`end_date - 1 day`) falls in the reporting period; use explicit `gt_exit_event` date where available | do not confuse the exclusive interval end with the last active day |
| Annual attrition rate | departures in calendar year ÷ average active headcount during year × 100 | use average of suitable snapshots; describe denominator methodology |
| Early-tenure exits | exits within first 12 months of an episode ÷ eligible starts/cohort | use cohort follow-up window, avoid right-censoring |
| Payroll expense | sum paid/earned compensation elements on selected accounting basis | distinguish accrued cost vs paid cash and adjustments |
| Attendance rate | attended scheduled workdays or hours ÷ eligible scheduled workdays or hours | avoid charging weekends/approved absence incorrectly |
| Overtime rate | approved overtime hours ÷ scheduled paid hours | check approved vs self-reported hours |
| Review coverage | episodes eligible and reviewed in defined window ÷ eligible episodes | latest review joins need as-of restrictions |
| Training completion | completed enrollments ÷ eligible enrollments | do not confuse with number of participants |
| Department turnover | applicable period exits attributed to effective department ÷ average headcount in that department | current HRIS department is not necessarily exit-time department |

## Attrition examples
Suppose a calendar-year period has 120 episode exits and average observed active headcount of 1,500. Then annual attrition = `120 / 1500 = 8%`. In contrast, `929 / 3000 ≈ 30.97%` is an **overall episode exit share** if 929 of 3000 historical episodes ended by a particular cutoff. These are different statistics.

## SQL pattern: correct active episodes from canonical truth (instructor)
```sql
-- ANSI-like example; parameter syntax varies by SQL engine.
SELECT COUNT(*) AS active_episodes
FROM gt_employment
WHERE start_date <= DATE '2025-08-31'
  AND (end_date IS NULL OR DATE '2025-08-31' < end_date);
```

## SQL pattern: monthly hires from canonical truth (instructor)
```sql
SELECT EXTRACT(YEAR FROM start_date) AS hire_year,
       EXTRACT(MONTH FROM start_date) AS hire_month,
       COUNT(*) AS joined_episodes
FROM gt_employment
WHERE start_date >= DATE '2020-01-01'
  AND start_date < DATE '2025-09-01'
  AND start_origin = 'APPLICATION_JOIN'
GROUP BY EXTRACT(YEAR FROM start_date), EXTRACT(MONTH FROM start_date)
ORDER BY hire_year, hire_month;
```

MySQL, SQLite, DuckDB and PostgreSQL differ in date and parameter syntax. This document uses illustrative portable SQL fragments; convert and test against the chosen engine.

## Student-facing reconciliation lab
The legacy-compatible `Employees.csv` represents current observed employment episodes, while `Recruitment.csv` represents applications. Counting applicants by `Recruitment_ID` would overstate unique candidates. Counting distinct names would undercount homonyms and overcount spelling changes. These are intended learning opportunities.

## Reporting design
Display `as-of` date and `data available through` prominently. For 2025 through August, use January–August 2024 vs January–August 2025, or disclose annualization assumptions. Never imply the generator's synthetic statistical associations are causal HR policy findings.
