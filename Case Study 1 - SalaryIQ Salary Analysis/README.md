# 💼 Case Study 1: SalaryIQ: Data Analyst Salary Analysis

## 📚 Table of Contents

- [Business Task](#business-task)
- [Dataset](#dataset)
- [Questions and Solutions](#questions-and-solutions)



## Business Task

A bootcamp graduate just received their first data analyst job offer of $52,000 in Dallas, TX and has 48 hours to decide. The problem: there's no reliable, validated benchmark to know if the offer is fair.

Using 2,252 real Glassdoor job postings, I used SQL to answer the questions that matter: **Where does salary really come from? Which factors actually move the number?**

This analysis forms the data layer behind the SalaryIQ Tableau dashboard.



## Dataset

**Source:** Glassdoor Data Analyst job postings scrape (US market)  
**Rows:** 2,252  
**Key columns:** `job_title`, `salary_avg`, `salary_min`, `salary_max`, `state`, `sector`, `seniority_level`, `rating`, `company_size_label`, `skill_python`, `skill_sql`, `skill_excel`, `skill_tableau`, `skill_power_bi`

> The dataset was cleaned prior to this analysis — salary strings were parsed to numeric, -1 sentinels were converted to NULL, seniority was derived from job title keywords, and 9 skill flags were extracted from job description text.



## Questions and Solutions



**1. What is the overall market salary for data analysts?**

```sql
SELECT 
  ROUND(AVG(salary_avg), 0)  AS mean_salary,
  ROUND(MIN(salary_avg), 0)  AS min_salary,
  ROUND(MAX(salary_avg), 0)  AS max_salary,
  COUNT(*)                   AS total_postings
FROM `salary_data`
WHERE salary_avg IS NOT NULL;
```

#### Steps:

- Use `AVG()` to calculate the mean salary across all postings.
- Use `MIN()` and `MAX()` to capture the full range.
- Use `COUNT(*)` to confirm the sample size backing these figures.
- Filter with `WHERE salary_avg IS NOT NULL` to exclude the one unparseable row.

#### Answer:

| mean_salary | min_salary | max_salary | total_postings |
|---|---|---|---|
| 72,123 | 33,500 | 150,000 | 2,252 |

- The market average for a US data analyst role is **$72,123**.
- The spread is $116,500 wide — which is why unfiltered benchmarks are useless for negotiation.

---

**2. Which states pay data analysts the most?**

```sql
SELECT 
  state,
  COUNT(*)                      AS job_count,
  ROUND(AVG(salary_avg), 0)     AS avg_salary,
  ROUND(MIN(salary_avg), 0)     AS min_salary,
  ROUND(MAX(salary_avg), 0)     AS max_salary
FROM `salary_data`
WHERE state IS NOT NULL
  AND LENGTH(state) = 2
GROUP BY state
HAVING COUNT(*) >= 15
ORDER BY avg_salary DESC
LIMIT 10;
```

#### Steps:

- Filter to standard 2-letter state codes using `LENGTH(state) = 2` — this removes a small number of malformed location entries.
- Apply `HAVING COUNT(*) >= 15` to ensure at least 15 postings per state before drawing conclusions — small samples produce unreliable averages.
- Order by `avg_salary DESC` to surface the highest-paying markets.

#### Answer:

| state | job_count | avg_salary | min_salary | max_salary |
|---|---|---|---|---|
| CA | 626 | 88,432 | 40,000 | 150,000 |
| IL | 164 | 78,311 | 59,000 | 113,000 |
| CO | 88 | 73,619 | 62,000 | 91,000 |
| NJ | 86 | 73,000 | 39,500 | 106,000 |
| NY | 345 | 71,412 | 39,500 | 106,000 |
| AZ | 97 | 70,789 | 61,500 | 74,000 |
| NC | 90 | 68,111 | 57,000 | 76,000 |
| VA | 48 | 65,188 | 56,000 | 80,500 |
| WA | 53 | 64,755 | 51,000 | 78,000 |
| PA | 114 | 61,728 | 40,500 | 88,500 |

- California pays **$16,309 above the national average** — the strongest location premium in the dataset.
- A **$50,902 confirmed gap** exists between CA ($88K) and UT ($37K) for the same job title.
- Location is the single strongest salary signal in the data, confirmed by Kruskal-Wallis H = 563.97 (p = 5×10⁻¹¹³).

---

**3. How much does seniority level actually add to salary?**

```sql
SELECT 
  seniority_level,
  COUNT(*)                      AS job_count,
  ROUND(AVG(salary_avg), 0)     AS avg_salary,
  ROUND(AVG(salary_min), 0)     AS avg_floor,
  ROUND(AVG(salary_max), 0)     AS avg_ceiling
FROM `salary_data`
GROUP BY seniority_level
ORDER BY avg_salary DESC;
```

#### Steps:

- Group by `seniority_level` — a derived column created from job title keywords (Senior/Sr → Senior, Entry/Junior/Jr → Entry-Level, everything else → Mid-Level).
- Calculate `avg_floor` and `avg_ceiling` from `salary_min` and `salary_max` to show the full negotiable range per level, not just the midpoint.

#### Answer:

| seniority_level | job_count | avg_salary | avg_floor | avg_ceiling |
|---|---|---|---|---|
| Senior | 672 | 74,163 | 55,911 | 92,415 |
| Mid-Level | 1,245 | 71,506 | 53,724 | 89,288 |
| Entry-Level | 335 | 70,324 | 52,985 | 87,663 |

- The gap from Entry to Senior is only **$3,839** — far smaller than the location or sector effect.
- Confirmed by hypothesis testing: seniority explains less than 1% of salary variance (η²_H = 0.84%).
- This means **where you work matters more than how long you've worked**.

---

**4. Which industry sectors pay the highest average salary?**

```sql
SELECT 
  sector,
  COUNT(*)                      AS job_count,
  ROUND(AVG(salary_avg), 0)     AS avg_salary
FROM `salary_data`
WHERE sector IS NOT NULL
GROUP BY sector
ORDER BY avg_salary DESC
LIMIT 12;
```

#### Steps:

- Filter `WHERE sector IS NOT NULL` to exclude rows where Glassdoor's -1 sentinel was converted to NULL during cleaning.
- No minimum job count filter here — sector comparison is informative even with smaller sample sizes, as long as we note the caveat.

#### Answer:

| sector | job_count | avg_salary |
|---|---|---|
| Biotech & Pharmaceuticals | 33 | 83,106 |
| Real Estate | 12 | 80,917 |
| Arts, Entertainment & Recreation | 7 | 80,643 |
| Accounting & Legal | 43 | 75,221 |
| Information Technology | 570 | 74,247 |
| Health Care | 151 | 72,808 |
| Business Services | 523 | 72,273 |
| Manufacturing | 40 | 72,075 |

- **Biotech leads at $83,106** — $11,000 above the national average.
- A Government analyst (not shown — $64,319) benchmarking against a Biotech offer has already lost before the conversation starts. The $17,000 gap is structural, not negotiable.
- Once inside the private sector, the market is efficient: switching between Finance, IT, and Healthcare moves the needle by only ~$3,000.

---

**5. Does listing Python skills lead to a higher salary?**

```sql
SELECT
  CASE 
    WHEN skill_python = 1 THEN 'With Python'
    ELSE 'Without Python'
  END                           AS python_status,
  COUNT(*)                      AS job_count,
  ROUND(AVG(salary_avg), 0)     AS avg_salary
FROM `salary_data`
WHERE salary_avg IS NOT NULL
GROUP BY skill_python
ORDER BY skill_python DESC;
```

#### Steps:

- `skill_python` is a binary flag (1 = Python mentioned in the job description, 0 = not mentioned), extracted via regex during the data cleaning phase.
- Use `CASE WHEN` to turn the 0/1 flag into a readable label.
- Compare the average salary across both groups to measure the premium.

#### Answer:

| python_status | job_count | avg_salary |
|---|---|---|
| With Python | 633 | 74,633 |
| Without Python | 1,619 | 71,142 |

- Python commands a **$3,491 confirmed annual salary premium**.
- Statistically validated: Mann-Whitney U, p = 0.006 across 2,252 postings.
- Python signals technical depth to the labour market. The market pays for signal quality.

---

**6. How often is each skill mentioned in job postings?**

```sql
SELECT 'SQL'              AS skill, SUM(skill_sql)              AS postings, ROUND(100.0 * SUM(skill_sql)              / COUNT(*), 1) AS demand_pct FROM `salary_data`
UNION ALL
SELECT 'Excel',                      SUM(skill_excel),            ROUND(100.0 * SUM(skill_excel)            / COUNT(*), 1) FROM `salary_data`
UNION ALL
SELECT 'Statistics',                 SUM(skill_statistics),       ROUND(100.0 * SUM(skill_statistics)       / COUNT(*), 1) FROM `salary_data`
UNION ALL
SELECT 'Python',                     SUM(skill_python),           ROUND(100.0 * SUM(skill_python)           / COUNT(*), 1) FROM `salary_data`
UNION ALL
SELECT 'Tableau',                    SUM(skill_tableau),          ROUND(100.0 * SUM(skill_tableau)          / COUNT(*), 1) FROM `salary_data`
UNION ALL
SELECT 'R',                          SUM(skill_r),                ROUND(100.0 * SUM(skill_r)                / COUNT(*), 1) FROM `salary_data`
UNION ALL
SELECT 'Power BI',                   SUM(skill_power_bi),         ROUND(100.0 * SUM(skill_power_bi)         / COUNT(*), 1) FROM `salary_data`
UNION ALL
SELECT 'Machine Learning',           SUM(skill_machine_learning), ROUND(100.0 * SUM(skill_machine_learning) / COUNT(*), 1) FROM `salary_data`
UNION ALL
SELECT 'Spark',                      SUM(skill_spark),            ROUND(100.0 * SUM(skill_spark)            / COUNT(*), 1) FROM `salary_data`
ORDER BY postings DESC;
```

#### Steps:

- Use `UNION ALL` to stack one row per skill rather than writing a wide pivot table.
- Calculate `demand_pct` as the percentage of all postings that mention each skill.
- `SUM()` on a binary 0/1 column gives the count of rows where the skill is mentioned.

#### Answer:

| skill | postings | demand_pct |
|---|---|---|
| SQL | 1,362 | 60.5% |
| Excel | 902 | 40.1% |
| Statistics | 836 | 37.1% |
| Python | 633 | 28.1% |
| Tableau | 617 | 27.4% |
| R | 441 | 19.6% |
| Power BI | 246 | 10.9% |
| Machine Learning | 180 | 8.0% |
| Spark | 71 | 3.2% |

- SQL is the most demanded skill at 60.5% — but see Q7 for why demand ≠ premium.
- Python appears in less than 1 in 3 postings — but it carries the highest salary premium.

---

**7. Which skills actually raise salary — and which ones lower it?**

```sql
SELECT
  'Python' AS skill,
  ROUND(AVG(CASE WHEN skill_python   = 1 THEN salary_avg END) -
        AVG(CASE WHEN skill_python   = 0 THEN salary_avg END), 0) AS salary_premium
FROM `salary_data`
UNION ALL
SELECT 'Tableau',
  ROUND(AVG(CASE WHEN skill_tableau  = 1 THEN salary_avg END) -
        AVG(CASE WHEN skill_tableau  = 0 THEN salary_avg END), 0) FROM `salary_data`
UNION ALL
SELECT 'SQL',
  ROUND(AVG(CASE WHEN skill_sql      = 1 THEN salary_avg END) -
        AVG(CASE WHEN skill_sql      = 0 THEN salary_avg END), 0) FROM `salary_data`
UNION ALL
SELECT 'Excel',
  ROUND(AVG(CASE WHEN skill_excel    = 1 THEN salary_avg END) -
        AVG(CASE WHEN skill_excel    = 0 THEN salary_avg END), 0) FROM `salary_data`
UNION ALL
SELECT 'Power BI',
  ROUND(AVG(CASE WHEN skill_power_bi = 1 THEN salary_avg END) -
        AVG(CASE WHEN skill_power_bi = 0 THEN salary_avg END), 0) FROM `salary_data`
ORDER BY salary_premium DESC;
```

#### Steps:

- Use `CASE WHEN` inside `AVG()` to calculate the average salary separately for rows with and without each skill — all in one pass.
- Subtract the two averages to get the net premium (positive = higher pay, negative = lower pay).
- `UNION ALL` stacks one result row per skill.

#### Answer:

| skill | salary_premium |
|---|---|
| Python | +3,491 |
| Tableau | +2,921 |
| SQL | -1,009 |
| Excel | -1,540 |
| Power BI | -3,985 |

- **The market pays for signal quality — not skill frequency.**
- SQL appears in 60% of postings but carries a slight *penalty* — it's table stakes, not a differentiator.
- Excel signals generalist. Python signals technical depth. Listing both on a resume may send mixed signals.

---

**8. Does company size affect salary?**

```sql
SELECT 
  company_size_label,
  COUNT(*)                      AS companies,
  ROUND(AVG(salary_avg), 0)     AS avg_salary
FROM `salary_data`
WHERE company_size_label != 'Unknown'
GROUP BY company_size_label
ORDER BY company_size_label;
```

#### Steps:

- Filter out `'Unknown'` company size — these are the rows where Glassdoor's Size field was NULL after sentinel conversion.
- The `company_size_label` column maps Glassdoor's raw size strings (e.g. `"1001 to 5000 employees"`) to 7 ordered categories created during cleaning.

#### Answer:

| company_size_label | companies | avg_salary |
|---|---|---|
| 1. Micro (1-50) | 347 | 72,712 |
| 2. Small (51-200) | 420 | 72,521 |
| 3. Small-Mid (201-500) | 249 | 71,193 |
| 4. Mid (501-1K) | 211 | 71,988 |
| 5. Large (1K-5K) | 348 | 72,869 |
| 6. Very Large (5K-10K) | 97 | 74,201 |
| 7. Enterprise (10K+) | 375 | 69,957 |

- The total range across all 7 size tiers is only **$4,244** — effectively flat.
- Company size explains just **0.03% of salary variance** (η²_H = 0.03%, Kruskal-Wallis).
- Bigger company ≠ bigger paycheck.

---

**9. What does the entry-level salary landscape look like by sector?**

```sql
SELECT 
  sector,
  COUNT(*)                      AS openings,
  ROUND(AVG(salary_min), 0)     AS avg_floor,
  ROUND(AVG(salary_max), 0)     AS avg_ceiling
FROM `salary_data`
WHERE seniority_level = 'Entry-Level'
  AND sector IS NOT NULL
GROUP BY sector
HAVING COUNT(*) >= 3
ORDER BY avg_floor DESC;
```

#### Steps:

- Filter to `seniority_level = 'Entry-Level'` — derived from job title keywords during cleaning.
- Use `salary_min` as the floor and `salary_max` as the ceiling to show the negotiable range, not just the midpoint.
- `HAVING COUNT(*) >= 3` removes sectors with too few postings to be reliable.

#### Answer:

| sector | openings | avg_floor | avg_ceiling |
|---|---|---|---|
| Insurance | 6 | 72,833 | 95,500 |
| Manufacturing | 9 | 64,222 | 93,222 |
| Consumer Services | 3 | 58,000 | 95,333 |
| Government | 5 | 56,800 | 90,600 |
| Health Care | 23 | 55,870 | 91,130 |
| Business Services | 85 | 53,565 | 86,459 |
| Information Technology | 91 | 53,011 | 90,736 |

- Insurance offers the highest entry-level salary floor ($72,833) — despite being a less obvious target for analysts.
- IT has the most openings (91) but a lower starting floor ($53,011).
- The ceiling across sectors is surprisingly similar (~$87–96K) — the real difference is the floor.

---

**10. What are the best state and sector combinations for Python + SQL roles?**

```sql
SELECT
  state,
  sector,
  COUNT(*)                      AS openings,
  ROUND(AVG(salary_avg), 0)     AS avg_salary
FROM `salary_data`
WHERE skill_python = 1
  AND skill_sql = 1
  AND state IS NOT NULL
  AND LENGTH(state) = 2
  AND sector IS NOT NULL
GROUP BY state, sector
HAVING COUNT(*) >= 5
ORDER BY avg_salary DESC
LIMIT 10;
```

#### Steps:

- Filter to rows where **both** `skill_python = 1` AND `skill_sql = 1` — targeting roles that explicitly require both skills.
- `HAVING COUNT(*) >= 5` ensures the average is backed by at least 5 postings before ranking.
- Group by `state` and `sector` together to find the most lucrative specific combinations.

#### Answer:

| state | sector | openings | avg_salary |
|---|---|---|---|
| CA | Health Care | 10 | 106,750 |
| CA | Finance | 16 | 98,438 |
| CA | Information Technology | 64 | 97,477 |
| CA | Real Estate | 5 | 89,100 |
| CA | Manufacturing | 7 | 85,071 |
| IL | Information Technology | 7 | 83,429 |
| CA | Business Services | 25 | 82,860 |
| IL | Business Services | 5 | 78,000 |
| NY | Business Services | 31 | 76,629 |

- **California Healthcare + Python + SQL = $106,750 average** — the single highest-paying combination in the dataset.
- California dominates 7 of the top 10 spots — reinforcing location as the primary salary lever.
- If you're targeting remote roles, aim for California-based companies in Health Care or Finance while leveraging Python and SQL as your core skill signal.
