# 1. PostgreSQL foundations

[Back to notes index](../README.md)

| [Previous: Notes index](../README.md) | [Notes index](../README.md) | [Next: Databases, schemas, and tables](02-databases-schemas-and-tables.md) |
|:--|:--:|--:|

PostgreSQL is a relational database system. A server process stores and protects data, while client programs send SQL statements and display results. psql is the command-line client used in these notes.

## In this chapter

- PostgreSQL server and client roles
- Connecting with `psql`
- Creating a database and checking a connection
- Reading basic query output
- Eight review questions

## Server, database, and connection

A PostgreSQL server can host several databases. Each connection works with one database at a time. Inside a database, schemas group objects such as tables and functions. A role is the identity used to connect and controls what the session may do.

The client and server can run on the same computer or communicate over a network. A connection needs a host, port, database, role, and authentication method. The usual default port is 5432, but installations can use a different port.

## Connect with psql

The connection flags make the target explicit. Replace the example database and role with values created for your installation.

~~~sh
psql --host localhost --port 5432 --username app_user --dbname postgres
~~~

psql prompts for a password when the server requests one. Avoid putting a real password directly in a shell command because commands can be saved in shell history or process details. For local practice, use the password prompt or a properly protected password file.

After connecting, these psql meta-commands inspect the session. They begin with a backslash and are interpreted by psql, not by the SQL server.

~~~text
\conninfo
\l
\du
\q
~~~

Use SQL to ask the server about the current session:

~~~sql
SELECT version();
SELECT current_database(), current_user;
SHOW server_version;
~~~

version() includes build information. server_version is easier to compare in scripts. SQL statements end with a semicolon; psql waits for it before sending a statement.

## Create a practice database

The createdb program is a client utility that asks the server to create a database. Run it from a terminal, not inside a SQL prompt.

~~~sh
createdb --host localhost --port 5432 --username app_user study_store
psql --host localhost --port 5432 --username app_user --dbname study_store
~~~

Or create the database from a connection that has permission to do so:

~~~sql
CREATE DATABASE study_store;
~~~

Database creation is separate from connecting to that database. After CREATE DATABASE, open a new connection with --dbname study_store.

## A first query and readable output

~~~sql
SELECT 'PostgreSQL is ready' AS status,
       current_timestamp AS checked_at;
~~~

In psql, \x toggles expanded display for wide rows and \timing toggles query timing. These are useful display settings, not SQL features.

~~~text
\x
\timing
~~~

## Common connection problems

- **Connection refused:** the server may be stopped, the host may be wrong, or the port may not be listening.
- **Password authentication failed:** verify the role name and password, then check the server's configured authentication rules.
- **Database does not exist:** connect to an existing database such as postgres, then create the intended database if your role is allowed.
- **Permission denied:** authentication succeeded, but the role lacks permission for that operation.
- **psql is not recognized:** install the client tools or add their executable directory to the terminal's PATH.

Do not fix a connection issue by making the server accept all users without passwords. Use a dedicated role and only grant the access needed for the task.

## Small practice check

Connect to study_store, print the server version and current role, then quit. Confirm that the database name shown by \conninfo is the one you expected before creating tables.

## Review questions

1. What does the PostgreSQL server do, and what does psql do?
2. What connection details identify a PostgreSQL session?
3. How is a psql meta-command different from an SQL statement?
4. What is the default PostgreSQL port, and can it be changed?
5. Why does creating a database not automatically move the current session into it?
6. Which query returns the current database and role?
7. What should you check when a connection is refused?
8. Why should a database role receive only the permissions it needs?

## References

- [PostgreSQL: What is PostgreSQL?](https://www.postgresql.org/docs/18/intro-whatis.html)
- [PostgreSQL: psql](https://www.postgresql.org/docs/18/app-psql.html)
- [PostgreSQL: Connecting to a database](https://www.postgresql.org/docs/18/libpq-connect.html)

---

| [Previous: Notes index](../README.md) | [Notes index](../README.md) | [Next: Databases, schemas, and tables](02-databases-schemas-and-tables.md) |
|:--|:--:|--:|
