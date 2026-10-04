# 5. Joins and relationships

[Back to notes index](../README.md)

| [Previous: Expressions, functions, and dates](04-expressions-functions-and-dates.md) | [Notes index](../README.md) | [Next: Aggregation and grouping](06-aggregation-and-grouping.md) |
|:--|:--:|--:|

This chapter connects rows across tables with join conditions and explains how relationship shape affects results.

## In this chapter

- Inner and outer joins
- Joining more than two tables
- Self joins and aliases
- Avoiding accidental row multiplication
- Eight review questions

## Match related rows with INNER JOIN

An inner join returns a row for every pair of rows whose join condition is true. In a one-to-many relationship, one customer can therefore appear once for each matching order.

~~~sql
SELECT c.id AS customer_id,
       c.full_name,
       o.id AS order_id,
       o.status,
       o.created_at
FROM store.customers AS c
JOIN store.orders AS o
  ON o.customer_id = c.id
ORDER BY o.created_at DESC, o.id DESC;
~~~

Use table aliases to make each column's source clear. Qualify columns when more than one joined table has the same column name.

## Keep unmatched rows with LEFT JOIN

A left join returns every row from its left input. When the right side has no match, its selected columns are NULL.

~~~sql
SELECT c.id,
       c.full_name,
       o.id AS order_id
FROM store.customers AS c
LEFT JOIN store.orders AS o
  ON o.customer_id = c.id
ORDER BY c.id, o.id;
~~~

To find customers with no orders, test a right-side column that cannot be NULL for a real matching row:

~~~sql
SELECT c.id, c.full_name
FROM store.customers AS c
LEFT JOIN store.orders AS o
  ON o.customer_id = c.id
WHERE o.id IS NULL
ORDER BY c.id;
~~~

## Join several tables

Follow the foreign-key path one relationship at a time. Each order item points to one product, so the result contains one row per product line.

~~~sql
SELECT o.id AS order_id,
       c.full_name AS customer,
       p.sku,
       p.name AS product,
       oi.quantity,
       oi.unit_price,
       oi.quantity * oi.unit_price AS line_total
FROM store.orders AS o
JOIN store.customers AS c
  ON c.id = o.customer_id
JOIN store.order_items AS oi
  ON oi.order_id = o.id
JOIN store.products AS p
  ON p.id = oi.product_id
ORDER BY o.id, p.id;
~~~

If a parent order should remain visible when it has no line items, left join the item and product tables. Keep filters for optional right-side rows in the ON clause:

~~~sql
SELECT o.id,
       oi.product_id,
       oi.quantity
FROM store.orders AS o
LEFT JOIN store.order_items AS oi
  ON oi.order_id = o.id
 AND oi.quantity > 1
ORDER BY o.id, oi.product_id;
~~~

Moving oi.quantity > 1 into WHERE would remove the NULL-extended rows and make the result behave like an inner join for that condition.

## Check row counts after joins

Before adding another table, ask what one output row represents. A join can multiply rows when both sides have several matches. Summing an order-level value after joining order items can count the same order total once for every line. Aggregate at the correct level first or sum line values at the line level.

COUNT(*) counts result rows. COUNT(o.id) counts matching non-null order identifiers. These can differ after an outer join.

~~~sql
SELECT c.id,
       count(o.id) AS number_of_orders
FROM store.customers AS c
LEFT JOIN store.orders AS o
  ON o.customer_id = c.id
GROUP BY c.id
ORDER BY c.id;
~~~

## Review questions

1. What does an inner join return?
2. What does a left join put in right-side columns when there is no match?
3. How can a query find customers who have no orders?
4. Why are table aliases useful in a multi-table query?
5. Why can a one-to-many join repeat the parent row?
6. What can happen if a filter on the right side of a left join is placed in WHERE?
7. Why can summing an order-level value after joining line items produce an inflated total?
8. What is the difference between COUNT(*) and COUNT(o.id) after a left join?

## References

- [PostgreSQL: Joined tables](https://www.postgresql.org/docs/18/queries-table-expressions.html#QUERIES-JOIN)
- [PostgreSQL: Aggregate functions](https://www.postgresql.org/docs/18/functions-aggregate.html)

---

| [Previous: Expressions, functions, and dates](04-expressions-functions-and-dates.md) | [Notes index](../README.md) | [Next: Aggregation and grouping](06-aggregation-and-grouping.md) |
