# Session 4: Real Analysis

**Dataset:** theLook eCommerce on BigQuery (`bigquery-public-data.thelook_ecommerce`)
**Length:** 2 hours

## By the end of this session you can

- Work with dates and timestamps to build trends
- Create custom categories with `CASE WHEN`
- Calculate rates safely with `COUNTIF` and `SAFE_DIVIDE`
- Break a long query into readable steps with CTEs (`WITH`)
- Answer an open business question end to end

## Agenda

| Time | Activity |
| --- | --- |
| 0:00 | Recap and warm-up |
| 0:05 | Live demo: dates, CASE, CTEs |
| 0:30 | Exercises 4.1 to 4.6 |
| 1:00 | Capstone in pairs |
| 1:40 | Capstone presentations |

## 1. Dates and timestamps

Most `created_at` columns are **timestamps** (date + time). Useful functions:

| Function | Example | Returns |
| --- | --- | --- |
| `DATE(ts)` | `DATE(created_at)` | just the date |
| `EXTRACT(part FROM ts)` | `EXTRACT(YEAR FROM created_at)` | 2025 |
| `DATE_TRUNC(date, part)` | `DATE_TRUNC(DATE(created_at), MONTH)` | first day of that month |
| `TIMESTAMP_DIFF(a, b, unit)` | `TIMESTAMP_DIFF(delivered_at, created_at, DAY)` | days between |
| `CURRENT_DATE()` | | today |
| `DATE_SUB(date, INTERVAL n unit)` | `DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)` | 30 days ago |

```sql
SELECT DATE_TRUNC(DATE(created_at), MONTH) AS month,
       COUNT(*) AS orders
FROM `bigquery-public-data.thelook_ecommerce.orders`
GROUP BY month
ORDER BY month;
```

## 2. CASE WHEN

Creates your own categories. Conditions are checked top to bottom; the first match wins.

```sql
SELECT name, retail_price,
  CASE
    WHEN retail_price < 25  THEN 'Budget'
    WHEN retail_price < 100 THEN 'Mid'
    ELSE 'Premium'
  END AS price_tier
FROM `bigquery-public-data.thelook_ecommerce.products`;
```

## 3. Rates: COUNTIF and SAFE_DIVIDE

- `COUNTIF(condition)` counts rows where the condition is true.
- `SAFE_DIVIDE(a, b)` returns `NULL` instead of an error when `b` is 0.

```sql
SELECT SAFE_DIVIDE(COUNTIF(status = 'Cancelled'), COUNT(*)) AS cancel_rate
FROM `bigquery-public-data.thelook_ecommerce.orders`;
```

## 4. CTEs (WITH)

A CTE is a named, temporary result you can query like a table. Use them to build an analysis in clear steps.

```sql
WITH order_counts AS (
  SELECT user_id, COUNT(*) AS num_orders
  FROM `bigquery-public-data.thelook_ecommerce.orders`
  GROUP BY user_id
)
SELECT num_orders, COUNT(*) AS users
FROM order_counts
GROUP BY num_orders
ORDER BY num_orders;
```

Several CTEs are separated by commas: `WITH a AS (...), b AS (...) SELECT ...`

## 5. Exercises

**4.1 Orders by year.** Count orders per year using `EXTRACT(YEAR FROM created_at)`.

**4.2 Monthly trend.** Completed-item revenue per month for the last 12 full months.

**4.3 Age bands.** Bucket users into `Under 25`, `25-44`, `45-64` and `65+` with `CASE`, then count users per band.

**4.4 Delivery time.** For delivered orders, what is the average number of days between `created_at` and `delivered_at`, by country?

**4.5 Return rate.** Return rate by product category: returned items divided by all items. Use `COUNTIF(status = 'Returned')` and `SAFE_DIVIDE`.

**4.6 CTE.** Using a CTE, find each user's total completed spend, then count how many users fall into spend tiers: `< $100`, `$100-499`, `$500+`.

## 6. Capstone (pairs, 40 minutes)

The head of marketing asks:

> **"Which traffic source brings us our most valuable customers, and should we shift budget?"**

Your answer should cover:

- Users by `traffic_source` (from `users`), and the share who ever bought something
- Revenue and average revenue per buyer by source (completed items only)
- Return rate by source
- One caveat you noticed in the data, and one follow-up question

Structure your query with CTEs. Present a 3-minute summary: your answer, the numbers behind it, and how confident you are.

**Finished early? Pick one:**

- **Website funnel:** using `events`, how many sessions reach each `event_type` (`home`, `department`, `product`, `cart`, `purchase`)? Where is the biggest drop-off?
- **Inventory:** using `inventory_items`, which categories have the most unsold stock (`sold_at IS NULL`), and how long has it sat?

## 7. Where to go next

- Window functions (`ROW_NUMBER`, `RANK`, running totals)
- Subqueries
- Other BigQuery public datasets to practice on
