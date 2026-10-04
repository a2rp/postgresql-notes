# 11. Views and generated data

[Back to notes index](../README.md)

| [Previous: Indexes and query plans](10-indexes-and-query-plans.md) | [Notes index](../README.md) | [Next: JSON and JSONB](12-json-and-jsonb.md) |
|:--|:--:|--:|

This chapter presents views, materialized views, and generated columns as ways to reuse or derive data.

## In this chapter

- Creating and replacing views
- Materialized views and refresh strategy
- Stored and virtual generated columns
- Choosing where derived values belong
- Eight review questions

## Reuse a query with a view

A view stores a query definition, not a snapshot of its rows. When the view is queried, PostgreSQL evaluates its defining query against the current tables.

~~~sql
CREATE OR REPLACE VIEW store.order_line_details AS
SELECT o.id AS order_id,
       c.full_name AS customer_name,
       p.sku,
       p.name AS product_name,
       oi.quantity,
       oi.unit_price,
       oi.quantity * oi.unit_price AS line_total
FROM store.orders AS o
JOIN store.customers AS c
  ON c.id = o.customer_id
JOIN store.order_items AS oi
  ON oi.order_id = o.id
JOIN store.products AS p
  ON p.id = oi.product_id;

SELECT *
FROM store.order_line_details
WHERE order_id = 42;
~~~

A view can simplify repeated read queries, but it does not automatically make a complicated query faster. Review its definition and permissions before exposing it to an application role.

## Store a result with a materialized view

A materialized view stores the result rows until it is refreshed. It is useful when a report is expensive and can tolerate data that is slightly behind the base tables.

~~~sql
CREATE MATERIALIZED VIEW store.monthly_order_totals AS
SELECT date_trunc('month', created_at) AS month_start,
       status,
       count(*) AS order_count
FROM store.orders
GROUP BY date_trunc('month', created_at), status;

CREATE UNIQUE INDEX monthly_order_totals_key
ON store.monthly_order_totals (month_start, status);

REFRESH MATERIALIZED VIEW store.monthly_order_totals;
~~~

REFRESH MATERIALIZED VIEW recomputes the stored result. REFRESH MATERIALIZED VIEW CONCURRENTLY allows readers to continue reading the old contents during refresh, but the view must already be populated and have a qualifying unique index that covers all rows.

~~~sql
REFRESH MATERIALIZED VIEW CONCURRENTLY store.monthly_order_totals;
~~~

Refresh scheduling is an application or operations decision. PostgreSQL does not refresh a materialized view automatically when its base tables change.

## Keep a derived value in the row

A generated column is computed from other columns in the same row. PostgreSQL 18 uses virtual generated columns by default; a stored generated column keeps the computed value on disk.

~~~sql
ALTER TABLE store.products
ADD COLUMN IF NOT EXISTS description text;

ALTER TABLE store.products
ADD COLUMN IF NOT EXISTS normalized_sku text
GENERATED ALWAYS AS (lower(sku)) STORED;
~~~

A generation expression must be immutable and may refer only to the current row. It cannot contain a subquery or read another table. Use a generated column for a deterministic row-level value, not a value that depends on today's date, another row, or external state.

Generated columns are not a replacement for a normal column when an application must choose and preserve a historical value. For example, an order item stores unit_price so a later product price change does not rewrite the price that was charged for an older order.

## Review questions

1. Does a normal view store a snapshot of its result rows?
2. When can a view make a repeated query easier to maintain?
3. What is the main difference between a materialized view and a normal view?
4. What does REFRESH MATERIALIZED VIEW do?
5. What conditions are needed for a concurrent materialized-view refresh?
6. When is a generated column evaluated?
7. What kinds of expressions can a generated column use?
8. Why should an order item keep the price charged at the time of purchase?

## References

- [PostgreSQL: CREATE VIEW](https://www.postgresql.org/docs/18/sql-createview.html)
- [PostgreSQL: Materialized views](https://www.postgresql.org/docs/18/rules-materializedviews.html)
- [PostgreSQL: Generated columns](https://www.postgresql.org/docs/18/ddl-generated-columns.html)

---

| [Previous: Indexes and query plans](10-indexes-and-query-plans.md) | [Notes index](../README.md) | [Next: JSON and JSONB](12-json-and-jsonb.md) |
