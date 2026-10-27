# Session 2 Answers: Summarizing Data

theLook data is regenerated regularly, so totals may differ from run to run. Other correct queries exist; these are one good answer each.

## 2.1 Count

```sql
SELECT COUNT(*) AS total_users,
       COUNT(DISTINCT country) AS countries
FROM `bigquery-public-data.thelook_ecommerce.users`;
```

## 2.2 Group by one column

```sql
SELECT traffic_source, COUNT(*) AS users
FROM `bigquery-public-data.thelook_ecommerce.users`
GROUP BY traffic_source
ORDER BY users DESC;
```

## 2.3 Averages

```sql
SELECT gender, ROUND(AVG(age), 1) AS avg_age
FROM `bigquery-public-data.thelook_ecommerce.users`
GROUP BY gender;
```

## 2.4 Price stats

```sql
SELECT category,
       COUNT(*) AS products,
       MIN(retail_price) AS min_price,
       MAX(retail_price) AS max_price,
       ROUND(AVG(retail_price), 2) AS avg_price
FROM `bigquery-public-data.thelook_ecommerce.products`
GROUP BY category
ORDER BY products DESC;
```

## 2.5 Group by two columns

```sql
SELECT status, gender, COUNT(*) AS orders
FROM `bigquery-public-data.thelook_ecommerce.orders`
GROUP BY status, gender
ORDER BY status, gender;
```

Both non-aggregated columns must appear in `GROUP BY`.

## 2.6 HAVING

```sql
SELECT brand, COUNT(*) AS products
FROM `bigquery-public-data.thelook_ecommerce.products`
GROUP BY brand
HAVING COUNT(*) > 50
ORDER BY products DESC;
```

Common mistake: `WHERE COUNT(*) > 50` causes an error, because `WHERE` runs before the groups exist.

## 2.7 Revenue

```sql
SELECT status, ROUND(SUM(sale_price), 2) AS revenue
FROM `bigquery-public-data.thelook_ecommerce.order_items`
GROUP BY status
ORDER BY revenue DESC;
```

Discussion:
- `Complete` is the safest definition of real revenue.
- `Shipped` and `Processing` may still be cancelled or returned, so many teams exclude them or report them separately.
- `Cancelled` and `Returned` are not revenue.
- The key lesson: decide which rows count **before** you sum.

## 2.8 NULL trap

```sql
SELECT COUNT(*) AS all_orders,
       COUNT(returned_at) AS returned_orders
FROM `bigquery-public-data.thelook_ecommerce.orders`;
```

`COUNT(col)` skips NULLs. `returned_at` is only filled in for returned orders, so `COUNT(returned_at)` equals the number of returned orders. The same rule means `AVG` ignores NULLs too.

## Stretch

```sql
SELECT country,
       COUNT(*) AS users,
       ROUND(AVG(age), 1) AS avg_age
FROM `bigquery-public-data.thelook_ecommerce.users`
GROUP BY country
HAVING AVG(age) > 40
   AND COUNT(*) >= 1000
ORDER BY avg_age DESC;
```

If this returns nothing, lower the thresholds and discuss why: theLook's ages are generated fairly evenly, so country averages sit close together.
