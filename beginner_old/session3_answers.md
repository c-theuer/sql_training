# Session 3 Answers: Combining Tables with JOINs

theLook data is regenerated regularly, so totals may differ from run to run. Other correct queries exist; these are one good answer each.

## 3.1 First join

```sql
SELECT oi.order_id, oi.sale_price, p.name, p.brand
FROM `bigquery-public-data.thelook_ecommerce.order_items` AS oi
JOIN `bigquery-public-data.thelook_ecommerce.products` AS p
  ON oi.product_id = p.id
LIMIT 20;
```

## 3.2 Join + filter

```sql
SELECT oi.order_id, p.name, u.first_name, u.country
FROM `bigquery-public-data.thelook_ecommerce.order_items` AS oi
JOIN `bigquery-public-data.thelook_ecommerce.products` AS p
  ON oi.product_id = p.id
JOIN `bigquery-public-data.thelook_ecommerce.users` AS u
  ON oi.user_id = u.id
WHERE p.category = 'Swim';
```

## 3.3 Join + aggregate

```sql
SELECT p.category, ROUND(SUM(oi.sale_price), 2) AS revenue
FROM `bigquery-public-data.thelook_ecommerce.order_items` AS oi
JOIN `bigquery-public-data.thelook_ecommerce.products` AS p
  ON oi.product_id = p.id
WHERE oi.status = 'Complete'
GROUP BY p.category
ORDER BY revenue DESC;
```

Note: `status` exists in both `orders` and `order_items`. Here we use the item-level status.

## 3.4 Three tables

```sql
SELECT u.country, p.department, ROUND(SUM(oi.sale_price), 2) AS revenue
FROM `bigquery-public-data.thelook_ecommerce.order_items` AS oi
JOIN `bigquery-public-data.thelook_ecommerce.products` AS p
  ON oi.product_id = p.id
JOIN `bigquery-public-data.thelook_ecommerce.users` AS u
  ON oi.user_id = u.id
WHERE oi.status = 'Complete'
GROUP BY u.country, p.department
ORDER BY revenue DESC;
```

## 3.5 LEFT JOIN

```sql
SELECT COUNT(*) AS users_without_orders
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
LEFT JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o
  ON u.id = o.user_id
WHERE o.order_id IS NULL;
```

Why it works: the `LEFT JOIN` keeps every user. Users with no orders get `NULL` in every `orders` column, and the `WHERE` keeps only those.

Common mistake: using `JOIN` (inner) here always returns 0, because unmatched users are dropped before the `WHERE` runs.

## 3.6 Fix the bug

**The problem:** `order_items` has one row per item, so `COUNT(*)` counts items, not orders. An order with 3 items is counted 3 times.

**Fix 1:** count distinct orders.

```sql
SELECT u.country, COUNT(DISTINCT oi.order_id) AS num_orders
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  ON u.id = oi.user_id
GROUP BY u.country
ORDER BY num_orders DESC;
```

**Fix 2:** join the table at the right grain.

```sql
SELECT u.country, COUNT(*) AS num_orders
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o
  ON u.id = o.user_id
GROUP BY u.country
ORDER BY num_orders DESC;
```

Both give the same totals. Fix 2 is clearer and cheaper.

## 3.7 Distribution centers

```sql
SELECT dc.name, COUNT(*) AS products
FROM `bigquery-public-data.thelook_ecommerce.products` AS p
JOIN `bigquery-public-data.thelook_ecommerce.distribution_centers` AS dc
  ON p.distribution_center_id = dc.id
GROUP BY dc.name
ORDER BY products DESC;
```

## Stretch

```sql
SELECT p.brand,
       ROUND(SUM(oi.sale_price), 2) AS revenue,
       ROUND(SUM(oi.sale_price - p.cost), 2) AS profit
FROM `bigquery-public-data.thelook_ecommerce.order_items` AS oi
JOIN `bigquery-public-data.thelook_ecommerce.products` AS p
  ON oi.product_id = p.id
WHERE oi.status = 'Complete'
GROUP BY p.brand
ORDER BY profit DESC
LIMIT 10;
```

To compare with the top 10 by revenue, change `ORDER BY profit DESC` to `ORDER BY revenue DESC`. Discussion: brands with higher margins can rank higher on profit than on revenue.
