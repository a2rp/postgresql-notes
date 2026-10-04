# 2. Databases, schemas, and tables

[Back to notes index](../README.md)

| [Previous: PostgreSQL foundations](01-postgresql-foundations.md) | [Notes index](../README.md) | [Next: Selecting, filtering, and sorting](03-selecting-filtering-and-sorting.md) |
|:--|:--:|--:|

This chapter builds the shared store schema used by later query examples. A database contains schemas, and schemas contain tables and other objects.

## In this chapter

- Creating and selecting a database
- Schemas and `search_path`
- Tables, columns, and common data types
- Creating tables and inspecting their definition
- Eight review questions

## Database and schema

A database is a connection boundary. A schema is a namespace inside a database. The default schema is usually public, but a named application schema keeps related objects together and avoids relying on one crowded namespace.

Connect to study_store as in Chapter 1, then create and select the schema:

~~~sql
CREATE SCHEMA IF NOT EXISTS store;
SET search_path TO store, public;
SHOW search_path;
~~~

The search path affects how an unqualified object name is resolved. For predictable scripts, qualify important objects with the schema name, such as store.customers. Creating a schema does not grant every role permission to use it.

## Choose a suitable data type

- Use bigint or integer for whole-number identifiers and counts. Identity columns can generate numeric keys.
- Use numeric(10, 2) for money-like decimal values that must be exact. Do not use floating-point types for exact currency arithmetic.
- Use text for variable-length text. PostgreSQL does not gain a performance advantage from arbitrary varchar limits; use a length limit only when it is a real business rule.
- Use boolean for true or false values. A nullable boolean also has an unknown state, so use NOT NULL when that state is not meaningful.
- Use date for a calendar date and timestamptz for a real point in time. PostgreSQL converts timestamptz input to an instant and displays it in the session time zone.
- Use uuid when globally generated identifiers are useful across systems. Use jsonb only for flexible document-shaped data, not as a replacement for stable relational columns.

## Create the store tables

These definitions establish the columns needed for upcoming examples. Later chapters revisit constraints and relationships in more depth.

~~~sql
CREATE TABLE store.customers (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name text NOT NULL,
    email text NOT NULL UNIQUE,
    created_at timestamptz NOT NULL DEFAULT current_timestamp
);

CREATE TABLE store.products (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    sku text NOT NULL UNIQUE,
    name text NOT NULL,
    price numeric(10, 2) NOT NULL CHECK (price >= 0),
    active boolean NOT NULL DEFAULT true
);

CREATE TABLE store.orders (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id bigint NOT NULL REFERENCES store.customers (id),
    status text NOT NULL DEFAULT 'pending',
    created_at timestamptz NOT NULL DEFAULT current_timestamp
);

CREATE TABLE store.order_items (
    order_id bigint NOT NULL REFERENCES store.orders (id),
    product_id bigint NOT NULL REFERENCES store.products (id),
    quantity integer NOT NULL CHECK (quantity > 0),
    unit_price numeric(10, 2) NOT NULL CHECK (unit_price >= 0),
    PRIMARY KEY (order_id, product_id)
);
~~~

The identity definition creates values for new rows when an insert omits id. The primary key both identifies each row and prevents duplicate or null key values. The order_items primary key says a product can appear only once per order; quantity represents multiple units.

## Inspect the objects

SQL catalog views can be queried from any PostgreSQL client:

~~~sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_schema = 'store'
ORDER BY table_name;

SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_schema = 'store'
  AND table_name = 'products'
ORDER BY ordinal_position;
~~~

In psql, these shortcuts are convenient during exploration:

~~~text
\dn
\dt store.*
\d store.products
~~~

## Naming and schema changes

Use lowercase snake_case names without double quotes. PostgreSQL folds unquoted identifiers to lowercase, while a quoted mixed-case name must be quoted every time it is referenced.

Use ALTER TABLE for deliberate schema changes. For example, add a description only if the application has that concept:

~~~sql
ALTER TABLE store.products
ADD COLUMN description text;
~~~

For a required column in a populated table, plan a safe migration: add it as nullable or with a suitable default, populate existing rows, then apply NOT NULL after checking the data.

## Practice check

Create the four tables, list them with \dt store.*, then inspect store.order_items. Confirm that both identifier columns have the same bigint type as the referenced primary keys.

## Review questions

1. How is a schema different from a database?
2. Why is a named application schema useful?
3. Which PostgreSQL type stores an exact decimal amount?
4. What is the difference between date and timestamptz?
5. What does GENERATED ALWAYS AS IDENTITY provide?
6. Why should an unquoted identifier usually use lowercase snake_case?
7. How can you inspect a table's columns with SQL?
8. Why might a required column need a staged migration on a populated table?

## References

- [PostgreSQL: Schemas](https://www.postgresql.org/docs/18/ddl-schemas.html)
- [PostgreSQL: Data types](https://www.postgresql.org/docs/18/datatype.html)
- [PostgreSQL: CREATE TABLE](https://www.postgresql.org/docs/18/sql-createtable.html)

---

| [Previous: PostgreSQL foundations](01-postgresql-foundations.md) | [Notes index](../README.md) | [Next: Selecting, filtering, and sorting](03-selecting-filtering-and-sorting.md) |
|:--|:--:|--:|
