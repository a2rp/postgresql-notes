# 16. Capstone: order management database

[Back to notes index](../README.md)

| [Previous: Node.js and PostgreSQL](15-nodejs-and-postgresql.md) | [Notes index](../README.md) | [Next: All code samples](98-all-code-samples.md) |
|:--|:--:|--:|

This chapter brings the schema, constraints, queries, transactions, and JavaScript database access together in one small order-management design.

## In this chapter

- Customer, product, order, and order-item tables
- Data integrity and indexes
- A transaction that creates an order
- Useful reporting queries and review checks
- Eight review questions

## Build the schema

Run this setup once in a new practice database such as study_store. It creates a separate capstone schema so the earlier examples remain available.

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

The customer email is unique, prices and quantities are checked, and foreign keys connect each record to its parent. Deleting an order removes its line items, but deleting a product that appears in order history is blocked.

## Add a complete sample order

This data-modifying CTE inserts a customer, product, order, and line item without assuming identity values:

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

Run the seed statement once. A second run will fail on the unique email and SKU, which prevents duplicate sample records.

## Read order details and customer totals

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

Aggregate at the customer level for a compact account summary:

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

COUNT(DISTINCT o.id) prevents the item join from counting one order once per line. The amount is intentionally the sum of line values.

## Create an order in JavaScript

This function uses the pool and transaction pattern from Chapter 15. It validates simple inputs, reads price from the database, and commits the order and item together.

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

The foreign key rejects an unknown customer, while the CHECK constraint rejects a negative quantity or price. The application should still validate input so it can return a clear response before sending a query.

## Final checks

- Create the schema and tables in a fresh practice database.
- Run the seed statement once and inspect its returned identifiers.
- Run the detail query and customer summary.
- Use EXPLAIN on a query filtered by customer_id and created_at.
- Test a transaction that inserts an order and then deliberately violates a constraint; confirm that rollback leaves no partial order.
- Back up the practice database and restore it under another name.

## Review questions

1. Which constraints protect the capstone customer and product data?
2. Why does the sample insert use RETURNING instead of assuming identity values?
3. What does ON DELETE CASCADE do for an order's line items?
4. Why does the product foreign key use RESTRICT?
5. Why is the charged unit price copied onto an order item?
6. Why does the customer report count distinct order IDs?
7. Which database connection must be used for both inserts in the Node.js function?
8. What should be checked before relying on a database backup?

## References

- [PostgreSQL: Data definition](https://www.postgresql.org/docs/18/ddl.html)
- [PostgreSQL: Data manipulation](https://www.postgresql.org/docs/18/dml.html)
- [PostgreSQL: EXPLAIN](https://www.postgresql.org/docs/18/sql-explain.html)

---

| [Previous: Node.js and PostgreSQL](15-nodejs-and-postgresql.md) | [Notes index](../README.md) | [Next: All code samples](98-all-code-samples.md) |
|:--|:--:|--:|
