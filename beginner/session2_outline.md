# Session 2: Summarizing Data

**Dataset:** theLook eCommerce on BigQuery (`bigquery-public-data.thelook_ecommerce`)
**Length:** 2 hours

## By the end of this session you can

- Count, sum and average values across a table
- Summarize by group with `GROUP BY`
- Filter groups with `HAVING`
- Explain how `NULL` values affect counts and averages

## Agenda

| Time | Activity |
| --- | --- |
| 0:00 | Recap and warm-up |
| 0:10 | Live demo: aggregates and GROUP BY |
| 0:40 | Exercises in pairs |
| 1:40 | Share-outs and debrief |

## 1. Warm-up

Without running it, predict what this returns. Then run it.

```sql
SELECT DISTINCT category
FROM `bigquery-public-data.thelook_ecommerce.products`
ORDER BY category;
```

## 2. Aggregate functions

Aggregates collapse many rows into one value.

| Function | What it returns |
| --- | --- |
| `COUNT(*)` | number of rows |
| `COUNT(col)` | number of rows where `col` is not NULL |
| `COUNT(DISTINCT col)` | number of unique values |
| `SUM(col)` | total |
| `AVG(col)` | average (ignores NULLs) |
| `MIN(col)`, `MAX(col)` | smallest, largest |
| `ROUND(x, 2)` | round to 2 decimal places |

```sql
SELECT COUNT(*) AS total_users,
       ROUND(AVG(age), 1) AS avg_age
FROM `bigquery-public-data.thelook_ecommerce.users`;
```

## 3. GROUP BY

`GROUP BY` runs the aggregate once per group.

```sql
SELECT country, COUNT(*) AS users
FROM `bigquery-public-data.thelook_ecommerce.users`
GROUP BY country
ORDER BY users DESC;
```

**The golden rule:** every column in `SELECT` must either be in `GROUP BY` or wrapped in an aggregate.

## 4. WHERE vs HAVING

- `WHERE` filters **rows before** grouping.
- `HAVING` filters **groups after** grouping.

```sql
SELECT country, COUNT(*) AS users
FROM `bigquery-public-data.thelook_ecommerce.users`
WHERE gender = 'F'            -- rows: only women
GROUP BY country
HAVING COUNT(*) > 1000        -- groups: only big countries
ORDER BY users DESC;
```

Order of clauses: `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT`.

## 5. Tips

- Name your aggregates with `AS` so results are readable.
- You can `ORDER BY` an aggregate's alias.
- An aggregate can't go in `WHERE`. If you see that error, you need `HAVING`.
- Before summing money, decide which rows should count (for example, should cancelled items count as revenue?).

## 6. Exercises

**2.1 Count.** How many users are in the `users` table? How many distinct countries do they come from?

**2.2 Group by one column.** How many users came from each `traffic_source`? Sort from most to fewest.

**2.3 Averages.** What is the average `age` of users by `gender`? Round to one decimal place.

**2.4 Price stats.** For each product `category`, show the number of products, the cheapest, the most expensive and the average `retail_price`.

**2.5 Group by two columns.** Count orders by `status` and `gender`.

**2.6 HAVING.** Which brands have more than 50 products? Sort by product count, highest first.

**2.7 Revenue.** Using `order_items`, what is total revenue (`SUM(sale_price)`) by item `status`? Which statuses should count as real revenue, and why?

**2.8 NULL trap.** Compare `COUNT(*)` with `COUNT(returned_at)` in `orders`. What does the difference tell you?

**Stretch.** Which countries have an average user age above 40 and at least 1,000 users?

## 7. Debrief questions

- When would you use `COUNT(DISTINCT ...)` instead of `COUNT(*)`?
- Explain the difference between `WHERE` and `HAVING` in one sentence.
- What would happen to total revenue if you forgot to filter on status?
