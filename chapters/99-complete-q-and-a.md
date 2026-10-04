# Complete Q&A

[Back to notes index](../README.md)

| [Previous: All code samples](98-all-code-samples.md) | [Notes index](../README.md) | [Next: Notes index](../README.md) |
|:--|:--:|--:|

This appendix will gather eight answered review questions from each core chapter for a complete 128-question review set.

## Chapter 1: PostgreSQL foundations

1. **What does the PostgreSQL server do, and what does psql do?**  
   **Answer:** The server stores data, runs SQL, and enforces access rules. psql connects as a client, sends SQL, and displays the results.
2. **What connection details identify a PostgreSQL session?**  
   **Answer:** The host, port, database, role, and authentication method identify the target and how the client connects.
3. **How is a psql meta-command different from an SQL statement?**  
   **Answer:** A psql meta-command begins with a backslash and is handled by the client. SQL is sent to and executed by the server.
4. **What is the default PostgreSQL port, and can it be changed?**  
   **Answer:** The default is 5432. A server can be configured to listen on another port.
5. **Why does creating a database not automatically move the current session into it?**  
   **Answer:** A connection is already attached to one database. Create the database, then open a new connection that names it.
6. **Which query returns the current database and role?**  
   **Answer:** SELECT current_database(), current_user; returns both values.
7. **What should you check when a connection is refused?**  
   **Answer:** Check that the server is running, the host and port are correct, and the server is listening on that address.
8. **Why should a database role receive only the permissions it needs?**  
   **Answer:** Limited permissions reduce the damage a mistake or compromised application credential can cause.

## Chapter 2: Databases, schemas, and tables

1. **How is a schema different from a database?**  
   **Answer:** A database is a connection boundary. A schema is a namespace inside that database for tables and other objects.
2. **Why is a named application schema useful?**  
   **Answer:** It groups related objects, reduces naming collisions, and makes access grants easier to scope.
3. **Which PostgreSQL type stores an exact decimal amount?**  
   **Answer:** numeric with an appropriate precision and scale stores exact decimal values.
4. **What is the difference between date and timestamptz?**  
   **Answer:** date stores a calendar date. timestamptz stores an instant and displays it using the session time zone.
5. **What does GENERATED ALWAYS AS IDENTITY provide?**  
   **Answer:** PostgreSQL generates a numeric value for inserted rows when the insert omits that identity column.
6. **Why should an unquoted identifier usually use lowercase snake_case?**  
   **Answer:** PostgreSQL folds unquoted names to lowercase. Lowercase names avoid needing quotes at every reference.
7. **How can you inspect a table's columns with SQL?**  
   **Answer:** Query information_schema.columns with the table_schema and table_name filters, then order by ordinal_position.
8. **Why might a required column need a staged migration on a populated table?**  
   **Answer:** Existing rows need valid values before NOT NULL can be enforced, so add, backfill, check, and then constrain the column.

## Chapter 3: Selecting, filtering, and sorting

1. **Why should an application query list the columns it needs?**  
   **Answer:** Explicit columns make the query clear and prevent new table columns from silently changing application output.
2. **What does DISTINCT do, and when can it hide a join problem?**  
   **Answer:** DISTINCT removes duplicate result rows. It can hide an accidental row multiplication that should be fixed in the join.
3. **What is the difference between WHERE and HAVING?**  
   **Answer:** WHERE filters input rows before grouping. HAVING filters groups after aggregate values are computed.
4. **Why does email = NULL not find rows with a missing email?**  
   **Answer:** Comparisons with NULL produce unknown. Use IS NULL or IS NOT NULL.
5. **What does BETWEEN include?**  
   **Answer:** It includes both its lower and upper endpoints.
6. **Why should ORDER BY include a unique tie-breaker for pagination?**  
   **Answer:** Equal sort values otherwise have no defined relative order, so rows can move between pages or repeat.
7. **How does ILIKE differ from LIKE in PostgreSQL?**  
   **Answer:** ILIKE performs case-insensitive pattern matching under the active collation; LIKE is case-sensitive.
8. **When is keyset pagination preferable to a large OFFSET?**  
   **Answer:** Keyset pagination is useful for large or changing result sets because it continues from the last sort key instead of skipping many rows.

## Chapter 4: Expressions, functions, and dates

1. **What does an SQL expression produce?**  
   **Answer:** An expression evaluates to a value that can be selected, filtered, grouped, or used in another expression.
2. **How does CASE choose its result?**  
   **Answer:** It returns the result for the first matching WHEN condition, or ELSE when no condition matches.
3. **What values does COALESCE return when its first argument is NULL?**  
   **Answer:** It returns the first argument in its list that is not NULL, or NULL if every argument is NULL.
4. **How is NULLIF useful when normalizing a known placeholder?**  
   **Answer:** NULLIF returns NULL when two values are equal, so a placeholder such as unknown can be converted to NULL.
5. **Why is numeric suitable for exact decimal amounts?**  
   **Answer:** numeric stores decimal values exactly within its declared precision, unlike approximate floating-point types.
6. **What changes when a timestamptz value is displayed in a different session time zone?**  
   **Answer:** Its displayed wall-clock time and offset change, but it still represents the same instant.
7. **Why is a half-open timestamp range useful for selecting one day?**  
   **Answer:** It includes midnight at the start and excludes midnight at the next day, covering every precision within that day.
8. **Why should formatted date text stay at the presentation edge?**  
   **Answer:** Typed date values compare and sort correctly; formatted text is intended for display and can be ambiguous.

## Chapter 5: Joins and relationships

1. **What does an inner join return?**  
   **Answer:** It returns pairs of rows whose join condition is true.
2. **What does a left join put in right-side columns when there is no match?**  
   **Answer:** It fills those columns with NULL while keeping the left-side row.
3. **How can a query find customers who have no orders?**  
   **Answer:** Left join orders by customer_id and filter for a non-nullable order key being NULL.
4. **Why are table aliases useful in a multi-table query?**  
   **Answer:** Aliases make column sources shorter and clearer, especially when tables share column names.
5. **Why can a one-to-many join repeat the parent row?**  
   **Answer:** Each matching child row forms a separate result pair with that parent.
6. **What can happen if a filter on the right side of a left join is placed in WHERE?**  
   **Answer:** It can remove NULL-extended rows and make the result act like an inner join for that filter.
7. **Why can summing an order-level value after joining line items produce an inflated total?**  
   **Answer:** The order-level value appears once for every joined line, so it may be added multiple times.
8. **What is the difference between COUNT(*) and COUNT(o.id) after a left join?**  
   **Answer:** COUNT(*) counts the NULL-extended result row, while COUNT(o.id) ignores the NULL order id and returns zero for no match.

## Chapter 6: Aggregation and grouping

1. **What is the difference between COUNT(*) and COUNT(column)?**  
   **Answer:** COUNT(*) counts every input row. COUNT(column) counts only rows where that column is not NULL.
2. **Does COUNT(DISTINCT column) count NULL?**  
   **Answer:** No. Aggregate COUNT ignores NULL, including when DISTINCT is used.
3. **What do SUM and AVG return when there are no non-null input values?**  
   **Answer:** Both return NULL. COALESCE can display a chosen fallback such as zero.
4. **What does GROUP BY do to the input rows?**  
   **Answer:** It partitions rows by the grouped values so aggregate functions produce a result for each group.
5. **How are WHERE and HAVING different?**  
   **Answer:** WHERE filters source rows before aggregation. HAVING filters the grouped results.
6. **Why can COUNT(*) return one for a parent with no matches after a left join?**  
   **Answer:** The left join still emits one NULL-extended row for that parent, and COUNT(*) counts it.
7. **What does the FILTER clause on an aggregate do?**  
   **Answer:** It limits the input rows for that one aggregate without removing rows used by other aggregates in the same result.
8. **Why should a total be calculated at the same level as the rows being summed?**  
   **Answer:** Joining at a different level can duplicate values and make a total incorrect.

## Chapter 7: Subqueries, CTEs, and set operations

1. **What shape must a scalar subquery return?**  
   **Answer:** It must return one column and at most one row. More than one row causes an error.
2. **When is EXISTS clearer than joining a related table?**  
   **Answer:** Use EXISTS when the question asks whether a matching row exists and the parent should not be repeated for every match.
3. **Why can NOT IN behave unexpectedly if its input contains NULL?**  
   **Answer:** A comparison against NULL can become unknown, so rows may not pass the predicate as expected.
4. **What is a CTE?**  
   **Answer:** A CTE is a named query result scoped to one statement, introduced with WITH.
5. **Is every non-recursive CTE materialized as a temporary table?**  
   **Answer:** No. PostgreSQL can fold a non-recursive CTE into its surrounding query when it is safe.
6. **What are the anchor and recursive parts of a recursive CTE?**  
   **Answer:** The anchor produces the starting rows. The recursive part repeatedly derives more rows from earlier results.
7. **What is the difference between UNION and UNION ALL?**  
   **Answer:** UNION removes duplicate rows; UNION ALL keeps them.
8. **What does EXCEPT return?**  
   **Answer:** It returns distinct rows from the first query that do not appear in the second query.

## Chapter 8: Inserts, updates, deletes, and transactions

1. **Why should an INSERT list its target columns?**  
   **Answer:** It documents which values go into which columns and avoids relying on table column order.
2. **What does RETURNING provide?**  
   **Answer:** It returns columns from rows inserted, updated, or deleted by that statement.
3. **What can happen if an UPDATE statement has no WHERE clause?**  
   **Answer:** It updates every row in the target table.
4. **When is ON CONFLICT DO NOTHING appropriate?**  
   **Answer:** Use it when a known uniqueness conflict means the proposed row should simply be skipped.
5. **What does EXCLUDED refer to in an upsert?**  
   **Answer:** It refers to the row proposed by INSERT that encountered a conflict.
6. **What should an application do if one statement in a transaction fails?**  
   **Answer:** Roll back the transaction, report or handle the error, and avoid treating partial work as committed.
7. **Why must the same connection be used for every statement in a transaction?**  
   **Answer:** A transaction is scoped to one PostgreSQL session and its connection.
8. **When is SELECT FOR UPDATE useful?**  
   **Answer:** It locks selected rows when a workflow must read a row and make a decision based on that state before the transaction finishes.

## Chapter 9: Keys, constraints, and data integrity

1. **What guarantees does a primary key provide?**  
   **Answer:** It uniquely identifies each row and disallows NULL in its key columns.
2. **What does a foreign key protect?**  
   **Answer:** It ensures a referencing value points to a row that exists in the referenced table.
3. **How do RESTRICT and CASCADE differ on parent deletion?**  
   **Answer:** RESTRICT blocks deletion while children exist. CASCADE deletes the matching child rows.
4. **Why should a CHECK column also be NOT NULL when absence is invalid?**  
   **Answer:** A CHECK passes when its expression is true or NULL, so it alone does not reject a missing value.
5. **Why can a UNIQUE column contain multiple NULL values by default?**  
   **Answer:** PostgreSQL treats NULL values as distinct for a regular unique constraint.
6. **Are identity sequence values guaranteed to be gapless?**  
   **Answer:** No. Failed or rolled-back inserts can consume sequence values.
7. **Does PostgreSQL automatically index a foreign-key column on the child table?**  
   **Answer:** No. It indexes primary and unique keys, but a child-side index must be added when useful.
8. **What does VALIDATE CONSTRAINT check?**  
   **Answer:** It scans existing rows to verify that they satisfy the constraint.

## Chapter 10: Indexes and query plans

1. **What does an index cost in addition to storage?**  
   **Answer:** It adds work to writes because PostgreSQL must maintain the index on inserts, updates, and deletes.
2. **What kinds of lookups does a B-tree support?**  
   **Answer:** B-trees support equality, range conditions, and ordered retrieval.
3. **Why does the leading column matter in a multicolumn B-tree?**  
   **Answer:** The index is ordered by its first column, so searches on leading columns are generally the most directly supported.
4. **What is a partial index useful for?**  
   **Answer:** It indexes only rows satisfying a predicate, which can reduce size for queries targeting that subset.
5. **What does EXPLAIN ANALYZE do that EXPLAIN alone does not?**  
   **Answer:** It executes the statement and reports actual row counts and execution measurements along with the plan.
6. **Why can a sequential scan be the right plan?**  
   **Answer:** It can be cheaper for a small table or when a query needs a large share of its rows.
7. **What should you inspect when estimated and actual row counts differ greatly?**  
   **Answer:** Check data distribution and statistics, then examine the plan nodes that use those estimates.
8. **What does ANALYZE collect for the planner?**  
   **Answer:** It gathers statistics about table columns and value distributions used to estimate query costs.

## Chapter 11: Views and generated data

1. **Does a normal view store a snapshot of its result rows?**  
   **Answer:** No. It stores a query definition that is evaluated against the current base tables when queried.
2. **When can a view make a repeated query easier to maintain?**  
   **Answer:** When several callers need the same named projection or join and one definition can keep that query consistent.
3. **What is the main difference between a materialized view and a normal view?**  
   **Answer:** A materialized view stores result rows until refreshed; a normal view evaluates its query when read.
4. **What does REFRESH MATERIALIZED VIEW do?**  
   **Answer:** It recomputes and replaces the stored result rows from the materialized view's defining query.
5. **What conditions are needed for a concurrent materialized-view refresh?**  
   **Answer:** The view must already be populated and have a qualifying unique index covering all rows.
6. **When is a generated column evaluated?**  
   **Answer:** PostgreSQL computes it from the row's other values when the row is written or read, according to whether it is stored or virtual.
7. **What kinds of expressions can a generated column use?**  
   **Answer:** It can use immutable expressions based on columns in the same row, without subqueries or references to other rows.
8. **Why should an order item keep the price charged at the time of purchase?**  
   **Answer:** It preserves the historical transaction price even if the current product price changes later.

## Chapter 12: JSON and JSONB

1. **What does JSONB store, and what textual details does it not preserve?**  
   **Answer:** It stores a parsed binary JSON structure but does not preserve input whitespace or object-key order.
2. **When is a regular relational column a better choice than JSONB?**  
   **Answer:** Use a column for stable data that needs constraints, joins, or frequent filtering.
3. **What is the difference between -> and ->>?**  
   **Answer:** -> returns a JSON value. ->> returns the selected value as text.
4. **What does the question-mark operator test?**  
   **Answer:** It tests whether a top-level key or string element exists in a JSONB value.
5. **What does the @> operator mean for JSONB?**  
   **Answer:** It tests whether the left JSONB value contains the structure on the right.
6. **What index type is commonly used for JSONB containment?**  
   **Answer:** GIN is commonly used for JSONB containment and key searches.
7. **What does the final true argument to jsonb_set request?**  
   **Answer:** It allows the final path element to be created when it does not already exist.
8. **Why should an application validate JSON document shape?**  
   **Answer:** JSONB validates JSON syntax, not the application's allowed keys, types, or business rules.

## Chapter 13: PL/pgSQL functions and triggers

1. **What does IMMUTABLE promise about a function?**  
   **Answer:** It promises that the function's result depends only on its arguments and does not change for the same inputs.
2. **Why is a function that reads table rows not IMMUTABLE?**  
   **Answer:** Its result can change when the table data changes even if its arguments stay the same.
3. **What does PL/pgSQL add around ordinary SQL statements?**  
   **Answer:** It adds procedural features such as local variables, branches, loops, and exception handling.
4. **What does the mark_order_paid function return when the order is not pending?**  
   **Answer:** It returns false because the conditional UPDATE changes and returns no row.
5. **In a BEFORE row trigger, what does NEW represent?**  
   **Answer:** NEW is the row value that the triggering insert or update is about to write.
6. **When should an update timestamp trigger run?**  
   **Answer:** It should run before each row update when every database writer must maintain the timestamp consistently.
7. **Why can triggers make application behavior harder to trace?**  
   **Answer:** They run implicitly as a side effect of another statement.
8. **What extra care does SECURITY DEFINER require?**  
   **Answer:** It needs a trusted owner, a fixed safe search_path, and restricted execute permissions.

## Chapter 14: Roles, privileges, and maintenance

1. **What is the difference between a LOGIN role and a NOLOGIN role?**  
   **Answer:** A LOGIN role can start a session. A NOLOGIN role can group privileges without connecting directly.
2. **Why should application roles receive only the privileges they need?**  
   **Answer:** This limits the effects of application bugs and compromised credentials.
3. **Which privilege lets a role use objects inside a schema?**  
   **Answer:** USAGE on the schema allows name resolution and access to objects for which the role also has privileges.
4. **Who is affected by ALTER DEFAULT PRIVILEGES?**  
   **Answer:** It affects future objects created by the role that runs the command, in the specified scope.
5. **How can a password be changed interactively without placing it in SQL history?**  
   **Answer:** Use psql's \password command to enter the new password through its prompt.
6. **What does pg_dump --format=custom create?**  
   **Answer:** It creates a custom-format logical backup of one database that pg_restore can selectively restore.
7. **Why should a backup be restored in a test before relying on it?**  
   **Answer:** A test restore verifies that the backup can actually recover usable data and schema.
8. **What do VACUUM and ANALYZE maintain?**  
   **Answer:** VACUUM manages dead-row space and visibility information. ANALYZE refreshes planner statistics.

## Chapter 15: Node.js and PostgreSQL

1. **Why is a connection pool useful in an application?**  
   **Answer:** It reuses a bounded set of database connections instead of repeatedly opening new ones.
2. **Where should database credentials come from?**  
   **Answer:** From protected deployment configuration or a secret manager, not committed source code.
3. **How do parameterized queries prevent SQL input from changing query structure?**  
   **Answer:** The query text and values are sent separately, so the server treats supplied values as data.
4. **Can a $1 parameter stand for a table name?**  
   **Answer:** No. Query parameters represent values, not SQL identifiers.
5. **Why must every statement in a transaction use the same client?**  
   **Answer:** PostgreSQL keeps a transaction within one connection; another client is another session.
6. **Why does the order sample read the product price from the database?**  
   **Answer:** It avoids trusting a client-supplied price and records the price currently stored for the product.
7. **What can happen if a checked-out client is never released?**  
   **Answer:** It can exhaust the pool so later requests wait for a connection that never becomes available.
8. **Why might node-postgres return a bigint or numeric value as a string?**  
   **Answer:** JavaScript numbers cannot exactly represent every large integer or decimal, so strings preserve the database value.

## Chapter 16: Capstone order management database

1. **Which constraints protect the capstone customer and product data?**  
   **Answer:** Primary keys identify rows, unique constraints protect email and SKU, checks validate price and quantity, and foreign keys connect records.
2. **Why does the sample insert use RETURNING instead of assuming identity values?**  
   **Answer:** Generated identity values depend on database state, so RETURNING supplies the actual values created by the insert.
3. **What does ON DELETE CASCADE do for an order's line items?**  
   **Answer:** Deleting an order automatically deletes its dependent order-item rows.
4. **Why does the product foreign key use RESTRICT?**  
   **Answer:** It prevents deleting a product that is part of retained order history.
5. **Why is the charged unit price copied onto an order item?**  
   **Answer:** It preserves the agreed price for that order if the product's current price later changes.
6. **Why does the customer report count distinct order IDs?**  
   **Answer:** Each order can produce several joined line rows, so counting distinct IDs prevents counting one order more than once.
7. **Which database connection must be used for both inserts in the Node.js function?**  
   **Answer:** Both inserts must use the same checked-out client that began the transaction.
8. **What should be checked before relying on a database backup?**  
   **Answer:** Restore it into a separate database and verify that expected tables and queries work.

---

| [Previous: All code samples](98-all-code-samples.md) | [Notes index](../README.md) | [Next: Notes index](../README.md) |
|:--|:--:|--:|
