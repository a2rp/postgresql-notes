# 3. Selecting, filtering, and sorting

[Back to notes index](../README.md)

| [Previous: Databases, schemas, and tables](02-databases-schemas-and-tables.md) | [Notes index](../README.md) | [Next: Expressions, functions, and dates](04-expressions-functions-and-dates.md) |
|:--|:--:|--:|

This chapter covers the `SELECT` statement, conditions, `NULL`, sorting, and limiting result rows.

## In this chapter

- Selecting explicit columns
- `WHERE`, comparisons, and boolean logic
- `NULL` checks and three-valued logic
- `ORDER BY`, `LIMIT`, and stable pagination
- Eight review questions

## Select explicit columns

Use explicit column names so a query documents the data it needs. Aliases make output headings easier to read.

~~~sql
SELECT id,
       full_name AS customer_name,
       email
FROM store.customers;
~~~

SELECT * is handy while exploring a table, but application queries should usually list their columns. Adding a column to a table should not silently change every API response that selected all columns.

DISTINCT removes duplicate result rows. Use it when deduplication is part of the question, not as a patch for an unexpected join.

~~~sql
SELECT DISTINCT status
FROM store.orders
ORDER BY status;
~~~

## Filter rows with WHERE

WHERE is evaluated before grouping and sorting. Combine conditions with AND and OR, and use parentheses when the intended grouping is important.

~~~sql
SELECT id, sku, name, price
FROM store.products
WHERE active = true
  AND price >= 10.00
  AND price < 100.00
ORDER BY price, id;
~~~

BETWEEN includes both endpoints. IN checks membership in a list. LIKE matches text patterns, where percent matches any sequence and underscore matches one character. ILIKE performs case-insensitive matching under the active collation.

~~~sql
SELECT id, name
FROM store.products
WHERE price BETWEEN 10.00 AND 50.00
  AND sku IN ('MUG-01', 'BAG-02');

SELECT id, full_name
FROM store.customers
WHERE full_name ILIKE 'ash%';
~~~

For a user-entered substring search, escape wildcard characters if the user expects percent and underscore to be literal. For large text search needs, consider PostgreSQL full-text search instead of a leading-wildcard pattern.

## NULL is an unknown value

NULL is not zero, false, or an empty string. A comparison with NULL produces unknown, so this does not find missing emails:

~~~sql
SELECT id, full_name
FROM store.customers
WHERE email = NULL;
~~~

Use IS NULL or IS NOT NULL:

~~~sql
SELECT id, full_name
FROM store.customers
WHERE email IS NULL;
~~~

Use IS DISTINCT FROM when two values must be compared and NULL should behave as a comparable value:

~~~sql
SELECT NULL::text IS DISTINCT FROM ''::text AS values_are_different;
~~~

Because email is NOT NULL in our schema, the first query returns no rows. The example shows the correct syntax for nullable columns.

## Order and limit results

Rows have no guaranteed order unless ORDER BY is present. Add a unique tie-breaker when ordering by a non-unique value.

~~~sql
SELECT id, name, price
FROM store.products
ORDER BY price ASC, id ASC
LIMIT 20;
~~~

Use NULLS FIRST or NULLS LAST to state where missing values belong. PostgreSQL defaults differ between ascending and descending order, so explicit placement helps when a sort column is nullable.

~~~sql
SELECT id, full_name, created_at
FROM store.customers
ORDER BY created_at DESC NULLS LAST, id DESC
LIMIT 20;
~~~

OFFSET pagination skips an increasing number of rows and can shift when concurrent inserts happen. For large lists, keyset pagination uses the last row of the previous page as a cursor:

~~~sql
SELECT id, full_name, created_at
FROM store.customers
WHERE (created_at, id) < ($1, $2)
ORDER BY created_at DESC, id DESC
LIMIT 20;
~~~

The cursor values must come from the last row shown on the previous page. The ordering columns and comparison direction must match.

## Review questions

1. Why should an application query list the columns it needs?
2. What does DISTINCT do, and when can it hide a join problem?
3. What is the difference between WHERE and HAVING?
4. Why does email = NULL not find rows with a missing email?
5. What does BETWEEN include?
6. Why should ORDER BY include a unique tie-breaker for pagination?
7. How does ILIKE differ from LIKE in PostgreSQL?
8. When is keyset pagination preferable to a large OFFSET?

## References

- [PostgreSQL: SELECT](https://www.postgresql.org/docs/18/sql-select.html)
- [PostgreSQL: Comparison functions and operators](https://www.postgresql.org/docs/18/functions-comparison.html)
- [PostgreSQL: Pattern matching](https://www.postgresql.org/docs/18/functions-matching.html)

---

| [Previous: Databases, schemas, and tables](02-databases-and-schemas-and-tables.md) | [Notes index](../README.md) | [Next: Expressions, functions, and dates](04-expressions-functions-and-dates.md) |
|:--|:--:|--:|
