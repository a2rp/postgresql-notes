# 13. PL/pgSQL functions and triggers

[Back to notes index](../README.md)

| [Previous: JSON and JSONB](12-json-and-jsonb.md) | [Notes index](../README.md) | [Next: Roles, privileges, and maintenance](14-roles-privileges-and-maintenance.md) |
|:--|:--:|--:|

This chapter introduces server-side functions and triggers, including when they help and when application logic is clearer.

## In this chapter

- SQL functions and PL/pgSQL blocks
- Parameters, return values, and exceptions
- Row-level triggers
- Trigger ordering, side effects, and testing
- Eight review questions

## Start with a SQL function

A SQL function gives a name and return type to a reusable expression or query. State volatility honestly: IMMUTABLE means the result depends only on its arguments, STABLE means it can depend on the current statement's database snapshot, and VOLATILE is the default for functions that can change or observe changing state.

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

This function reads table data, so it is STABLE rather than IMMUTABLE. A function's volatility declaration affects planning and must match its behavior.

## Use PL/pgSQL for procedural steps

PL/pgSQL adds variables, branches, loops, and exception handling around SQL. This function returns true only when it changes an existing pending order:

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

The status condition makes repeated calls safe: after the first successful change, the order is no longer pending. Keep the state transition rule in one place if several database clients must enforce the same rule.

## Add a row-level trigger

A trigger runs automatically when its configured table event occurs. First add an update timestamp and define a trigger function:

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

Attach it to updates:

~~~sql
DROP TRIGGER IF EXISTS products_set_updated_at ON store.products;

CREATE TRIGGER products_set_updated_at
BEFORE UPDATE ON store.products
FOR EACH ROW
EXECUTE FUNCTION store.set_product_updated_at();
~~~

In a BEFORE row trigger, NEW contains the row that will be written. Return NEW to continue the update with the changed timestamp. A statement-level trigger runs once per statement and does not have one NEW row.

Triggers can make writes harder to understand because effects happen implicitly. Use them for rules that truly belong to every database write, such as a consistent audit timestamp. Keep application-specific workflow visible in ordinary statements when possible.

## Errors and security

Raise a useful database error when a stored procedure must reject invalid input. Avoid catching every exception and silently continuing; that can turn a failed write into a misleading success.

Functions run with invoker privileges by default. SECURITY DEFINER changes the privilege model and needs careful ownership, a fixed safe search_path, and restricted execute permissions. Do not add it as a convenience shortcut.

## Review questions

1. What does IMMUTABLE promise about a function?
2. Why is a function that reads table rows not IMMUTABLE?
3. What does PL/pgSQL add around ordinary SQL statements?
4. What does the mark_order_paid function return when the order is not pending?
5. In a BEFORE row trigger, what does NEW represent?
6. When should an update timestamp trigger run?
7. Why can triggers make application behavior harder to trace?
8. What extra care does SECURITY DEFINER require?

## References

- [PostgreSQL: CREATE FUNCTION](https://www.postgresql.org/docs/18/sql-createfunction.html)
- [PostgreSQL: PL/pgSQL](https://www.postgresql.org/docs/18/plpgsql.html)
- [PostgreSQL: Triggers](https://www.postgresql.org/docs/18/triggers.html)

---

| [Previous: JSON and JSONB](12-json-and-jsonb.md) | [Notes index](../README.md) | [Next: Roles, privileges, and maintenance](14-roles-privileges-and-maintenance.md) |
|:--|:--:|--:|
