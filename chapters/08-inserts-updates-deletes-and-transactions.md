# 8. Inserts, updates, deletes, and transactions

[Back to notes index](../README.md)

| [Previous: Subqueries, CTEs, and set operations](07-subqueries-ctes-and-set-operations.md) | [Notes index](../README.md) | [Next: Keys, constraints, and data integrity](09-keys-constraints-and-data-integrity.md) |
|:--|:--:|--:|

This chapter changes data safely and explains how transactions keep related changes together.

## In this chapter

- `INSERT`, `UPDATE`, and `DELETE`
- `RETURNING` and `ON CONFLICT`
- `BEGIN`, `COMMIT`, and `ROLLBACK`
- Isolation, locks, and safe concurrent changes
- Eight review questions

## Insert rows and read generated values

Name the columns in every insert. This makes the statement resilient to column order changes and makes defaults visible.

~~~sql
INSERT INTO store.customers (full_name, email)
VALUES ('Mira Patel', 'mira@example.test')
RETURNING id, full_name, created_at;
~~~

The generated id and default timestamp are returned by the same statement. For multiple rows, RETURNING returns one result row for each inserted row:

~~~sql
INSERT INTO store.products (sku, name, price)
VALUES
    ('MUG-01', 'Travel mug', 19.50),
    ('BAG-02', 'Canvas bag', 34.00)
RETURNING id, sku;
~~~

## Update and delete with a clear condition

Check the WHERE clause before running a change. Without it, UPDATE changes every row and DELETE removes every row.

~~~sql
UPDATE store.products
SET price = 21.00
WHERE sku = 'MUG-01'
RETURNING id, sku, price;
~~~

The returned rows show what changed. A client should also check the affected-row count when it expects exactly one row.

~~~sql
DELETE FROM store.products
WHERE sku = 'OLD-ITEM'
RETURNING id, sku;
~~~

The product may not be deletable if an order item references it. That is a useful integrity safeguard. Consider deactivating a product instead of deleting historical data.

## Handle a known uniqueness conflict

ON CONFLICT handles a conflict with a unique or exclusion constraint. State the intended policy instead of catching every database error.

~~~sql
INSERT INTO store.customers (full_name, email)
VALUES ('Mira Patel', 'mira@example.test')
ON CONFLICT (email) DO NOTHING
RETURNING id, email;
~~~

DO NOTHING returns no row when the email already exists. An upsert can update the matching row, but only do so when replacing those values is the intended business rule:

~~~sql
INSERT INTO store.products (sku, name, price)
VALUES ('MUG-01', 'Travel mug', 19.50)
ON CONFLICT (sku) DO UPDATE
SET name = EXCLUDED.name,
    price = EXCLUDED.price
RETURNING id, sku, name, price;
~~~

EXCLUDED refers to the row proposed for insertion. The existing table row remains the target of the update.

## Keep related changes in one transaction

A transaction groups statements into one unit. If a later statement fails, roll back earlier statements so the database does not keep a partial order.

~~~sql
BEGIN;

INSERT INTO store.orders (customer_id, status)
VALUES (1, 'pending')
RETURNING id;

-- Use the returned order id in the next statement.
INSERT INTO store.order_items (order_id, product_id, quantity, unit_price)
VALUES (42, 1, 2, 19.50);

COMMIT;
~~~

Here 42 is a placeholder for the id returned by the first statement. In an application, keep both statements on the same database connection and pass the returned id as a parameter. If an insert fails, issue ROLLBACK.

~~~sql
BEGIN;
UPDATE store.orders
SET status = 'paid'
WHERE id = 42
  AND status = 'pending'
RETURNING id, status;
COMMIT;
~~~

The status predicate makes the transition conditional. Check that exactly one row was returned before treating the order as paid. A transaction does not make a multi-step business rule correct by itself; it makes the chosen statements atomic.

Savepoints let a transaction roll back part of its work while keeping earlier statements:

~~~sql
BEGIN;
SAVEPOINT before_optional_change;
-- Perform an optional change here.
ROLLBACK TO SAVEPOINT before_optional_change;
COMMIT;
~~~

## Concurrency basics

PostgreSQL uses multiversion concurrency control so readers and writers usually do not block each other. Writes still take locks, and conflicting writes may wait. Keep transactions short and acquire locks in a consistent order to reduce contention and deadlocks.

SELECT FOR UPDATE locks selected rows until the transaction ends. Use it when a workflow must read a row and then make a decision based on that exact row state. For simple state changes, a conditional UPDATE with RETURNING is often enough.

## Review questions

1. Why should an INSERT list its target columns?
2. What does RETURNING provide?
3. What can happen if an UPDATE statement has no WHERE clause?
4. When is ON CONFLICT DO NOTHING appropriate?
5. What does EXCLUDED refer to in an upsert?
6. What should an application do if one statement in a transaction fails?
7. Why must the same connection be used for every statement in a transaction?
8. When is SELECT FOR UPDATE useful?

## References

- [PostgreSQL: INSERT](https://www.postgresql.org/docs/18/sql-insert.html)
- [PostgreSQL: UPDATE](https://www.postgresql.org/docs/18/sql-update.html)
- [PostgreSQL: DELETE](https://www.postgresql.org/docs/18/sql-delete.html)
- [PostgreSQL: Transactions](https://www.postgresql.org/docs/18/tutorial-transactions.html)
- [PostgreSQL: Concurrency control](https://www.postgresql.org/docs/18/mvcc.html)

---

| [Previous: Subqueries, CTEs, and set operations](07-subqueries-ctes-and-set-operations.md) | [Notes index](../README.md) | [Next: Keys, constraints, and data integrity](09-keys-constraints-and-data-integrity.md) |
