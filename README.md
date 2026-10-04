# PostgreSQL Study Notes

These are my personal study notes from learning PostgreSQL and using relational databases in projects. They collect the core ideas, SQL patterns, and practical checks I want to be able to revisit when designing, querying, and maintaining a database.

The examples use PostgreSQL 18 and build on one small order-management database, so tables and queries connect across chapters. SQL is the main focus. The application chapter uses JavaScript with Node.js and the `pg` package.

## Core topics

1. [PostgreSQL foundations](chapters/01-postgresql-foundations.md)
2. [Databases, schemas, and tables](chapters/02-databases-schemas-and-tables.md)
3. [Selecting, filtering, and sorting](chapters/03-selecting-filtering-and-sorting.md)
4. [Expressions, functions, and dates](chapters/04-expressions-functions-and-dates.md)
5. [Joins and relationships](chapters/05-joins-and-relations.md)
6. [Aggregation and grouping](chapters/06-aggregation-and-grouping.md)
7. [Subqueries, CTEs, and set operations](chapters/07-subqueries-ctes-and-set-operations.md)
8. [Inserts, updates, deletes, and transactions](chapters/08-inserts-updates-deletes-and-transactions.md)
9. [Keys, constraints, and data integrity](chapters/09-keys-constraints-and-data-integrity.md)
10. [Indexes and query plans](chapters/10-indexes-and-query-plans.md)
11. [Views and generated data](chapters/11-views-and-generated-data.md)
12. [JSON and JSONB](chapters/12-json-and-jsonb.md)
13. [PL/pgSQL functions and triggers](chapters/13-plpgsql-functions-and-triggers.md)
14. [Roles, privileges, and maintenance](chapters/14-roles-privileges-and-maintenance.md)
15. [Node.js and PostgreSQL](chapters/15-nodejs-and-postgresql.md)
16. [Capstone: order management database](chapters/16-capstone-order-management-database.md)

## How I use these notes

I keep the chapters focused on the concepts I need most often. Each one explains the purpose of a feature, shows SQL that can be adapted to a real schema, and points out mistakes that are easy to make. The examples use named tables and consistent columns so I can follow how a relational design grows from a few rows into a working application database.

The last chapters gather the examples and review questions in one place. I update these notes as I test queries and use the ideas in my own work.

## Reference

- [PostgreSQL 18 documentation](https://www.postgresql.org/docs/18/)
- [PostgreSQL SQL language reference](https://www.postgresql.org/docs/18/sql.html)
- [PostgreSQL release notes](https://www.postgresql.org/docs/release/)
- [node-postgres documentation](https://node-postgres.com/)

## Links

- Portfolio: https://www.ashishranjan.net
- GitHub: https://github.com/a2rp
- CodePen: https://codepen.io/ash1198
- LinkedIn: https://www.linkedin.com/in/aashishranjan
- Facebook: https://www.facebook.com/theash.ashish/
- YouTube: https://www.youtube.com/@ashishranjan-ashz?sub_confirmation=1
- Email: mailto:ash.ranjan09@gmail.com

## Support

- Support: https://a2rp-donation-page.netlify.app/
- Buy Me a Coffee: https://buymeacoffee.com/ashishranjan
- Patreon: https://www.patreon.com/ashishranjan
