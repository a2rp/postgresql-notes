# All code samples

[Back to notes index](../README.md)

| [Previous: Capstone: order management database](16-capstone-order-management-database.md) | [Notes index](../README.md) | [Next: Complete Q&A](99-complete-q-and-a.md) |
|:--|:--:|--:|

This appendix gathers the fenced SQL, shell, psql, and JavaScript examples from the 16 core chapters. The examples stay grouped by the chapter where they are explained.

## Chapter 1: 1. PostgreSQL foundations

[Open the chapter](01-postgresql-foundations.md)

### Sample 1

~~~sh
psql --host localhost --port 5432 --username app_user --dbname postgres
~~~

### Sample 2

~~~text
\conninfo
\l
\du
\q
~~~

### Sample 3

~~~sql
SELECT version();
SELECT current_database(), current_user;
SHOW server_version;
~~~

### Sample 4

~~~sh
createdb --host localhost --port 5432 --username app_user study_store
psql --host localhost --port 5432 --username app_user --dbname study_store
~~~

### Sample 5

~~~sql
CREATE DATABASE study_store;
~~~

### Sample 6

~~~sql
SELECT 'PostgreSQL is ready' AS status,
       current_timestamp AS checked_at;
~~~

### Sample 7

~~~text
\x
\timing
~~~

## Chapter 2: 2. Databases, schemas, and tables

[Open the chapter](02-databases-schemas-and-tables.md)

### Sample 1

~~~sql
CREATE SCHEMA IF NOT EXISTS store;
SET search_path TO store, public;
SHOW search_path;
~~~

### Sample 2

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

### Sample 3

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

### Sample 4

~~~text
\dn
\dt store.*
\d store.products
~~~

### Sample 5

~~~sql
ALTER TABLE store.products
ADD COLUMN description text;
~~~

## Chapter 3: 3. Selecting, filtering, and sorting

[Open the chapter](03-selecting-filtering-and-sorting.md)

### Sample 1

~~~sql
SELECT id,
       full_name AS customer_name,
       email
FROM store.customers;
~~~

### Sample 2

~~~sql
SELECT DISTINCT status
FROM store.orders
ORDER BY status;
~~~

### Sample 3

~~~sql
SELECT id, sku, name, price
FROM store.products
WHERE active = true
  AND price >= 10.00
  AND price < 100.00
ORDER BY price, id;
~~~

### Sample 4

~~~sql
SELECT id, name
FROM store.products
WHERE price BETWEEN 10.00 AND 50.00
  AND sku IN ('MUG-01', 'BAG-02');

SELECT id, full_name
FROM store.customers
WHERE full_name ILIKE 'ash%';
~~~

### Sample 5

~~~sql
SELECT id, full_name
FROM store.customers
WHERE email = NULL;
~~~

### Sample 6

~~~sql
SELECT id, full_name
FROM store.customers
WHERE email IS NULL;
~~~

### Sample 7

~~~sql
SELECT NULL::text IS DISTINCT FROM ''::text AS values_are_different;
~~~

### Sample 8

~~~sql
SELECT id, name, price
FROM store.products
ORDER BY price ASC, id ASC
LIMIT 20;
~~~

### Sample 9

~~~sql
SELECT id, full_name, created_at
FROM store.customers
ORDER BY created_at DESC NULLS LAST, id DESC
LIMIT 20;
~~~

### Sample 10

~~~sql
SELECT id, full_name, created_at
FROM store.customers
WHERE (created_at, id) < ($1, $2)
ORDER BY created_at DESC, id DESC
LIMIT 20;
~~~

## Chapter 4: 4. Expressions, functions, and dates

[Open the chapter](04-expressions-functions-and-dates.md)

### Sample 1

~~~sql
SELECT id,
       price,
       price * 1.10 AS price_with_tax
FROM store.products
WHERE active;
~~~

### Sample 2

~~~sql
SELECT id,
       name,
       CASE
           WHEN price >= 100 THEN 'premium'
           WHEN price >= 25 THEN 'standard'
           ELSE 'budget'
       END AS price_band
FROM store.products;
~~~

### Sample 3

~~~sql
SELECT COALESCE(NULL::text, 'not provided') AS display_name;
SELECT NULLIF('unknown', 'unknown') AS normalized_value;
~~~

### Sample 4

~~~sql
SELECT lower(trim('  PostgreSQL  ')) AS normalized_text,
       char_length('database') AS character_count,
       round(19.995::numeric, 2) AS rounded_amount;
~~~

### Sample 5

~~~sql
SELECT full_name || ' <' || email || '>' AS contact
FROM store.customers;
~~~

### Sample 6

~~~sql
SELECT CURRENT_DATE AS today,
       CURRENT_TIMESTAMP AS now_with_time_zone,
       current_setting('TimeZone') AS session_time_zone;
~~~

### Sample 7

~~~sql
SELECT CURRENT_DATE + 7 AS one_week_from_today,
       CURRENT_TIMESTAMP + interval '90 minutes' AS later_today;
~~~

### Sample 8

~~~sql
SELECT id, customer_id, created_at
FROM store.orders
WHERE created_at >= TIMESTAMPTZ '2026-10-01 00:00:00+00'
  AND created_at <  TIMESTAMPTZ '2026-10-02 00:00:00+00';
~~~

### Sample 9

~~~sql
SELECT date_trunc('month', created_at AT TIME ZONE 'Asia/Kolkata') AS local_month,
       count(*) AS order_count
FROM store.orders
GROUP BY local_month
ORDER BY local_month;
~~~

## Chapter 5: 5. Joins and relationships

[Open the chapter](05-joins-and-relations.md)

### Sample 1

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

### Sample 2

~~~sql
SELECT c.id,
       c.full_name,
       o.id AS order_id
FROM store.customers AS c
LEFT JOIN store.orders AS o
  ON o.customer_id = c.id
ORDER BY c.id, o.id;
~~~

### Sample 3

~~~sql
SELECT c.id, c.full_name
FROM store.customers AS c
LEFT JOIN store.orders AS o
  ON o.customer_id = c.id
WHERE o.id IS NULL
ORDER BY c.id;
~~~

### Sample 4

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

### Sample 5

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

### Sample 6

~~~sql
SELECT c.id,
       count(o.id) AS number_of_orders
FROM store.customers AS c
LEFT JOIN store.orders AS o
  ON o.customer_id = c.id
GROUP BY c.id
ORDER BY c.id;
~~~

## Chapter 6: 6. Aggregation and grouping

[Open the chapter](06-aggregation-and-grouping.md)

### Sample 1

~~~sql
SELECT count(*) AS product_rows,
       count(description) AS products_with_description,
       count(DISTINCT active) AS distinct_active_values
FROM store.products;
~~~

### Sample 2

~~~sql
SELECT coalesce(sum(price), 0::numeric) AS total_catalog_value,
       avg(price) AS average_price,
       min(price) AS lowest_price,
       max(price) AS highest_price
FROM store.products
WHERE active;
~~~

### Sample 3

~~~sql
SELECT status,
       count(*) AS order_count
FROM store.orders
GROUP BY status
ORDER BY order_count DESC, status;
~~~

### Sample 4

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

### Sample 5

~~~sql
SELECT customer_id,
       count(*) AS order_count
FROM store.orders
WHERE created_at >= current_date - 30
GROUP BY customer_id
HAVING count(*) >= 2
ORDER BY order_count DESC, customer_id;
~~~

### Sample 6

~~~sql
SELECT customer_id,
       count(*) AS all_orders,
       count(*) FILTER (WHERE status = 'paid') AS paid_orders,
       count(*) FILTER (WHERE status = 'cancelled') AS cancelled_orders
FROM store.orders
GROUP BY customer_id
ORDER BY customer_id;
~~~

## Chapter 7: 7. Subqueries, CTEs, and set operations

[Open the chapter](07-subqueries-ctes-and-set-operations.md)

### Sample 1

~~~sql
SELECT id, name, price
FROM store.products
WHERE price > (
    SELECT avg(price)
    FROM store.products
)
ORDER BY price DESC, id;
~~~

### Sample 2

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

### Sample 3

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

### Sample 4

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

### Sample 5

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

### Sample 6

~~~sql
SELECT email AS contact
FROM store.customers
UNION
SELECT 'support@example.test'::text AS contact
ORDER BY contact;
~~~

## Chapter 8: 8. Inserts, updates, deletes, and transactions

[Open the chapter](08-inserts-updates-deletes-and-transactions.md)

### Sample 1

~~~sql
INSERT INTO store.customers (full_name, email)
VALUES ('Mira Patel', 'mira@example.test')
RETURNING id, full_name, created_at;
~~~

### Sample 2

~~~sql
INSERT INTO store.products (sku, name, price)
VALUES
    ('MUG-01', 'Travel mug', 19.50),
    ('BAG-02', 'Canvas bag', 34.00)
RETURNING id, sku;
~~~

### Sample 3

~~~sql
UPDATE store.products
SET price = 21.00
WHERE sku = 'MUG-01'
RETURNING id, sku, price;
~~~

### Sample 4

~~~sql
DELETE FROM store.products
WHERE sku = 'OLD-ITEM'
RETURNING id, sku;
~~~

### Sample 5

~~~sql
INSERT INTO store.customers (full_name, email)
VALUES ('Mira Patel', 'mira@example.test')
ON CONFLICT (email) DO NOTHING
RETURNING id, email;
~~~

### Sample 6

~~~sql
INSERT INTO store.products (sku, name, price)
VALUES ('MUG-01', 'Travel mug', 19.50)
ON CONFLICT (sku) DO UPDATE
SET name = EXCLUDED.name,
    price = EXCLUDED.price
RETURNING id, sku, name, price;
~~~

### Sample 7

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

### Sample 8

~~~sql
BEGIN;
UPDATE store.orders
SET status = 'paid'
WHERE id = 42
  AND status = 'pending'
RETURNING id, status;
COMMIT;
~~~

### Sample 9

~~~sql
BEGIN;
SAVEPOINT before_optional_change;
-- Perform an optional change here.
ROLLBACK TO SAVEPOINT before_optional_change;
COMMIT;
~~~

## Chapter 9: 9. Keys, constraints, and data integrity

[Open the chapter](09-keys-constraints-and-data-integrity.md)

### Sample 1

~~~sql
CREATE TABLE store.coupon_codes (
    id bigint GENERATED ALWAYS AS IDENTITY,
    code text NOT NULL,
    discount_percent numeric(5, 2) NOT NULL,
    active boolean NOT NULL DEFAULT true,
    CONSTRAINT coupon_codes_pkey PRIMARY KEY (id),
    CONSTRAINT coupon_codes_code_key UNIQUE (code),
    CONSTRAINT coupon_codes_discount_check
        CHECK (discount_percent > 0 AND discount_percent <= 100)
);
~~~

### Sample 2

~~~sql
ALTER TABLE store.orders
ADD CONSTRAINT orders_customer_fkey
FOREIGN KEY (customer_id)
REFERENCES store.customers (id)
ON DELETE RESTRICT;
~~~

### Sample 3

~~~sql
CREATE TABLE store.product_reviews (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    product_id bigint NOT NULL REFERENCES store.products (id),
    rating integer NOT NULL CHECK (rating BETWEEN 1 AND 5),
    body text,
    created_at timestamptz NOT NULL DEFAULT current_timestamp
);
~~~

### Sample 4

~~~sql
CREATE INDEX orders_customer_id_idx
ON store.orders (customer_id);

CREATE INDEX order_items_product_id_idx
ON store.order_items (product_id);
~~~

### Sample 5

~~~sql
ALTER TABLE store.orders
ADD CONSTRAINT orders_status_check
CHECK (status IN ('pending', 'paid', 'cancelled')) NOT VALID;

ALTER TABLE store.orders
VALIDATE CONSTRAINT orders_status_check;
~~~

## Chapter 10: 10. Indexes and query plans

[Open the chapter](10-indexes-and-query-plans.md)

### Sample 1

~~~sql
CREATE INDEX orders_customer_created_idx
ON store.orders (customer_id, created_at DESC, id DESC);

SELECT id, status, created_at
FROM store.orders
WHERE customer_id = 42
ORDER BY created_at DESC, id DESC
LIMIT 20;
~~~

### Sample 2

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

### Sample 3

~~~sql
CREATE INDEX customers_lower_email_idx
ON store.customers (lower(email));

SELECT id, full_name
FROM store.customers
WHERE lower(email) = lower('MIRA@example.test');
~~~

### Sample 4

~~~sql
EXPLAIN
SELECT id, status, created_at
FROM store.orders
WHERE customer_id = 42
ORDER BY created_at DESC, id DESC
LIMIT 20;
~~~

### Sample 5

~~~sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT id, status, created_at
FROM store.orders
WHERE customer_id = 42
ORDER BY created_at DESC, id DESC
LIMIT 20;
~~~

### Sample 6

~~~sql
ANALYZE store.orders;
~~~

## Chapter 11: 11. Views and generated data

[Open the chapter](11-views-and-generated-data.md)

### Sample 1

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

### Sample 2

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

### Sample 3

~~~sql
REFRESH MATERIALIZED VIEW CONCURRENTLY store.monthly_order_totals;
~~~

### Sample 4

~~~sql
ALTER TABLE store.products
ADD COLUMN IF NOT EXISTS description text;

ALTER TABLE store.products
ADD COLUMN IF NOT EXISTS normalized_sku text
GENERATED ALWAYS AS (lower(sku)) STORED;
~~~

## Chapter 12: 12. JSON and JSONB

[Open the chapter](12-json-and-jsonb.md)

### Sample 1

~~~sql
ALTER TABLE store.products
ADD COLUMN IF NOT EXISTS attributes jsonb NOT NULL DEFAULT '{}'::jsonb;
~~~

### Sample 2

~~~sql
UPDATE store.products
SET attributes = '{"color":"black","shipping":{"weight_grams":240},"tags":["gift","travel"]}'::jsonb
WHERE sku = 'MUG-01';
~~~

### Sample 3

~~~sql
SELECT sku,
       attributes -> 'tags' AS tags_json,
       attributes ->> 'color' AS color_text,
       attributes #>> '{shipping,weight_grams}' AS nested_weight_text
FROM store.products
WHERE sku = 'MUG-01';
~~~

### Sample 4

~~~sql
SELECT sku
FROM store.products
WHERE attributes ? 'color';
~~~

### Sample 5

~~~sql
SELECT sku, name
FROM store.products
WHERE attributes @> '{"tags":["travel"]}'::jsonb;
~~~

### Sample 6

~~~sql
CREATE INDEX products_attributes_gin_idx
ON store.products
USING gin (attributes);
~~~

### Sample 7

~~~sql
UPDATE store.products
SET attributes = jsonb_set(
    attributes,
    ARRAY['color'],
    to_jsonb('graphite'::text),
    true
)
WHERE sku = 'MUG-01'
RETURNING sku, attributes;
~~~

## Chapter 13: 13. PL/pgSQL functions and triggers

[Open the chapter](13-plpgsql-functions-and-triggers.md)

### Sample 1

~~~sql
CREATE OR REPLACE FUNCTION store.order_item_total(p_order_id bigint)
RETURNS numeric
LANGUAGE sql
STABLE
AS $function$
    SELECT coalesce(sum(quantity * unit_price), 0::numeric)
    FROM store.order_items
    WHERE order_id = p_order_id
$function$;

SELECT store.order_item_total(42);
~~~

### Sample 2

~~~sql
CREATE OR REPLACE FUNCTION store.mark_order_paid(p_order_id bigint)
RETURNS boolean
LANGUAGE plpgsql
AS $function$
DECLARE
    changed_id bigint;
BEGIN
    UPDATE store.orders
    SET status = 'paid'
    WHERE id = p_order_id
      AND status = 'pending'
    RETURNING id INTO changed_id;

    RETURN changed_id IS NOT NULL;
END;
$function$;
~~~

### Sample 3

~~~sql
ALTER TABLE store.products
ADD COLUMN IF NOT EXISTS updated_at timestamptz NOT NULL DEFAULT current_timestamp;

CREATE OR REPLACE FUNCTION store.set_product_updated_at()
RETURNS trigger
LANGUAGE plpgsql
AS $function$
BEGIN
    NEW.updated_at := clock_timestamp();
    RETURN NEW;
END;
$function$;
~~~

### Sample 4

~~~sql
DROP TRIGGER IF EXISTS products_set_updated_at ON store.products;

CREATE TRIGGER products_set_updated_at
BEFORE UPDATE ON store.products
FOR EACH ROW
EXECUTE FUNCTION store.set_product_updated_at();
~~~

## Chapter 14: 14. Roles, privileges, and maintenance

[Open the chapter](14-roles-privileges-and-maintenance.md)

### Sample 1

~~~sql
CREATE ROLE store_reader NOLOGIN;
CREATE ROLE store_runtime LOGIN;

GRANT CONNECT ON DATABASE study_store TO store_reader;
GRANT USAGE ON SCHEMA store TO store_reader;
GRANT SELECT ON ALL TABLES IN SCHEMA store TO store_reader;

GRANT CONNECT ON DATABASE study_store TO store_runtime;
GRANT USAGE ON SCHEMA store TO store_runtime;
GRANT SELECT, INSERT, UPDATE ON
    store.customers,
    store.orders,
    store.order_items,
    store.products
TO store_runtime;
~~~

### Sample 2

~~~sql
GRANT store_reader TO store_runtime;
~~~

### Sample 3

~~~sql
ALTER DEFAULT PRIVILEGES IN SCHEMA store
GRANT SELECT ON TABLES TO store_reader;
~~~

### Sample 4

~~~sh
pg_dump --format=custom --dbname=study_store --file=study_store.dump
createdb study_store_restore
pg_restore --dbname=study_store_restore study_store.dump
~~~

### Sample 5

~~~sh
pg_dumpall --globals-only --file=cluster-roles.sql
~~~

### Sample 6

~~~sql
VACUUM (ANALYZE) store.orders;

SELECT pid, usename, state, wait_event_type, wait_event
FROM pg_stat_activity
WHERE datname = current_database()
ORDER BY pid;
~~~

## Chapter 15: 15. Node.js and PostgreSQL

[Open the chapter](15-nodejs-and-postgresql.md)

### Sample 1

~~~sh
npm init -y
npm install pg
~~~

### Sample 2

~~~js
const { Pool } = require('pg');

const pool = new Pool({
  max: 10,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
});

async function checkDatabase() {
  const result = await pool.query(
    'SELECT current_database() AS database, current_user AS role'
  );
  return result.rows[0];
}

checkDatabase()
  .then((row) => console.log(row))
  .catch((error) => {
    console.error('Database check failed:', error.message);
    process.exitCode = 1;
  })
  .finally(() => pool.end());
~~~

### Sample 3

~~~js
async function findCustomerByEmail(email) {
  const result = await pool.query(
    'SELECT id, full_name, email FROM store.customers WHERE email = $1',
    [email]
  );

  return result.rows[0] ?? null;
}
~~~

### Sample 4

~~~js
// Unsafe: input can change the SQL statement.
const unsafeSql = "SELECT id FROM store.customers WHERE email = '" + email + "'";
~~~

### Sample 5

~~~js
const sortColumns = {
  name: 'name',
  price: 'price'
};

function productSortSql(requestedSort) {
  const column = sortColumns[requestedSort] ?? sortColumns.name;
  return 'SELECT id, name, price FROM store.products ORDER BY ' + column + ', id';
}
~~~

### Sample 6

~~~js
async function createOrder({ customerId, productId, quantity }) {
  const client = await pool.connect();

  try {
    await client.query('BEGIN');

    const productResult = await client.query(
      'SELECT price ' +
      'FROM store.products ' +
      'WHERE id = $1 AND active ' +
      'FOR SHARE',
      [productId]
    );

    if (productResult.rowCount !== 1) {
      throw new Error('Product is missing or inactive');
    }

    const orderResult = await client.query(
      'INSERT INTO store.orders (customer_id, status) ' +
      "VALUES ($1, 'pending') " +
      'RETURNING id',
      [customerId]
    );

    const orderId = orderResult.rows[0].id;
    const unitPrice = productResult.rows[0].price;

    await client.query(
      'INSERT INTO store.order_items ' +
      '(order_id, product_id, quantity, unit_price) ' +
      'VALUES ($1, $2, $3, $4)',
      [orderId, productId, quantity, unitPrice]
    );

    await client.query('COMMIT');
    return orderId;
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
~~~

## Chapter 16: 16. Capstone: order management database

[Open the chapter](16-capstone-order-management-database.md)

### Sample 1

~~~sql
CREATE SCHEMA capstone;

CREATE TABLE capstone.customers (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    full_name text NOT NULL,
    email text NOT NULL UNIQUE,
    created_at timestamptz NOT NULL DEFAULT current_timestamp
);

CREATE TABLE capstone.products (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    sku text NOT NULL UNIQUE,
    name text NOT NULL,
    price numeric(10, 2) NOT NULL CHECK (price >= 0),
    active boolean NOT NULL DEFAULT true
);

CREATE TABLE capstone.orders (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id bigint NOT NULL
        REFERENCES capstone.customers (id) ON DELETE RESTRICT,
    status text NOT NULL DEFAULT 'pending'
        CHECK (status IN ('pending', 'paid', 'cancelled')),
    created_at timestamptz NOT NULL DEFAULT current_timestamp
);

CREATE TABLE capstone.order_items (
    order_id bigint NOT NULL
        REFERENCES capstone.orders (id) ON DELETE CASCADE,
    product_id bigint NOT NULL
        REFERENCES capstone.products (id) ON DELETE RESTRICT,
    quantity integer NOT NULL CHECK (quantity > 0),
    unit_price numeric(10, 2) NOT NULL CHECK (unit_price >= 0),
    PRIMARY KEY (order_id, product_id)
);

CREATE INDEX capstone_orders_customer_created_idx
ON capstone.orders (customer_id, created_at DESC, id DESC);

CREATE INDEX capstone_order_items_product_idx
ON capstone.order_items (product_id);
~~~

### Sample 2

~~~sql
WITH new_customer AS (
    INSERT INTO capstone.customers (full_name, email)
    VALUES ('Mira Patel', 'mira@example.test')
    RETURNING id
),
new_product AS (
    INSERT INTO capstone.products (sku, name, price)
    VALUES ('MUG-01', 'Travel mug', 19.50)
    RETURNING id, price
),
new_order AS (
    INSERT INTO capstone.orders (customer_id, status)
    SELECT id, 'pending'
    FROM new_customer
    RETURNING id
)
INSERT INTO capstone.order_items
    (order_id, product_id, quantity, unit_price)
SELECT new_order.id, new_product.id, 2, new_product.price
FROM new_order
CROSS JOIN new_product
RETURNING order_id, product_id, quantity, unit_price;
~~~

### Sample 3

~~~sql
SELECT o.id AS order_id,
       o.status,
       o.created_at,
       c.full_name AS customer,
       c.email,
       p.sku,
       p.name AS product,
       oi.quantity,
       oi.unit_price,
       oi.quantity * oi.unit_price AS line_total
FROM capstone.orders AS o
JOIN capstone.customers AS c
  ON c.id = o.customer_id
JOIN capstone.order_items AS oi
  ON oi.order_id = o.id
JOIN capstone.products AS p
  ON p.id = oi.product_id
ORDER BY o.created_at DESC, o.id DESC, p.id;
~~~

### Sample 4

~~~sql
SELECT c.id,
       c.full_name,
       count(DISTINCT o.id) AS order_count,
       coalesce(sum(oi.quantity * oi.unit_price), 0::numeric) AS amount_ordered
FROM capstone.customers AS c
LEFT JOIN capstone.orders AS o
  ON o.customer_id = c.id
LEFT JOIN capstone.order_items AS oi
  ON oi.order_id = o.id
GROUP BY c.id, c.full_name
ORDER BY amount_ordered DESC, c.id;
~~~

### Sample 5

~~~js
async function createCapstoneOrder({ customerId, productId, quantity }) {
  if (!Number.isInteger(quantity) || quantity < 1) {
    throw new TypeError('quantity must be a positive integer');
  }

  const client = await pool.connect();

  try {
    await client.query('BEGIN');

    const product = await client.query(
      'SELECT price FROM capstone.products WHERE id = $1 AND active FOR SHARE',
      [productId]
    );

    if (product.rowCount !== 1) {
      throw new Error('Product is missing or inactive');
    }

    const order = await client.query(
      "INSERT INTO capstone.orders (customer_id) VALUES ($1) RETURNING id",
      [customerId]
    );

    const orderId = order.rows[0].id;

    await client.query(
      'INSERT INTO capstone.order_items ' +
      '(order_id, product_id, quantity, unit_price) VALUES ($1, $2, $3, $4)',
      [orderId, productId, quantity, product.rows[0].price]
    );

    await client.query('COMMIT');
    return orderId;
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
~~~

## Chapter 98: 

[Open the chapter](98-all-code-samples.md)

This chapter has no fenced code samples.

## Chapter 99: 

[Open the chapter](99-complete-q-and-a.md)

This chapter has no fenced code samples.

---

| [Previous: Capstone: order management database](16-capstone-order-management-database.md) | [Notes index](../README.md) | [Next: Complete Q&A](99-complete-q-and-a.md) |
|:--|:--:|--:|
