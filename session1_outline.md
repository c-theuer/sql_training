# SQL Training Session 1: Joins

## Where We Are

In Session 0 we set up BigQuery and saw how a join can quietly double-count. Since then, you've worked through filtering, GROUP BY, CASE WHEN, dates, CTEs and a first look at LEFT vs INNER joins in the other instructor's sessions. Tonight we go deeper on the three joins you'll use most and practice choosing between them.

## Today's Goal

Given a business question, you can decide whether it needs an **inner join**, a **left join** or a **self join**, and write the query in BigQuery.

## The Tables We'll Use

All tables live in `bigquery-public-data.thelook_ecommerce`. These are the keys that connect them:

| From | Column | Matches | Relationship |
| --- | --- | --- | --- |
| `orders` | `user_id` | `users.id` | One user has many orders |
| `order_items` | `order_id` | `orders.order_id` | One order has many items |
| `order_items` | `user_id` | `users.id` | One user has many items |
| `order_items` | `product_id` | `products.id` | One product appears in many items |
| `products` | `distribution_center_id` | `distribution_centers.id` | One center ships many products |

Remember the Session 0 lesson: when you join a "one" table to a "many" table, each "one" row repeats once per match. Watch what you SUM.

## Inner Join

**An inner join keeps only rows that have a match in both tables.** Anything without a partner disappears.

This toy example runs in BigQuery with no tables. It builds two tiny tables with a CTE so you can see exactly which rows survive:

```sql
WITH customers AS (
  SELECT 1 AS customer_id, 'Ana' AS name UNION ALL
  SELECT 2, 'Ben' UNION ALL
  SELECT 3, 'Cal'
),
orders AS (
  SELECT 101 AS order_id, 1 AS customer_id, 50 AS amount UNION ALL
  SELECT 102, 1, 20 UNION ALL
  SELECT 103, 2, 35 UNION ALL
  SELECT 104, 9, 80
)
SELECT c.name, o.order_id, o.amount
FROM customers AS c
INNER JOIN orders AS o
  ON c.customer_id = o.customer_id
ORDER BY o.order_id;
```

| name | order_id | amount |
| --- | --- | --- |
| Ana | 101 | 50 |
| Ana | 102 | 20 |
| Ben | 103 | 35 |

- Cal has no orders, so Cal is gone.
- Order 104 belongs to customer 9, who doesn't exist, so it's gone too.
- Ana appears twice because she has two orders.

**On theLook:** orders placed in September 2026 by customer country.

```sql
SELECT u.country, COUNT(*) AS orders
FROM `bigquery-public-data.thelook_ecommerce.orders` AS o
INNER JOIN `bigquery-public-data.thelook_ecommerce.users` AS u
  ON o.user_id = u.id
WHERE o.created_at >= '2026-09-01'
  AND o.created_at < '2026-10-01'
GROUP BY u.country
ORDER BY orders DESC
LIMIT 5;
```

**Use it when** you only care about things that exist on both sides, like "revenue by product category." A sale with no product isn't something you want to report.

`JOIN` with no keyword is the same as `INNER JOIN`.

## Left Join

**A left join keeps every row from the left table (the one after FROM).** When there's no match on the right, the right side's columns come back as NULL.

Same toy tables, one word changed:

```sql
-- same WITH block as above
SELECT c.name, o.order_id, o.amount
FROM customers AS c
LEFT JOIN orders AS o
  ON c.customer_id = o.customer_id
ORDER BY o.order_id;
```

| name | order_id | amount |
| --- | --- | --- |
| Cal | NULL | NULL |
| Ana | 101 | 50 |
| Ana | 102 | 20 |
| Ben | 103 | 35 |

Cal stays, with NULLs. Order 104 is still gone, because it's on the right side and only the left side is protected.

### Pattern 1: Find what's missing

Add `WHERE right_table.key IS NULL` to keep only the rows that didn't match. This finds October 2026 signups who have never ordered:

```sql
SELECT u.id, u.first_name, u.created_at
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
LEFT JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o
  ON u.id = o.user_id
WHERE u.created_at >= '2026-10-01'
  AND o.order_id IS NULL
ORDER BY u.created_at
LIMIT 10;
```

### Pattern 2: Keep the zeros

When you're computing a rate or a list that should include zeros, a left join keeps the denominator honest:

```sql
SELECT
  COUNT(DISTINCT u.id) AS october_signups,
  COUNT(DISTINCT o.user_id) AS signups_with_an_order
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
LEFT JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o
  ON u.id = o.user_id
WHERE u.created_at >= '2026-10-01';
```

`COUNT(column)` skips NULLs, so `COUNT(DISTINCT o.user_id)` counts only signups that matched an order.

### The trap: WHERE vs ON

A filter on the **right** table in the WHERE clause throws away the NULL rows and turns your left join back into an inner join. If you want to limit which right-side rows can match, put the condition in the **ON** clause instead.

```sql
-- Drops users with no September order: WHERE runs after the join
LEFT JOIN orders AS o ON u.id = o.user_id
WHERE o.created_at >= '2026-09-01'

-- Keeps every user, and only September orders can match
LEFT JOIN orders AS o
  ON u.id = o.user_id
  AND o.created_at >= '2026-09-01'
```

Filters on the **left** table belong in WHERE as usual.

## Self Join

**A self join joins a table to itself.** You give the same table two aliases so it can play two roles, like "today" and "yesterday," or "first order" and "next order."

### LeetCode 197: Rising Temperature

> Given a `Weather` table with `id`, `recordDate` and `temperature`, find the `id` of every date whose temperature is higher than the previous day's.

Each row needs to be compared to a *different row in the same table*. That's the signal for a self join.

```sql
WITH weather AS (
  SELECT 1 AS id, DATE '2015-01-01' AS record_date, 10 AS temperature UNION ALL
  SELECT 2, DATE '2015-01-02', 25 UNION ALL
  SELECT 3, DATE '2015-01-03', 20 UNION ALL
  SELECT 4, DATE '2015-01-04', 30
)
SELECT today.id
FROM weather AS today
INNER JOIN weather AS yesterday
  ON yesterday.record_date = DATE_SUB(today.record_date, INTERVAL 1 DAY)
WHERE today.temperature > yesterday.temperature;
```

Result: ids 2 and 4.

- **Join on the date, not the id.** `yesterday.id = today.id - 1` is a common wrong answer. Ids can have gaps or be out of date order.
- The first day has no "yesterday," so the inner join drops it. That's what we want.

### The same idea on theLook

Which days in September 2026 had more revenue than the day before? First build a daily table with a CTE, then join it to itself:

```sql
WITH daily AS (
  SELECT DATE(created_at) AS sale_date, ROUND(SUM(sale_price), 2) AS revenue
  FROM `bigquery-public-data.thelook_ecommerce.order_items`
  WHERE status NOT IN ('Cancelled', 'Returned')
    AND created_at >= '2026-09-01'
    AND created_at < '2026-10-01'
  GROUP BY sale_date
)
SELECT today.sale_date, yesterday.revenue AS yesterday_revenue, today.revenue AS today_revenue
FROM daily AS today
INNER JOIN daily AS yesterday
  ON yesterday.sale_date = DATE_SUB(today.sale_date, INTERVAL 1 DAY)
WHERE today.revenue > yesterday.revenue
ORDER BY today.sale_date;
```

### Watch out: a row can match itself

When you join a table to itself, every row is a candidate partner for itself. Add a condition that rules it out, such as `a.order_id < b.order_id` (which also stops each pair from showing up twice) or `b.created_at > a.created_at`.

## Choosing a Join

Ask these questions in order:

1. **Am I comparing rows of a table to other rows of the same table?** (this day vs the day before, one order vs the same customer's next order) → **self join**
2. **Should rows with no match still show up?** (customers who never ordered, products with zero sales, a rate where the denominator must include everyone) → **left join**, with the table you want to keep on the left
3. **Otherwise** → **inner join**

Before you trust a number, check your row counts. Did the join drop rows you needed, or multiply rows you're summing?

## More Info

- [LeetCode 197: Rising Temperature](https://leetcode.com/problems/rising-temperature/)
- [SQLZoo: The JOIN operation](https://sqlzoo.net/wiki/The_JOIN_operation)
- [BigQuery JOIN reference](https://cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax#join_types)

## Today's Exercises

Each question needs at least one join. Figure out which kind (inner, left or self) fits the question, then write the query. Some need more than one join.

All tables are in `bigquery-public-data.thelook_ecommerce`. "Revenue" means the sum of `order_items.sale_price`.

1. **Top categories.** The merchandising team wants the top 5 product categories by revenue in September 2026, counting only order items with a status of `Complete`.

2. **Dead stock.** The buyer wants to clear out products that nobody has ever ordered. How many never-ordered products are there in each department?

3. **Coming back.** Marketing wants to know how loyal recent customers are. Of the orders placed between January 1 and June 30, 2026, how many distinct customers placed another order within 30 days after one of those orders?

4. **Busiest warehouse.** Operations is planning staffing. Which distribution center's products brought in the most revenue in Q3 2026 (July 1 through September 30)? Leave out order items that were cancelled or returned.

5. **First-week conversion.** For users who signed up in September 2026, show each traffic source with the number of signups, the number who placed an order within 7 days of signing up, and that percentage. Every traffic source with signups should appear.

6. **Slow weeks.** The growth team wants to spot slowdowns. List every week in 2026 (weeks start Monday, through the week of October 19) where new user signups were lower than the week before. Show both weeks' counts.

7. **Quiet jeans brands.** The jeans buyer wants every brand in the `Jeans` category with the number of units sold in September 2026, not counting cancelled or returned items. Brands that sold nothing should show 0, and the list should start with them.

8. **Double orders.** Customer support suspects a checkout bug is creating duplicate orders. Find every customer who placed two or more orders on the same calendar day in September 2026. Show the customer, the date and each pair of order ids once.
