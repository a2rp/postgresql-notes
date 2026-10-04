# 4. Expressions, functions, and dates

[Back to notes index](../README.md)

| [Previous: Selecting, filtering, and sorting](03-selecting-filtering-and-sorting.md) | [Notes index](../README.md) | [Next: Joins and relationships](05-joins-and-relations.md) |
|:--|:--:|--:|

This chapter uses expressions, conditional logic, built-in functions, and date and time types.

## In this chapter

- Arithmetic and text expressions
- `CASE`, `COALESCE`, and `NULLIF`
- String and numeric functions
- Date, time, interval, and time-zone basics
- Eight review questions

## Expressions and conditional values

An expression produces a value. It can combine columns, constants, operators, and functions. Give calculated output a clear alias.

~~~sql
SELECT id,
       price,
       price * 1.10 AS price_with_tax
FROM store.products
WHERE active;
~~~

CASE chooses a value based on conditions. Put the most specific condition first and include ELSE so the result is defined for every row.

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

COALESCE returns the first non-null argument. NULLIF returns NULL when its two arguments are equal. Both can express a useful rule without a long CASE expression.

~~~sql
SELECT COALESCE(NULL::text, 'not provided') AS display_name;
SELECT NULLIF('unknown', 'unknown') AS normalized_value;
~~~

Use these functions with care: replacing NULL with a display value in a query is different from changing the stored value.

## Text and numeric functions

~~~sql
SELECT lower(trim('  PostgreSQL  ')) AS normalized_text,
       char_length('database') AS character_count,
       round(19.995::numeric, 2) AS rounded_amount;
~~~

PostgreSQL numeric rounds exact decimal values. Floating-point values are approximate, so their displayed last digits can vary with calculations.

Common text operations include concatenation with ||, substring extraction with substring, and replacement with replace. Prefer parameterized query inputs for values instead of building SQL text by concatenation.

~~~sql
SELECT full_name || ' <' || email || '>' AS contact
FROM store.customers;
~~~

## Dates, timestamps, and time zones

Use date for a calendar date. Use timestamptz for an instant such as when an order was created. The session time zone changes how a timestamptz is displayed, not the stored instant.

~~~sql
SELECT CURRENT_DATE AS today,
       CURRENT_TIMESTAMP AS now_with_time_zone,
       current_setting('TimeZone') AS session_time_zone;
~~~

CURRENT_TIMESTAMP is fixed at the start of the current transaction. clock_timestamp() reads the wall clock when called. Most business records should use a transaction-stable creation time.

An interval represents a span of time. Date arithmetic can use an integer number of days or an interval:

~~~sql
SELECT CURRENT_DATE + 7 AS one_week_from_today,
       CURRENT_TIMESTAMP + interval '90 minutes' AS later_today;
~~~

Filter a day of timestamptz values with a half-open range. This includes every instant from midnight up to, but not including, the next midnight.

~~~sql
SELECT id, customer_id, created_at
FROM store.orders
WHERE created_at >= TIMESTAMPTZ '2026-10-01 00:00:00+00'
  AND created_at <  TIMESTAMPTZ '2026-10-02 00:00:00+00';
~~~

Avoid casting the indexed timestamp column to date in the filter. A range condition is clear about the time zone and is easier to support with a normal index on created_at.

For calendar reporting in a chosen zone, convert the instant to local wall time before truncating:

~~~sql
SELECT date_trunc('month', created_at AT TIME ZONE 'Asia/Kolkata') AS local_month,
       count(*) AS order_count
FROM store.orders
GROUP BY local_month
ORDER BY local_month;
~~~

to_char is useful for presentation, but formatted text should not be used as the stored date or as a replacement for typed date comparisons.

## Review questions

1. What does an SQL expression produce?
2. How does CASE choose its result?
3. What values does COALESCE return when its first argument is NULL?
4. How is NULLIF useful when normalizing a known placeholder?
5. Why is numeric suitable for exact decimal amounts?
6. What changes when a timestamptz value is displayed in a different session time zone?
7. Why is a half-open timestamp range useful for selecting one day?
8. Why should formatted date text stay at the presentation edge?

## References

- [PostgreSQL: Conditional expressions](https://www.postgresql.org/docs/18/functions-conditional.html)
- [PostgreSQL: String functions and operators](https://www.postgresql.org/docs/18/functions-string.html)
- [PostgreSQL: Date and time types](https://www.postgresql.org/docs/18/datatype-datetime.html)
- [PostgreSQL: Date and time functions](https://www.postgresql.org/docs/18/functions-datetime.html)

---

| [Previous: Selecting, filtering, and sorting](03-selecting-filtering-and-sorting.md) | [Notes index](../README.md) | [Next: Joins and relationships](05-joins-and-relations.md) |
