# Session 1 Answers: Getting Data Out

theLook data is regenerated regularly, so row counts may differ from run to run. Other correct queries exist; these are one good answer each.

## 1.1 First look

```sql
SELECT id, first_name, last_name, age, country
FROM `bigquery-public-data.thelook_ecommerce.users`
LIMIT 20;
```

## 1.2 One filter

```sql
SELECT name, brand, retail_price
FROM `bigquery-public-data.thelook_ecommerce.products`
WHERE category = 'Jeans';
```

## 1.3 Sorting

```sql
SELECT name, retail_price
FROM `bigquery-public-data.thelook_ecommerce.products`
ORDER BY retail_price DESC
LIMIT 10;
```

## 1.4 Two conditions

```sql
SELECT first_name, last_name, age, country
FROM `bigquery-public-data.thelook_ecommerce.users`
WHERE gender = 'F'
  AND age BETWEEN 18 AND 25
  AND country = 'United States';
```

Note: `BETWEEN` includes both 18 and 25.

## 1.5 IN

```sql
SELECT email, traffic_source, country
FROM `bigquery-public-data.thelook_ecommerce.users`
WHERE traffic_source IN ('Email', 'Facebook');
```

Equivalent: `WHERE traffic_source = 'Email' OR traffic_source = 'Facebook'`.

## 1.6 Pattern matching

```sql
SELECT name, category
FROM `bigquery-public-data.thelook_ecommerce.products`
WHERE name LIKE '%Hoodie%';
```

`LIKE` is case-sensitive in BigQuery, so this misses "hoodie" and "HOODIE". To catch every spelling:

```sql
WHERE LOWER(name) LIKE '%hoodie%'
```

## 1.7 DISTINCT

```sql
SELECT DISTINCT status
FROM `bigquery-public-data.thelook_ecommerce.orders`;
```

Expected values: `Processing`, `Shipped`, `Complete`, `Cancelled`, `Returned`.

## 1.8 Combine it

```sql
SELECT name, category, retail_price
FROM `bigquery-public-data.thelook_ecommerce.products`
WHERE department = 'Women'
  AND retail_price > 50
ORDER BY retail_price ASC
LIMIT 15;
```

## Stretch

```sql
SELECT order_id, status, created_at
FROM `bigquery-public-data.thelook_ecommerce.orders`
WHERE shipped_at IS NULL;
```

Why `= NULL` fails: `NULL` means "unknown," and comparing anything to unknown gives unknown, never true. So `WHERE shipped_at = NULL` returns zero rows. Always use `IS NULL` or `IS NOT NULL`.

Discussion: check the `status` of these orders. They should mostly be `Processing` or `Cancelled`, which makes sense for orders that never shipped.
