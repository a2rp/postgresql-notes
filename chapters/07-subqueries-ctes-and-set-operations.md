# 7. Subqueries, CTEs, and set operations

[Back to notes index](../README.md)

| [Previous: Aggregation and grouping](06-aggregation-and-grouping.md) | [Notes index](../README.md) | [Next: Inserts, updates, deletes, and transactions](08-inserts-updates-deletes-and-transactions.md) |
|:--|:--:|--:|

This chapter organizes multi-step queries with subqueries and common table expressions, then combines compatible results.

## In this chapter

- Scalar and correlated subqueries
- `EXISTS` and membership checks
- CTEs and recursive query basics
- `UNION`, `INTERSECT`, and `EXCEPT`
- Eight review questions

## Subqueries for one value or a set

A scalar subquery returns one column and at most one row. It can be used where a single value is expected.

~~~sql
SELECT id, name, price
FROM store.products
WHERE price > (
    SELECT avg(price)
    FROM store.products
)
ORDER BY price DESC, id;
~~~

If a scalar subquery returns more than one row, PostgreSQL raises an error. Use IN, ANY, or a join when multiple values are intended.

EXISTS checks whether a subquery returns at least one row. It avoids multiplying customer rows when the question is only whether a related order exists.

~~~sql
SELECT c.id, c.full_name
FROM store.customers AS c
WHERE EXISTS (
    SELECT 1
    FROM store.orders AS o
    WHERE o.customer_id = c.id
      AND o.status = 'paid'
)
ORDER BY c.id;
~~~

NOT EXISTS is a clear way to find rows with no matching child:

~~~sql
SELECT p.id, p.sku, p.name
FROM store.products AS p
WHERE NOT EXISTS (
    SELECT 1
    FROM store.order_items AS oi
    WHERE oi.product_id = p.id
)
ORDER BY p.id;
~~~

NOT IN can produce surprising results if its input set contains NULL. Prefer NOT EXISTS for anti-join logic unless NULL behavior is explicitly handled.

## Name query steps with CTEs

A common table expression names a query result for one statement. It can make multi-step logic easier to read:

~~~sql
WITH customer_totals AS (
    SELECT o.customer_id,
           sum(oi.quantity * oi.unit_price) AS amount
    FROM store.orders AS o
    JOIN store.order_items AS oi
      ON oi.order_id = o.id
    GROUP BY o.customer_id
)
SELECT c.id, c.full_name, t.amount
FROM customer_totals AS t
JOIN store.customers AS c
  ON c.id = t.customer_id
WHERE t.amount >= 100
ORDER BY t.amount DESC, c.id;
~~~

A non-recursive CTE is a readability tool. PostgreSQL may fold a CTE into the surrounding query when that is safe; it is not always a temporary table.

## Walk a hierarchy with a recursive CTE

A recursive CTE has an anchor query and a recursive query. UNION ALL repeats the recursive part until it finds no more rows.

~~~sql
WITH RECURSIVE category_source(id, parent_id, name) AS (
    VALUES
        (1, NULL::integer, 'Accessories'),
        (2, 1, 'Bags'),
        (3, 1, 'Drinkware'),
        (4, 2, 'Travel bags')
),
category_tree(id, parent_id, name, depth) AS (
    SELECT id, parent_id, name, 0
    FROM category_source
    WHERE parent_id IS NULL

    UNION ALL

    SELECT child.id, child.parent_id, child.name, parent.depth + 1
    FROM category_tree AS parent
    JOIN category_source AS child
      ON child.parent_id = parent.id
)
SELECT id, parent_id, name, depth
FROM category_tree
ORDER BY depth, id;
~~~

Real hierarchies should prevent cycles or include a visited-path check so bad parent data cannot recurse forever.

## Combine compatible results

UNION combines result sets and removes duplicates. UNION ALL keeps duplicates and usually does less work. The two queries must return the same number of columns with compatible types in each position.

~~~sql
SELECT email AS contact
FROM store.customers
UNION
SELECT 'support@example.test'::text AS contact
ORDER BY contact;
~~~

INTERSECT returns rows present in both results. EXCEPT returns rows from the first result that are absent from the second. Use these when set semantics describe the task more clearly than a join.

## Review questions

1. What shape must a scalar subquery return?
2. When is EXISTS clearer than joining a related table?
3. Why can NOT IN behave unexpectedly if its input contains NULL?
4. What is a CTE?
5. Is every non-recursive CTE materialized as a temporary table?
6. What are the anchor and recursive parts of a recursive CTE?
7. What is the difference between UNION and UNION ALL?
8. What does EXCEPT return?

## References

- [PostgreSQL: Subquery expressions](https://www.postgresql.org/docs/18/functions-subquery.html)
- [PostgreSQL: WITH queries](https://www.postgresql.org/docs/18/queries-with.html)
- [PostgreSQL: Combining queries](https://www.postgresql.org/docs/18/queries-union.html)

---

| [Previous: Aggregation and grouping](06-aggregation-and-grouping.md) | [Notes index](../README.md) | [Next: Inserts, updates, deletes, and transactions](08-inserts-updates-deletes-and-transactions.md) |
|:--|:--:|--:|
