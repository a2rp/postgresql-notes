# 15. Node.js and PostgreSQL

[Back to notes index](../README.md)

| [Previous: Roles, privileges, and maintenance](14-roles-privileges-and-maintenance.md) | [Notes index](../README.md) | [Next: Capstone: order management database](16-capstone-order-management-database.md) |
|:--|:--:|--:|

This chapter connects a JavaScript application to PostgreSQL with `pg`, parameterized queries, and a connection pool.

## In this chapter

- Installing and configuring `pg`
- Parameterized queries and safe input handling
- Pool usage, transactions, and error cleanup
- Mapping rows and handling dates and numeric values
- Eight review questions

## Add the PostgreSQL driver

The pg package, also called node-postgres, connects a JavaScript application to PostgreSQL. Start a Node.js project and add the driver:

~~~sh
npm init -y
npm install pg
~~~

The examples use CommonJS JavaScript. They work in a default Node.js project without adding a build step.

## Create and close a connection pool

A pool reuses a limited number of database connections. Create one pool for the application process, not a new pool for each request.

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

The pool reads the standard PGHOST, PGPORT, PGDATABASE, PGUSER, and PGPASSWORD environment variables when they are set. Supply secrets through the deployment environment or a secret manager. Do not put a real password in source control.

Choose pool limits with the database's total connection budget in mind. Ten connections per application process can become one hundred when ten processes run.

## Pass values as parameters

Use numbered parameters for values. PostgreSQL receives the query structure separately from the data values, so user input cannot change the SQL syntax.

~~~js
async function findCustomerByEmail(email) {
  const result = await pool.query(
    'SELECT id, full_name, email FROM store.customers WHERE email = $1',
    [email]
  );

  return result.rows[0] ?? null;
}
~~~

Do not concatenate user input into SQL:

~~~js
// Unsafe: input can change the SQL statement.
const unsafeSql = "SELECT id FROM store.customers WHERE email = '" + email + "'";
~~~

Parameters represent values, not table or column names. If a sort column must be chosen dynamically, map a known user choice to a fixed allowlist of SQL fragments and keep values parameterized.

~~~js
const sortColumns = new Map([
  ['name', 'name'],
  ['price', 'price']
]);

function productSortSql(requestedSort) {
  const column = sortColumns.get(requestedSort) ?? 'name';
  return 'SELECT id, name, price FROM store.products ORDER BY ' + column + ', id';
}
~~~

Only the fixed strings in sortColumns enter the SQL statement. Never use arbitrary request text as an identifier.

## Use one client for a transaction

pool.query is convenient for one independent query. A transaction belongs to one connection, so check out a client and send every transaction statement through that same client.

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

The product price comes from the database instead of from an untrusted client request. The shared row lock keeps that product row from being changed or deleted until this transaction completes. The order and item are committed together. Add input validation before the transaction and handle rollback failures in a production error path.

Always release a checked-out client, including after an error. A leaked client can exhaust the pool and leave later requests waiting. Close the pool during process shutdown with await pool.end().

## Know how values are returned

The driver returns rows as JavaScript objects in result.rows. PostgreSQL bigint and numeric values are commonly returned as strings to avoid silently losing precision in JavaScript numbers. Convert only when the expected range and precision are known.

JSON and JSONB values are parsed into JavaScript values. PostgreSQL timestamps are commonly parsed as JavaScript Date objects, which have millisecond precision and time-zone behavior to keep in mind. Use timestamptz for instants and serialize dates consistently at API boundaries.

## Review questions

1. Why is a connection pool useful in an application?
2. Where should database credentials come from?
3. How do parameterized queries prevent SQL input from changing query structure?
4. Can a $1 parameter stand for a table name?
5. Why must every statement in a transaction use the same client?
6. Why does the order sample read the product price from the database?
7. What can happen if a checked-out client is never released?
8. Why might node-postgres return a bigint or numeric value as a string?

## References

- [node-postgres: Connecting](https://node-postgres.com/features/connecting)
- [node-postgres: Queries and parameters](https://node-postgres.com/features/queries)
- [node-postgres: Pooling](https://node-postgres.com/features/pooling)
- [node-postgres: Transactions](https://node-postgres.com/features/transactions)
- [node-postgres: Data types](https://node-postgres.com/features/types)

---

| [Previous: Roles, privileges, and maintenance](14-roles-privileges-and-maintenance.md) | [Notes index](../README.md) | [Next: Capstone: order management database](16-capstone-order-management-database.md) |
