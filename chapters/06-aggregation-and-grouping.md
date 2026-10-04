# 6. Aggregation and grouping

[Back to notes index](../README.md)

| [Previous: Joins and relationships](05-joins-and-relations.md) | [Notes index](../README.md) | [Next: Subqueries, CTEs, and set operations](07-subqueries-ctes-and-set-operations.md) |
|:--|:--:|--:|

This chapter summarizes rows with aggregate functions and groups results at a chosen level.

## In this chapter

- `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX`
- `GROUP BY` and `HAVING`
- Aggregate behavior with `NULL`
- Filtering before and after aggregation
- Eight review questions

## Aggregate rows into one result

An aggregate function summarizes several input rows. COUNT(*) counts rows, COUNT(column) counts non-null values, and COUNT(DISTINCT column) counts unique non-null values.

~~~sql
SELECT count(*) AS product_rows,
       count(description) AS products_with_description,
       count(DISTINCT active) AS distinct_active_values
FROM store.products;
~~~

The products table does not yet have a description column, so add it in Chapter 11 before running the second expression. The example illustrates why a query must match the actual schema.

SUM and AVG ignore NULL inputs and return NULL when there are no non-null input values. COALESCE can display zero for an empty total when zero is the correct business meaning.

~~~sql
SELECT coalesce(sum(price), 0::numeric) AS total_catalog_value,
       avg(price) AS average_price,
       min(price) AS lowest_price,
       max(price) AS highest_price
FROM store.products
WHERE active;
~~~

## Group rows

GROUP BY creates one result row for each distinct group. Every selected expression must either be grouped or aggregated.

~~~sql
SELECT status,
       count(*) AS order_count
FROM store.orders
GROUP BY status
ORDER BY order_count DESC, status;
~~~

Aggregate across related tables at the level the question asks for. This query returns one row per customer, including customers without orders:

~~~sql
SELECT c.id,
       c.full_name,
       count(o.id) AS order_count,
       coalesce(sum(oi.quantity * oi.unit_price), 0::numeric) AS amount_ordered
FROM store.customers AS c
LEFT JOIN store.orders AS o
  ON o.customer_id = c.id
LEFT JOIN store.order_items AS oi
  ON oi.order_id = o.id
GROUP BY c.id, c.full_name
ORDER BY amount_ordered DESC, c.id;
~~~

COUNT(o.id) returns zero for a customer with no orders. COUNT(*) would count the NULL-extended left-join row and return one. The line total is calculated once per order item.

## Filter groups and use conditional aggregates

WHERE filters input rows before grouping. HAVING filters completed groups after aggregate values exist.

~~~sql
SELECT customer_id,
       count(*) AS order_count
FROM store.orders
WHERE created_at >= current_date - 30
GROUP BY customer_id
HAVING count(*) >= 2
ORDER BY order_count DESC, customer_id;
~~~

Aggregate FILTER applies a condition to one aggregate without removing rows from other aggregates in the same group:

~~~sql
SELECT customer_id,
       count(*) AS all_orders,
       count(*) FILTER (WHERE status = 'paid') AS paid_orders,
       count(*) FILTER (WHERE status = 'cancelled') AS cancelled_orders
FROM store.orders
GROUP BY customer_id
ORDER BY customer_id;
~~~

An aggregate FILTER condition is not a substitute for choosing the correct rows for every selected aggregate. Read each aggregate independently.

## Review questions

1. What is the difference between COUNT(*) and COUNT(column)?
2. Does COUNT(DISTINCT column) count NULL?
3. What do SUM and AVG return when there are no non-null input values?
4. What does GROUP BY do to the input rows?
5. How are WHERE and HAVING different?
6. Why can COUNT(*) return one for a parent with no matches after a left join?
7. What does the FILTER clause on an aggregate do?
8. Why should a total be calculated at the same level as the rows being summed?

## References

- [PostgreSQL: Aggregate functions](https://www.postgresql.org/docs/18/functions-aggregate.html)
- [PostgreSQL: GROUP BY and HAVING](https://www.postgresql.org/docs/18/queries-table-expressions.html#QUERIES-GROUP)

---

| [Previous: Joins and relationships](05-joins-and-relations.md) | [Notes index](../README.md) | [Next: Subqueries, CTEs, and set operations](07-subqueries-ctes-and-set-operations.md) |
|:--|:--:|--:|
