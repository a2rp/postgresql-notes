# 14. Roles, privileges, and maintenance

[Back to notes index](../README.md)

| [Previous: PL/pgSQL functions and triggers](13-plpgsql-functions-and-triggers.md) | [Notes index](../README.md) | [Next: Node.js and PostgreSQL](15-nodejs-and-postgresql.md) |
|:--|:--:|--:|

This chapter covers database access, least-privilege roles, backups, and routine maintenance.

## In this chapter

- Roles, ownership, and grants
- Application roles and schema privileges
- `pg_dump` and `pg_restore`
- Vacuum, analyze, logs, and safe operations
- Eight review questions

## Roles and least privilege

A role can own objects, receive privileges, or log in. A NOLOGIN role is useful as a permission group. Give application users only the access they need.

Run role and grant commands as a database administrator or the object owner:

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

The runtime role above cannot delete rows or create objects. Add those privileges only when the application has a clear need. Grant membership in the reader group if the runtime role should also read:

~~~sql
GRANT store_reader TO store_runtime;
~~~

New tables do not automatically inherit grants given to existing tables. The owner of future tables can set default privileges:

~~~sql
ALTER DEFAULT PRIVILEGES IN SCHEMA store
GRANT SELECT ON TABLES TO store_reader;
~~~

Default privileges apply to objects created later by the role that runs this command. Apply them for each object-creating role used by migrations.

## Authentication and secrets

Use PostgreSQL's host-based authentication rules to decide which users can connect from which addresses. Prefer SCRAM password authentication for password-based connections, restrict network access, and keep application credentials in an environment-specific secret store.

For an interactive password change, use the psql command \password so the password is not typed as a SQL literal into command history. Do not commit database passwords or production connection strings.

## Back up and restore

pg_dump creates a consistent logical backup of one database. The custom format supports selective restore and is read by pg_restore:

~~~sh
pg_dump --format=custom --dbname=study_store --file=study_store.dump
createdb study_store_restore
pg_restore --dbname=study_store_restore study_store.dump
~~~

Test a backup by restoring it into a separate database and checking important tables and queries. A backup that has never been restored is not a verified recovery plan.

pg_dump backs up one database's contents and schema. Roles and other cluster-wide objects need a separate globals export:

~~~sh
pg_dumpall --globals-only --file=cluster-roles.sql
~~~

Protect backup files because they may contain personal or confidential data. Automate backups and retain them according to a tested recovery policy.

## Routine maintenance and visibility

Autovacuum normally vacuums and analyzes tables as data changes. Vacuum reclaims reusable space and maintains visibility information. Analyze refreshes planner statistics. Avoid disabling autovacuum without a measured reason and a replacement plan.

~~~sql
VACUUM (ANALYZE) store.orders;

SELECT pid, usename, state, wait_event_type, wait_event
FROM pg_stat_activity
WHERE datname = current_database()
ORDER BY pid;
~~~

VACUUM cannot run inside a transaction block. VACUUM FULL rewrites a table and takes a strong lock, so it is not a routine substitute for regular vacuum.

## Review questions

1. What is the difference between a LOGIN role and a NOLOGIN role?
2. Why should application roles receive only the privileges they need?
3. Which privilege lets a role use objects inside a schema?
4. Who is affected by ALTER DEFAULT PRIVILEGES?
5. How can a password be changed interactively without placing it in SQL history?
6. What does pg_dump --format=custom create?
7. Why should a backup be restored in a test before relying on it?
8. What do VACUUM and ANALYZE maintain?

## References

- [PostgreSQL: Database roles](https://www.postgresql.org/docs/18/user-manag.html)
- [PostgreSQL: Privileges](https://www.postgresql.org/docs/18/ddl-priv.html)
- [PostgreSQL: pg_dump](https://www.postgresql.org/docs/18/app-pgdump.html)
- [PostgreSQL: Routine vacuuming](https://www.postgresql.org/docs/18/routine-vacuuming.html)

---

| [Previous: PL/pgSQL functions and triggers](13-plpgsql-functions-and-triggers.md) | [Notes index](../README.md) | [Next: Node.js and PostgreSQL](15-nodejs-and-postgresql.md) |
|:--|:--:|--:|
