# Session 4 Answers: Real Analysis

theLook data is regenerated regularly, so totals may differ from run to run. Other correct queries exist; these are one good answer each.

## 4.1 Orders by year

```sql
SELECT EXTRACT(YEAR FROM created_at) AS year, COUNT(*) AS orders
FROM `bigquery-public-data.thelook_ecommerce.orders`
GROUP BY year
ORDER BY year;
```

## 4.2 Monthly trend

```sql
SELECT DATE_TRUNC(DATE(created_at), MONTH) AS month,
       ROUND(SUM(sale_price), 2) AS revenue
FROM `bigquery-public-data.thelook_ecommerce.order_items`
WHERE status = 'Complete'
  AND DATE(created_at) >= DATE_SUB(DATE_TRUNC(CURRENT_DATE(), MONTH), INTERVAL 12 MONTH)
  AND DATE(created_at) <  DATE_TRUNC(CURRENT_DATE(), MONTH)
GROUP BY month
ORDER BY month;
```

Why "full months": the second condition excludes the current month, which is only partly over and would look like a drop.

Tip: click **Chart** in the results pane to see the trend as a line.

## 4.3 Age bands

```sql
SELECT
  CASE
    WHEN age < 25 THEN 'Under 25'
    WHEN age < 45 THEN '25-44'
    WHEN age < 65 THEN '45-64'
    ELSE '65+'
  END AS age_band,
  COUNT(*) AS users
FROM `bigquery-public-data.thelook_ecommerce.users`
GROUP BY age_band
ORDER BY MIN(age);
```

`ORDER BY MIN(age)` sorts the bands youngest first instead of alphabetically.

## 4.4 Delivery time

```sql
SELECT u.country,
       ROUND(AVG(TIMESTAMP_DIFF(o.delivered_at, o.created_at, DAY)), 1) AS avg_days
FROM `bigquery-public-data.thelook_ecommerce.orders` AS o
JOIN `bigquery-public-data.thelook_ecommerce.users` AS u
  ON o.user_id = u.id
WHERE o.delivered_at IS NOT NULL
GROUP BY u.country
ORDER BY avg_days DESC;
```

## 4.5 Return rate

```sql
SELECT p.category,
       COUNTIF(oi.status = 'Returned') AS returned_items,
       COUNT(*) AS total_items,
       ROUND(SAFE_DIVIDE(COUNTIF(oi.status = 'Returned'), COUNT(*)), 3) AS return_rate
FROM `bigquery-public-data.thelook_ecommerce.order_items` AS oi
JOIN `bigquery-public-data.thelook_ecommerce.products` AS p
  ON oi.product_id = p.id
GROUP BY p.category
ORDER BY return_rate DESC;
```

Discussion: should cancelled items be in the denominator? A cancelled item was never delivered, so it could never be returned. Excluding them (`WHERE oi.status <> 'Cancelled'`) is a defensible choice. Either way, say which you chose.

## 4.6 CTE

```sql
WITH user_spend AS (
  SELECT user_id, SUM(sale_price) AS spend
  FROM `bigquery-public-data.thelook_ecommerce.order_items`
  WHERE status = 'Complete'
  GROUP BY user_id
)
SELECT
  CASE
    WHEN spend < 100 THEN '< $100'
    WHEN spend < 500 THEN '$100-499'
    ELSE '$500+'
  END AS tier,
  COUNT(*) AS users
FROM user_spend
GROUP BY tier
ORDER BY MIN(spend);
```

Note: users with no completed purchases are not in `user_spend` at all, so they don't appear in any tier.

## Capstone reference solution

```sql
WITH users_by_source AS (
  SELECT traffic_source, COUNT(*) AS users
  FROM `bigquery-public-data.thelook_ecommerce.users`
  GROUP BY traffic_source
),
buyer_value AS (
  SELECT user_id,
         SUM(IF(status = 'Complete', sale_price, 0)) AS revenue,
         COUNTIF(status = 'Returned') AS returned_items,
         COUNT(*) AS items
  FROM `bigquery-public-data.thelook_ecommerce.order_items`
  GROUP BY user_id
)
SELECT u.traffic_source,
       s.users,
       COUNT(b.user_id) AS buyers,
       ROUND(SAFE_DIVIDE(COUNT(b.user_id), s.users), 3) AS buyer_rate,
       ROUND(SUM(b.revenue), 2) AS revenue,
       ROUND(SAFE_DIVIDE(SUM(b.revenue), COUNT(b.user_id)), 2) AS revenue_per_buyer,
       ROUND(SAFE_DIVIDE(SUM(b.returned_items), SUM(b.items)), 3) AS return_rate
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
JOIN users_by_source AS s
  ON u.traffic_source = s.traffic_source
LEFT JOIN buyer_value AS b
  ON u.id = b.user_id
GROUP BY u.traffic_source, s.users
ORDER BY revenue_per_buyer DESC;
```

How it works:
- `users_by_source` counts all users per source.
- `buyer_value` builds one row per buyer, so later sums don't double-count.
- The `LEFT JOIN` keeps users who never bought; `COUNT(b.user_id)` counts only the ones who did.

What to look for in student answers:
- Correct grain (no double-counting from joining items to users)
- Revenue limited to completed items
- An honest caveat. theLook is synthetic, so the sources often perform similarly. "The data doesn't show a clear winner" is a strong answer when it's true.
- A sensible follow-up, such as marketing cost per source (not in this dataset) to judge return on spend

## Alternate prompt: website funnel

```sql
SELECT event_type,
       COUNT(DISTINCT session_id) AS sessions
FROM `bigquery-public-data.thelook_ecommerce.events`
WHERE event_type IN ('home', 'department', 'product', 'cart', 'purchase')
GROUP BY event_type
ORDER BY sessions DESC;
```

The biggest drop-off is where the session count falls most between consecutive steps.

## Alternate prompt: inventory

```sql
SELECT product_category,
       COUNT(*) AS unsold_units,
       ROUND(AVG(TIMESTAMP_DIFF(CURRENT_TIMESTAMP(), created_at, DAY)), 0) AS avg_days_in_stock
FROM `bigquery-public-data.thelook_ecommerce.inventory_items`
WHERE sold_at IS NULL
GROUP BY product_category
ORDER BY unsold_units DESC;
```
