# 🌿 Case Study 2: Indonesia CO₂ Emissions Analysis

## 📚 Table of Contents

- [Business Task](#business-task)
- [Dataset](#dataset)
- [Questions and Solutions](#questions-and-solutions)



## Business Task

Indonesia is not Southeast Asia's highest per-capita CO₂ emitter but it is the region's **largest absolute driver of emissions** due to scale, rapid economic growth, and deep coal dependency. This analysis uses SQL to quantify that claim: how fast are emissions growing, what's driving them, and how does Indonesia compare to its neighbours?

This analysis extends the findings of my [Southeast Asia CO₂ presentation](https://github.com/chimkeninasal/southeast-asia-co2-analysis) using SQL instead of Power BI.



## Dataset

**Source:** Our World in Data CO₂ and Greenhouse Gas Emissions  
**Rows:** 50,191 (filtered to country-level data; aggregated regions excluded)  
**Key columns:** `Country`, `year`, `co2`, `coal_co2`, `oil_co2`, `gas_co2`, `cement_co2`, `flaring_co2`, `co2_per_capita`, `co2_growth_prct`, `share_global_co2`, `population`, `gdp`  
**Coverage:** 255 countries and regions, 1750–2023

> Note: Queries below filter to `Country = 'Indonesia'` or a defined set of Southeast Asian countries. Aggregated regional rows (e.g. `'Asia'`, `'World'`) were excluded — they have no `iso_code` in the raw data.



## Questions and Solutions

---

**1. What are Indonesia's 10 highest CO₂ emission years on record?**

```sql
SELECT 
  year, 
  ROUND(co2, 2)                                                AS co2_mt,
  ROUND(co2 - LAG(co2) OVER (ORDER BY year), 2)               AS yoy_change_mt
FROM `co2_data`
WHERE Country = 'Indonesia'
  AND co2 IS NOT NULL
ORDER BY co2 DESC
LIMIT 10;
```

#### Steps:

- Use `LAG(co2) OVER (ORDER BY year)` to access the previous year's value and calculate the year-on-year change in absolute terms (million tonnes).
- `LAG()` is a window function — it looks back one row within the ordered partition, which here is the full Indonesia time series sorted by year.
- Filter `WHERE co2 IS NOT NULL` to exclude the pre-1889 rows where Indonesia's emissions data does not exist.
- Order by `co2 DESC` to surface the peak years, then apply `LIMIT 10`.

#### Answer:

| year | co2_mt | yoy_change_mt |
|---|---|---|
| 2022 | 737.07 | +117.45 |
| 2023 | 733.22 | -3.85 |
| 2019 | 653.79 | +59.69 |
| 2021 | 619.62 | +11.40 |
| 2020 | 608.22 | -45.57 |
| 2018 | 594.10 | +37.16 |
| 2017 | 556.94 | +16.86 |
| 2016 | 540.09 | +0.94 |
| 2015 | 539.15 | +51.26 |
| 2012 | 515.95 | +15.23 |

- All 10 peak years are **post-2012** — Indonesia's emissions problem is recent and accelerating.
- 2022 saw the single largest year-on-year jump: **+117.45 Mt**, driven by the nickel smelter boom and post-COVID industrial rebound.
- 2020's -45.57 Mt drop (COVID) was fully reversed within two years.

---

**2. How has Indonesia's CO₂ grown decade by decade?**

```sql
SELECT
  (year / 10) * 10              AS decade,
  COUNT(*)                      AS years,
  ROUND(AVG(co2), 2)            AS avg_co2_mt,
  ROUND(MAX(co2), 2)            AS peak_co2_mt
FROM `co2_data`
WHERE Country = 'Indonesia'
  AND co2 IS NOT NULL
GROUP BY decade
ORDER BY decade;
```

#### Steps:

- `(year / 10) * 10` is integer division, it floors the year to the decade (e.g. 2019 → 2010, 1987 → 1980). This creates a decade grouping without needing a date function.
- `AVG(co2)` gives the typical emissions level for the decade; `MAX(co2)` shows the peak within it.
- `GROUP BY decade` aggregates all years in the same 10-year bucket.

#### Answer:

| decade | years | avg_co2_mt | peak_co2_mt |
|---|---|---|---|
| 1960 | 10 | 24.90 | 33.36 |
| 1970 | 10 | 60.56 | 95.14 |
| 1980 | 10 | 115.04 | 132.72 |
| 1990 | 10 | 225.58 | 292.00 |
| 2000 | 10 | 343.60 | 398.94 |
| 2010 | 10 | 532.35 | 653.79 |
| 2020 | 4 | 674.54 | 737.07 |

- Indonesia's average CO₂ has **roughly doubled each decade** since the 1960s.
- The 2020s decade average (674 Mt) already exceeds the entire 2010s average (532 Mt), based on just 4 years of data.

---

**3. What is the year-on-year CO₂ growth rate from 2010 to 2023?**

```sql
SELECT 
  year,
  ROUND(co2, 2)                 AS co2_mt,
  ROUND(co2_growth_prct, 1)     AS yoy_growth_pct
FROM `co2_data`
WHERE Country = 'Indonesia'
  AND year BETWEEN 2010 AND 2023
ORDER BY year;
```

#### Steps:

- `co2_growth_prct` is a pre-calculated field in the OWID dataset — the percentage change in CO₂ from the previous year.
- `BETWEEN 2010 AND 2023` is inclusive on both ends in BigQuery.
- `ROUND(co2_growth_prct, 1)` rounds to 1 decimal place for readability.

#### Answer:

| year | co2_mt | yoy_growth_pct |
|---|---|---|
| 2010 | 445.81 | 11.7% |
| 2011 | 500.73 | 12.3% |
| 2012 | 515.95 | 3.0% |
| 2013 | 489.06 | -5.2% |
| 2014 | 487.89 | -0.2% |
| 2015 | 539.15 | 10.5% |
| 2016 | 540.09 | 0.2% |
| 2017 | 556.94 | 3.1% |
| 2018 | 594.10 | 6.7% |
| 2019 | 653.79 | 10.0% |
| 2020 | 608.22 | -7.0% |
| 2021 | 619.62 | 1.9% |
| 2022 | 737.07 | 19.0% |
| 2023 | 733.22 | -0.5% |

- 2022 shows a **+19.0% single-year jump** — the largest on record.
- The brief dip in 2013–2014 was followed by the 2015 coal push, which reset the trajectory upward.
- COVID (2020: -7.0%) fully rebounded within 2 years — matching the pattern of every previous crisis-driven dip.

---

**4. Which fuel type is responsible for most of Indonesia's emissions in 2022?**

```sql
SELECT 'Coal'    AS fuel, ROUND(coal_co2, 2)    AS co2_mt FROM `co2_data` WHERE Country = 'Indonesia' AND year = 2022
UNION ALL SELECT 'Oil',    ROUND(oil_co2, 2)     FROM `co2_data` WHERE Country = 'Indonesia' AND year = 2022
UNION ALL SELECT 'Gas',    ROUND(gas_co2, 2)     FROM `co2_data` WHERE Country = 'Indonesia' AND year = 2022
UNION ALL SELECT 'Cement', ROUND(cement_co2, 2)  FROM `co2_data` WHERE Country = 'Indonesia' AND year = 2022
UNION ALL SELECT 'Flaring',ROUND(flaring_co2, 2) FROM `co2_data` WHERE Country = 'Indonesia' AND year = 2022
ORDER BY co2_mt DESC;
```

#### Steps:

- Each fuel type is stored in a separate column in the OWID dataset `UNION ALL` stacks them into rows for clean comparison.
- All five queries reference the same filter (`Country = 'Indonesia' AND year = 2022`) to ensure they're pulling from the same base record.
- `ORDER BY co2_mt DESC` ranks by contribution size.

#### Answer:

| fuel | co2_mt |
|---|---|
| Coal | 404.57 |
| Oil | 216.56 |
| Gas | 85.93 |
| Cement | 26.84 |
| Flaring | 3.16 |

- **Coal accounts for 54.9% of all Indonesia's emissions** in 2022 up from 34.2% in 2010.
- The coal figure alone (404 Mt) is larger than the total CO₂ of most Southeast Asian countries.
- Oil has remained relatively flat at ~200–220 Mt since 2010. The growth is almost entirely coal.

---

**5. How does Indonesia compare to its Southeast Asian neighbours in 2022?**

```sql
SELECT 
  Country,
  ROUND(co2, 2)                 AS total_co2_mt,
  ROUND(co2_per_capita, 2)      AS co2_per_person_t,
  ROUND(share_global_co2, 2)    AS pct_of_global
FROM `co2_data`
WHERE Country IN ('Indonesia', 'Vietnam', 'Thailand', 'Philippines', 'Malaysia', 'Singapore', 'Myanmar')
  AND year = 2022
ORDER BY total_co2_mt DESC;
```

#### Steps:

- Use `IN()` with a defined list of Southeast Asian countries, note these must match the exact country name strings in the dataset.
- `co2_per_capita` is in tonnes per person; `share_global_co2` is a percentage of global total.
- Comparing all three columns side by side reveals the difference between absolute and per-capita narratives.

#### Answer:

| Country | total_co2_mt | co2_per_person_t | pct_of_global |
|---|---|---|---|
| Indonesia | 737.07 | 2.64 | 1.98% |
| Vietnam | 297.63 | 2.99 | 0.80% |
| Malaysia | 285.45 | 8.23 | 0.77% |
| Thailand | 272.57 | 3.80 | 0.73% |
| Philippines | 144.90 | 1.27 | 0.39% |
| Singapore | 47.87 | 8.47 | 0.13% |
| Myanmar | 29.53 | 0.55 | 0.08% |

- Indonesia emits **2.5× more than Vietnam**, the next largest in absolute terms.
- But per capita, Malaysia (8.23t) and Singapore (8.47t) are both **3× higher** than Indonesia (2.64t).
- Indonesia's challenge is **scale, not lifestyle** — population of 280M amplifies even modest per-person emissions into a regional problem.

---

**6. In which years did Indonesia's CO₂ emissions actually fall?**

```sql
SELECT 
  year,
  ROUND(co2, 2)                 AS co2_mt,
  ROUND(co2_growth_prct, 1)     AS growth_pct
FROM `co2_data`
WHERE Country = 'Indonesia'
  AND co2_growth_prct < 0
  AND year >= 1960
ORDER BY co2_growth_prct ASC;
```

#### Steps:

- `WHERE co2_growth_prct < 0` filters to only years where emissions fell year-on-year.
- `AND year >= 1960` excludes early historical records where the data is sparse and percentage changes are less meaningful.
- `ORDER BY co2_growth_prct ASC` puts the largest drops first (most negative at top).

#### Answer:

| year | co2_mt | growth_pct |
|---|---|---|
| 1998 | 244.52 | -12.7% |
| 1962 | 22.98 | -11.6% |
| 1960 | 21.39 | -10.4% |
| 2020 | 608.22 | -7.0% |
| 2008 | 365.72 | -5.7% |
| 2013 | 489.06 | -5.2% |
| 2000 | 281.33 | -3.7% |

- **Emissions only fell during major economic crises** — 1998 Asian Financial Crisis (-12.7%), 2008 Global Financial Crisis (-5.7%), 2020 COVID (-7.0%).
- Every dip was followed by a sharp rebound — the underlying trend never changed direction.
- This confirms that crisis-driven reductions are temporary. Only structural policy change can bend the curve.

---

**7. Did Indonesia's 2015 coal expansion change its emissions trajectory?**

```sql
SELECT
  CASE 
    WHEN year < 2014 THEN 'Pre-2014 (2000–2013)'
    ELSE 'Post-2014 (2014–2022)'
  END                                          AS period,
  COUNT(*)                                     AS years,
  ROUND(AVG(co2), 2)                           AS avg_annual_co2_mt,
  ROUND(AVG(co2_growth_prct), 1)               AS avg_growth_pct,
  ROUND(MAX(co2) - MIN(co2), 2)                AS total_increase_mt
FROM `co2_data`
WHERE Country = 'Indonesia'
  AND year BETWEEN 2000 AND 2022
  AND co2 IS NOT NULL
GROUP BY period
ORDER BY period;
```

#### Steps:

- Use `CASE WHEN year < 2014` to split the analysis into two periods: before and after the "35,000 Megawatt" coal power expansion launched in 2015.
- `AVG(co2_growth_prct)` gives the typical annual growth rate for each period.
- `MAX(co2) - MIN(co2)` gives the total absolute increase in emissions across each period.

#### Answer:

| period | years | avg_annual_co2_mt | avg_growth_pct | total_increase_mt |
|---|---|---|---|---|
| Post-2014 (2014–2022) | 9 | 592.99 | 4.9% | 249.18 |
| Pre-2014 (2000–2013) | 14 | 384.82 | 4.0% | 234.62 |

- Average annual growth rate increased from **4.0% to 4.9%** after the coal expansion.
- Before 2014, energy efficiency improvements were saving ~120 Mt per year — the economy was decoupling from emissions.
- After 2014, efficiency gains stalled and the **grid became dirtier** (+45 Mt from increased carbon intensity). A 0.9 percentage point increase in growth rate compounds significantly over a decade.

---

**8. How has Indonesia's share of global CO₂ changed over time?**

```sql
SELECT 
  year,
  ROUND(co2, 2)                 AS co2_mt,
  ROUND(share_global_co2, 2)    AS pct_of_global_co2
FROM `co2_data`
WHERE Country = 'Indonesia'
  AND year IN (1990, 1995, 2000, 2005, 2010, 2015, 2020, 2022)
ORDER BY year;
```

#### Steps:

- Use `IN()` with a list of benchmark years (every 5 years from 1990) to create a clean long-term trend view without querying every year.
- `share_global_co2` is the percentage of total global emissions attributable to Indonesia in that year.

#### Answer:

| year | co2_mt | pct_of_global_co2 |
|---|---|---|
| 1990 | 155.08 | 0.68% |
| 1995 | 222.41 | 0.94% |
| 2000 | 281.33 | 1.10% |
| 2005 | 347.62 | 1.17% |
| 2010 | 445.81 | 1.34% |
| 2015 | 539.15 | 1.52% |
| 2020 | 608.22 | 1.73% |
| 2022 | 737.07 | 1.98% |

- Indonesia's global share has **nearly tripled since 1990** from 0.68% to 1.98%.
- It is now approaching 2% of all global CO₂ — a significant position for a country that is not the world's largest economy.
- The acceleration post-2015 is visible: from 1.52% to 1.98% in just 7 years.

---

**9. How has coal's share of Indonesia's total emissions changed since 2010?**

```sql
SELECT 
  year,
  ROUND(coal_co2, 2)                          AS coal_mt,
  ROUND(oil_co2, 2)                           AS oil_mt,
  ROUND(co2, 2)                               AS total_mt,
  ROUND(100.0 * coal_co2 / co2, 1)            AS coal_share_pct
FROM `co2_data`
WHERE Country = 'Indonesia'
  AND year BETWEEN 2010 AND 2022
  AND co2 IS NOT NULL
ORDER BY year;
```

#### Steps:

- `100.0 * coal_co2 / co2` calculates coal's percentage contribution to total emissions. Using `100.0` (not `100`) forces floating-point division in BigQuery.
- Show both absolute values (`coal_mt`, `oil_mt`) and the percentage to illustrate that oil stayed flat while coal surged.

#### Answer:

| year | coal_mt | oil_mt | total_mt | coal_share_pct |
|---|---|---|---|---|
| 2010 | 152.67 | 195.67 | 445.81 | 34.2% |
| 2012 | 187.17 | 218.78 | 515.95 | 36.3% |
| 2015 | 197.81 | 217.06 | 539.15 | 36.7% |
| 2018 | 262.22 | 212.95 | 594.10 | 44.1% |
| 2019 | 315.40 | 216.54 | 653.79 | 48.2% |
| 2020 | 300.52 | 196.67 | 608.22 | 49.4% |
| 2022 | 404.57 | 216.56 | 737.07 | 54.9% |

- Coal has gone from **34% to 55% of total emissions** in 12 years — the grid is getting dirtier, not cleaner.
- Oil has remained almost flat at ~200–220 Mt throughout the period. All the growth is coal.
- Reducing coal dependency is the single highest-leverage intervention available to Indonesia.

---

**10. How does Indonesia's per-capita CO₂ compare to its neighbours over 2000–2022?**

```sql
SELECT 
  Country,
  ROUND(AVG(co2_per_capita), 2)   AS avg_co2_per_capita_t,
  ROUND(MIN(co2_per_capita), 2)   AS min,
  ROUND(MAX(co2_per_capita), 2)   AS max
FROM `co2_data`
WHERE Country IN ('Indonesia', 'Vietnam', 'Thailand', 'Philippines', 'Malaysia')
  AND year BETWEEN 2000 AND 2022
  AND co2_per_capita IS NOT NULL
GROUP BY Country
ORDER BY avg_co2_per_capita_t DESC;
```

#### Steps:

- `AVG(co2_per_capita)` across 23 years (2000–2022) gives a stable long-run average, smoothing out single-year fluctuations.
- `MIN()` and `MAX()` show how much each country's per-capita emissions have moved over the period.
- `WHERE co2_per_capita IS NOT NULL` excludes years where the field is empty.

#### Answer:

| Country | avg_co2_per_capita_t | min | max |
|---|---|---|---|
| Malaysia | 7.07 | 5.35 | 8.37 |
| Thailand | 3.50 | 2.66 | 4.04 |
| Vietnam | 1.85 | 0.70 | 3.70 |
| Indonesia | 1.85 | 1.30 | 2.64 |
| Philippines | 0.98 | 0.74 | 1.29 |

- Indonesia and Vietnam have **identical average per-capita emissions** (1.85 t/person) over 2000–2022.
- Malaysia emits **4× more per person** than Indonesia yet Indonesia generates 2.5× Malaysia's absolute total due to its population of 280M.
- Indonesia's emissions problem is one of **population scale**, not individual consumption level. Clean energy infrastructure benefits all 280M people simultaneously.
