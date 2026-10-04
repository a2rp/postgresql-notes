# 10. Indexes and query plans

[Back to notes index](../README.md)

| [Previous: Keys, constraints, and data integrity](09-keys-constraints-and-data-integrity.md) | [Notes index](../README.md) | [Next: Views and generated data](11-views-and-generated-data.md) |
|:--|:--:|--:|

This chapter explains how indexes can reduce work and how to read a query plan before changing a schema.

## In this chapter

- B-tree and other common index types
- Choosing useful index columns
- `EXPLAIN` and `EXPLAIN ANALYZE`
- Costs, statistics, and common performance mistakes
- Eight review questions

## An index is a trade-off

An index is a separate data structure that helps PostgreSQL find or order rows without scanning the whole table. It takes disk space and adds work to INSERT, UPDATE, and DELETE. Add an index for a measured query pattern, not for every column.

B-tree is the default index type and supports equality, ranges, and ordered retrieval. This index supports looking up a customer's newest orders:

~~~sql
CREATE INDEX orders_customer_created_idx
ON store.orders (customer_id, created_at DESC, id DESC);

SELECT id, status, created_at
FROM store.orders
WHERE customer_id = 42
ORDER BY created_at DESC, id DESC
LIMIT 20;
~~~

In a multicolumn B-tree, leading columns matter. An index beginning with customer_id is useful for a customer filter, but is not generally a substitute for an index that begins with status.

## Match index shape to the query

A partial index stores only rows matching its predicate. It can help when a query repeatedly asks for a small, stable subset:

~~~sql
CREATE INDEX products_active_price_idx
ON store.products (price, id)
WHERE active;

SELECT id, name, price
FROM store.products
WHERE active
  AND price < 50
ORDER BY price, id;
~~~

The query predicate must imply the partial-index predicate for the planner to use it. A partial index does not replace a general index for queries that need inactive rows too.

An expression index can support a search on the same expression:

~~~sql
CREATE INDEX customers_lower_email_idx
ON store.customers (lower(email));

SELECT id, full_name
FROM store.customers
WHERE lower(email) = lower('MIRA@example.test');
~~~

Before adding a unique expression index, check for existing duplicates under that expression. A normal unique constraint on email is case-sensitive and does not provide case-insensitive uniqueness.

GIN is commonly used for membership searches in JSONB and full-text search. BRIN can be compact for very large tables where values follow the physical row order, such as append-heavy time data. Choose an index type based on the operators and data shape.

## Read a plan

EXPLAIN shows the plan PostgreSQL estimates it will use:

~~~sql
EXPLAIN
SELECT id, status, created_at
FROM store.orders
WHERE customer_id = 42
ORDER BY created_at DESC, id DESC
LIMIT 20;
~~~

EXPLAIN ANALYZE runs the statement and reports actual row counts and timing alongside estimates. Use it for SELECT queries when measuring real execution:

~~~sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT id, status, created_at
FROM store.orders
WHERE customer_id = 42
ORDER BY created_at DESC, id DESC
LIMIT 20;
~~~

EXPLAIN ANALYZE executes the query. It can change data if the statement is INSERT, UPDATE, or DELETE. For a DML check, wrap the change in a transaction and roll it back after reviewing the plan.

Read the plan from the leaves upward. Compare estimated rows with actual rows, notice repeated loops, and check whether sorting or reading many heap pages dominates. A sequential scan can be the right plan for a small table or a query that returns a large fraction of rows.

Planner estimates depend on statistics. ANALYZE gathers column statistics; autovacuum normally runs analyze as needed. After a major data change, run ANALYZE on the affected table and compare plans again.

~~~sql
ANALYZE store.orders;
~~~

Do not compare only the cost number between unrelated databases. Cost is an estimate in planner units, not elapsed milliseconds.

## Review questions

1. What does an index cost in addition to storage?
2. What kinds of lookups does a B-tree support?
3. Why does the leading column matter in a multicolumn B-tree?
4. What is a partial index useful for?
5. What does EXPLAIN ANALYZE do that EXPLAIN alone does not?
6. Why can a sequential scan be the right plan?
7. What should you inspect when estimated and actual row counts differ greatly?
8. What does ANALYZE collect for the planner?

## References

- [PostgreSQL: Indexes](https://www.postgresql.org/docs/18/indexes.html)
- [PostgreSQL: EXPLAIN](https://www.postgresql.org/docs/18/sql-explain.html)
- [PostgreSQL: Planner statistics](https://www.postgresql.org/docs/18/planner-stats.html)

---

| [Previous: Keys, constraints, and data integrity](09-keys-constraints-and-data-integrity.md) | [Notes index](../README.md) | [Next: Views and generated data](11-views-and-generated-data.md) |
|:--|:--:|--:|
