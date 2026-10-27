## Session 1 Exercise Answers

Each answer says which join fits and why. theLook data is regenerated regularly, so your numbers may differ from the sample results here (run on October 27, 2026).

| # | Exercise | Join |
| --- | --- | --- |
| 1 | Top categories | Inner |
| 2 | Dead stock | Left |
| 3 | Coming back | Self |
| 4 | Busiest warehouse | Inner (two of them) |
| 5 | First-week conversion | Left, with the filter in ON |
| 6 | Slow weeks | Self |
| 7 | Quiet jeans brands | Left, with the filter in ON |
| 8 | Double orders | Self |

### 1. Top categories: inner join

We only want sales that have a product, and products that have sales. Nothing without a match belongs in a revenue ranking.

```sql
SELECT p.category, ROUND(SUM(i.sale_price), 2) AS revenue
FROM `bigquery-public-data.thelook_ecommerce.order_items` AS i
INNER JOIN `bigquery-public-data.thelook_ecommerce.products` AS p
  ON i.product_id = p.id
WHERE i.status = 'Complete'
  AND i.created_at >= '2026-09-01'
  AND i.created_at < '2026-10-01'
GROUP BY p.category
ORDER BY revenue DESC
LIMIT 5;
```

Sample result: Jeans, Outerwear & Coats, Sweaters, Suits & Sport Coats, Sleep & Lounge.

### 2. Dead stock: left join

The question is about products with **no** match in `order_items`. An inner join would throw away exactly the rows we're looking for. Put `products` on the left and keep the rows where the right side is NULL.

```sql
SELECT p.department, COUNT(*) AS never_ordered_products
FROM `bigquery-public-data.thelook_ecommerce.products` AS p
LEFT JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS i
  ON p.id = i.product_id
WHERE i.id IS NULL
GROUP BY p.department
ORDER BY never_ordered_products DESC;
```

Sample result: Women 48, Men 8.

`COUNT(*)` is safe here because each unmatched product produces exactly one row.

### 3. Coming back: self join

We're comparing each order to *other orders in the same table* from the same customer. Give `orders` two roles: the original order and the next one.

```sql
SELECT COUNT(DISTINCT first_order.user_id) AS customers_who_came_back
FROM `bigquery-public-data.thelook_ecommerce.orders` AS first_order
INNER JOIN `bigquery-public-data.thelook_ecommerce.orders` AS next_order
  ON next_order.user_id = first_order.user_id
  AND next_order.created_at > first_order.created_at
  AND next_order.created_at <= TIMESTAMP_ADD(first_order.created_at, INTERVAL 30 DAY)
WHERE first_order.created_at >= '2026-01-01'
  AND first_order.created_at < '2026-07-01';
```

Sample result: 982 customers (out of 18,149 who ordered in that period).

- `next_order.created_at > first_order.created_at` stops an order from matching itself.
- `COUNT(DISTINCT ...)` matters: a customer with several qualifying pairs would otherwise be counted several times.
- Stopping the first orders at June 30 means every one of them has a full 30 days of follow-up data.

### 4. Busiest warehouse: inner join, twice

`order_items` doesn't know the distribution center, so we hop through `products`. Every sale has a product and every product has a center, and we don't need centers with no sales, so inner joins fit.

```sql
SELECT dc.name AS distribution_center, ROUND(SUM(i.sale_price), 2) AS revenue
FROM `bigquery-public-data.thelook_ecommerce.order_items` AS i
INNER JOIN `bigquery-public-data.thelook_ecommerce.products` AS p
  ON i.product_id = p.id
INNER JOIN `bigquery-public-data.thelook_ecommerce.distribution_centers` AS dc
  ON p.distribution_center_id = dc.id
WHERE i.status NOT IN ('Cancelled', 'Returned')
  AND i.created_at >= '2026-07-01'
  AND i.created_at < '2026-10-01'
GROUP BY dc.name
ORDER BY revenue DESC;
```

Sample result: Houston TX, then Memphis TN and Chicago IL.

### 5. First-week conversion: left join, filter in ON

The denominator is *all* September signups, including the ones who never ordered. That means `users` on the left and a left join.

The 7-day rule is a condition on the right table. If it went in WHERE, users with no order would have a NULL `o.created_at`, fail the test and vanish, which would make every percentage 100%. In ON, it only limits which orders can match.

```sql
SELECT
  u.traffic_source,
  COUNT(DISTINCT u.id) AS signups,
  COUNT(DISTINCT o.user_id) AS ordered_within_7_days,
  ROUND(100 * COUNT(DISTINCT o.user_id) / COUNT(DISTINCT u.id), 1) AS pct_ordered
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
LEFT JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o
  ON u.id = o.user_id
  AND o.created_at < TIMESTAMP_ADD(u.created_at, INTERVAL 7 DAY)
WHERE u.created_at >= '2026-09-01'
  AND u.created_at < '2026-10-01'
GROUP BY u.traffic_source
ORDER BY signups DESC;
```

Sample result:

| traffic_source | signups | ordered_within_7_days | pct_ordered |
| --- | --- | --- | --- |
| Search | 727 | 128 | 17.6 |
| Organic | 151 | 34 | 22.5 |
| Facebook | 71 | 21 | 29.6 |
| Email | 44 | 7 | 15.9 |
| Display | 28 | 7 | 25.0 |

The filters on `u.created_at` stay in WHERE because they're on the left table.

### 6. Slow weeks: self join

This is LeetCode 197 with weeks instead of days. Build a weekly table in a CTE, then join it to itself, matching each week to the week before it.

```sql
WITH weekly AS (
  SELECT DATE_TRUNC(DATE(created_at), WEEK(MONDAY)) AS week_start, COUNT(*) AS signups
  FROM `bigquery-public-data.thelook_ecommerce.users`
  WHERE created_at >= '2026-01-05'
    AND created_at < '2026-10-26'
  GROUP BY week_start
)
SELECT this_week.week_start, last_week.signups AS last_week_signups, this_week.signups AS this_week_signups
FROM weekly AS this_week
INNER JOIN weekly AS last_week
  ON last_week.week_start = DATE_SUB(this_week.week_start, INTERVAL 1 WEEK)
WHERE this_week.signups < last_week.signups
ORDER BY this_week.week_start;
```

Sample result: 20 weeks, starting with the week of January 19 (254 down to 244).

- January 5, 2026 is the first Monday of the year, and October 26 is the Monday after the last full week. Using whole weeks keeps a partial week from looking like a slowdown.
- Join on the date, not on a row number.

### 7. Quiet jeans brands: left join, filter in ON

Brands with zero sales must appear, so `products` goes on the left. The September and status rules apply to `order_items`, the right table, so they go in ON. If they were in WHERE, the zero-sale brands would disappear.

```sql
SELECT p.brand, COUNT(i.id) AS units_sold
FROM `bigquery-public-data.thelook_ecommerce.products` AS p
LEFT JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS i
  ON p.id = i.product_id
  AND i.status NOT IN ('Cancelled', 'Returned')
  AND i.created_at >= '2026-09-01'
  AND i.created_at < '2026-10-01'
WHERE p.category = 'Jeans'
GROUP BY p.brand
ORDER BY units_sold, p.brand;
```

Sample result: 223 jeans brands, 118 of them with 0 units sold.

- `COUNT(i.id)`, not `COUNT(*)`. A brand with no sales still has one row with NULLs, and `COUNT(*)` would count it as 1.
- `p.category = 'Jeans'` stays in WHERE because it's on the left table.
- One "brand" is NULL. That's a data quality issue worth pointing out to the buyer.

### 8. Double orders: self join

We're pairing each order with another order from the same table, same customer, same day.

```sql
SELECT
  a.user_id,
  DATE(a.created_at) AS order_date,
  a.order_id AS first_order_id,
  b.order_id AS second_order_id
FROM `bigquery-public-data.thelook_ecommerce.orders` AS a
INNER JOIN `bigquery-public-data.thelook_ecommerce.orders` AS b
  ON a.user_id = b.user_id
  AND DATE(a.created_at) = DATE(b.created_at)
  AND a.order_id < b.order_id
WHERE a.created_at >= '2026-09-01'
  AND a.created_at < '2026-10-01'
ORDER BY order_date, a.user_id;
```

`a.order_id < b.order_id` does two jobs:

- It stops each order from pairing with itself. With `=` or no condition, every order in September would show up.
- It shows each pair once. With `<>`, you'd get both (106003, 106006) and (106006, 106003).

A customer with three orders on one day shows up as three pairs: (1, 2), (1, 3) and (2, 3).
