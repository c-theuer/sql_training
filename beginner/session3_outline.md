# Session 3: Combining Tables with JOINs

**Dataset:** theLook eCommerce on BigQuery (`bigquery-public-data.thelook_ecommerce`)
**Length:** 2 hours

## By the end of this session you can

- Explain primary keys and foreign keys
- Combine two or three tables with `INNER JOIN` and `LEFT JOIN`
- Find records with no match
- Spot and fix double-counting caused by joins

## Agenda

| Time | Activity |
| --- | --- |
| 0:00 | Recap and warm-up |
| 0:10 | How the tables connect |
| 0:20 | Live demo: INNER JOIN, LEFT JOIN, the grain trap |
| 0:50 | Exercises in pairs |
| 1:40 | Share-outs and debrief |

## 1. Warm-up

How many products are in each department? Write it from memory.

## 2. How the tables connect

A **primary key** uniquely identifies each row (`users.id`). A **foreign key** points to another table's primary key (`orders.user_id` points to `users.id`).

| From | Column | Points to |
| --- | --- | --- |
| `orders` | `user_id` | `users.id` |
| `order_items` | `order_id` | `orders.order_id` |
| `order_items` | `user_id` | `users.id` |
| `order_items` | `product_id` | `products.id` |
| `order_items` | `inventory_item_id` | `inventory_items.id` |
| `inventory_items` | `product_id` | `products.id` |
| `products` | `distribution_center_id` | `distribution_centers.id` |
| `events` | `user_id` | `users.id` |

`order_items` is the hub: most business questions start there.

## 3. INNER JOIN

Keeps only rows that match in **both** tables.

```sql
SELECT oi.order_id, oi.sale_price, p.name, p.brand
FROM `bigquery-public-data.thelook_ecommerce.order_items` AS oi
JOIN `bigquery-public-data.thelook_ecommerce.products` AS p
  ON oi.product_id = p.id
LIMIT 10;
```

- `JOIN` and `INNER JOIN` mean the same thing.
- Short aliases (`oi`, `p`) keep queries readable. Prefix columns with the alias.

## 4. LEFT JOIN

Keeps **every** row from the left table, even with no match. Missing values come back as `NULL`.

```sql
SELECT u.id, u.first_name, o.order_id
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
LEFT JOIN `bigquery-public-data.thelook_ecommerce.orders` AS o
  ON u.id = o.user_id
LIMIT 20;
```

To find rows with **no** match, add `WHERE o.order_id IS NULL`.

## 5. Joining three tables

Chain joins one after another:

```sql
FROM order_items AS oi
JOIN products AS p ON oi.product_id = p.id
JOIN users    AS u ON oi.user_id    = u.id
```

(Use the full backticked paths in real queries.)

## 6. The grain trap

After a join, ask: **"what does one row mean now?"**

Joining `orders` to `order_items` gives one row per **item**, not per order. An order with 3 items appears 3 times. Counting rows or summing order-level values now gives inflated totals. Fix it with `COUNT(DISTINCT order_id)` or by joining at the right level.

## 7. Tips

- Compare row counts before and after a join. A big jump is a warning sign.
- An `INNER JOIN` silently drops rows with no match. If you need them, use `LEFT JOIN`.
- When a filter is on the right table of a `LEFT JOIN`, think carefully: a `WHERE` on that table can turn it back into an inner join.

## 8. Exercises

**3.1 First join.** For 20 order items, show `order_id`, `sale_price`, and the product `name` and `brand`.

**3.2 Join + filter.** List order items for products in the `Swim` category, with the buyer's `first_name` and `country`.

**3.3 Join + aggregate.** Total revenue by product `category`, using only items with status `Complete`. Highest first.

**3.4 Three tables.** Revenue by user `country` and product `department` for completed items.

**3.5 LEFT JOIN.** How many users have never placed an order?

**3.6 Fix the bug.** This query should give the number of orders per country, but the totals are too high. Find and fix the problem.

```sql
SELECT u.country, COUNT(*) AS num_orders
FROM `bigquery-public-data.thelook_ecommerce.users` AS u
JOIN `bigquery-public-data.thelook_ecommerce.order_items` AS oi
  ON u.id = oi.user_id
GROUP BY u.country;
```

**3.7 Distribution centers.** Count products by distribution center `name`.

**Stretch.** Profit by brand: `SUM(sale_price - cost)` for completed items (cost is in `products`). Which 10 brands earn the most profit? Do they match the 10 brands with the most revenue?

## 9. Debrief questions

- When would you pick `LEFT JOIN` over `INNER JOIN`?
- What was wrong with the query in 3.6, and how did you know?
- Which table would you start from to answer "what's our best-selling brand?"
