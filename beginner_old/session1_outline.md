# Session 1: Getting Data Out

**Dataset:** theLook eCommerce on BigQuery (`bigquery-public-data.thelook_ecommerce`)
**Length:** 1 hour

## By the end of this session you can

- Find a table in BigQuery and read its schema
- Pull specific columns from a table
- Filter rows with `WHERE`
- Sort and limit results
- List the unique values in a column

## 1. Setup check

1. Open console.cloud.google.com/bigquery and sign in.
2. In the Explorer panel, find `bigquery-public-data` > `thelook_ecommerce`.
3. Run this. Seeing 10 rows means you're ready.

```sql
SELECT *
FROM `bigquery-public-data.thelook_ecommerce.users`
LIMIT 10;
```

## 2. The dataset

theLook is a fictional online clothing store. Seven tables:

| Table | One row = | Useful columns |
| --- | --- | --- |
| `users` | one customer | `id`, `first_name`, `age`, `gender`, `country`, `traffic_source` |
| `orders` | one order | `order_id`, `user_id`, `status`, `created_at`, `num_of_item` |
| `order_items` | one product in one order | `order_id`, `user_id`, `product_id`, `sale_price`, `status` |
| `products` | one product | `id`, `name`, `brand`, `category`, `department`, `cost`, `retail_price` |
| `inventory_items` | one physical unit in stock | `product_id`, `sold_at`, `cost` |
| `distribution_centers` | one warehouse | `id`, `name` |
| `events` | one website page view | `user_id`, `session_id`, `event_type`, `uri` |

**Habit:** before querying a table, open its **Schema** and **Preview** tabs. Ask yourself: "what does one row mean?"

## 3. Anatomy of a query

```sql
SELECT   first_name, age, country          -- which columns
FROM     `bigquery-public-data.thelook_ecommerce.users`  -- which table
WHERE    country = 'Brazil'                 -- which rows
ORDER BY age DESC                           -- in what order
LIMIT    10;                                -- how many
```

The clauses always go in this order: `SELECT`, `FROM`, `WHERE`, `ORDER BY`, `LIMIT`.

## 4. Key concepts

**Pick columns.** List them with commas, or use `*` for all.

**Aliases.** Rename a column in the output: `retail_price AS price`.

**Filtering with WHERE**

| Operator | Example |
| --- | --- |
| `=`, `<>` | `country = 'Brazil'` |
| `>`, `<`, `>=`, `<=` | `age >= 30` |
| `BETWEEN` | `age BETWEEN 18 AND 25` (includes both ends) |
| `IN` | `traffic_source IN ('Email', 'Search')` |
| `LIKE` | `name LIKE '%Jacket%'` (`%` = any characters) |
| `IS NULL`, `IS NOT NULL` | `returned_at IS NULL` |

**Combining conditions.** Use `AND` and `OR`. When you mix them, use parentheses:
`WHERE country = 'Brazil' AND (age < 20 OR age > 60)`

**Sorting.** `ORDER BY col ASC` (smallest first, the default) or `DESC` (largest first).

**DISTINCT.** `SELECT DISTINCT country` returns each value once.

## 5. Tips

- Text values use **single quotes**: `'Jeans'`, not `"Jeans"`.
- Text matching is **case-sensitive**. Check exact values with `SELECT DISTINCT` first.
- `NULL` means unknown. Use `IS NULL`, never `= NULL`.
- BigQuery charges by columns scanned, not rows returned. Select only the columns you need.

## 6. Exercises

**1.1 First look.** Show the `id`, `first_name`, `last_name`, `age` and `country` of 20 users.

**1.2 One filter.** List all products in the `Jeans` category. Show `name`, `brand` and `retail_price`.

**1.3 Sorting.** Show the 10 most expensive products by `retail_price`, highest first.

**1.4 Two conditions.** Find female users (`gender = 'F'`) aged 18 to 25 who live in the United States.

**1.5 IN.** List users whose `traffic_source` is either `Email` or `Facebook`. Show `email`, `traffic_source` and `country`.

**1.6 Pattern matching.** Find products whose `name` contains the word "Hoodie". Watch the capital letter.

**1.7 DISTINCT.** What are all the possible values of `status` in the `orders` table?

**1.8 Combine it.** Show the 15 cheapest products in the `Women` department that cost more than $50. Show `name`, `category` and `retail_price`, cheapest first.

**Stretch.** Which orders have never shipped, meaning `shipped_at` is empty? Why doesn't `= NULL` work?

## 7. Debrief questions

- What does one row in `orders` represent? In `order_items`?
- Why might a query return zero rows when you expected some?
- Why doesn't `LIMIT` make a query cheaper?

## 8. Post Class Survey

Please fill out this short, anonymous [feedback survey](https://docs.google.com/forms/d/e/1FAIpQLSe3E7PFT4MoT2F-99V1-FKeXkiTLqQRLj1cZO8kREXkqdyrmg/viewform) on the first two sessions. It takes about 3 minutes.
