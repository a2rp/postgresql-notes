# 12. JSON and JSONB

[Back to notes index](../README.md)

| [Previous: Views and generated data](11-views-and-generated-data.md) | [Notes index](../README.md) | [Next: PL/pgSQL functions and triggers](13-plpgsql-functions-and-triggers.md) |
|:--|:--:|--:|

This chapter stores and queries structured JSON values while keeping relational columns for stable, important fields.

## In this chapter

- `json` and `jsonb` differences
- Reading and updating JSON fields
- Containment operators and JSONPath basics
- Indexing JSONB and choosing a relational design
- Eight review questions

## Decide what belongs in JSONB

Use ordinary columns for stable fields that need constraints, joins, or frequent filtering. JSONB is useful for optional attributes whose shape varies by record. It stores a parsed binary representation that PostgreSQL can query and index; it does not preserve the original whitespace or object-key order.

Add a JSONB column with an empty object as the default:

~~~sql
ALTER TABLE store.products
ADD COLUMN IF NOT EXISTS attributes jsonb NOT NULL DEFAULT '{}'::jsonb;
~~~

An empty object is different from SQL NULL. It says that the product has no supplied attributes while keeping the column present.

~~~sql
UPDATE store.products
SET attributes = '{"color":"black","shipping":{"weight_grams":240},"tags":["gift","travel"]}'::jsonb
WHERE sku = 'MUG-01';
~~~

JSON input must be valid JSON. A JSON number, string, boolean, object, array, and JSON null are distinct JSON values.

## Read JSON values

The -> operator returns a JSON value. The ->> operator returns text. For a nested object, chain operators or use a path:

~~~sql
SELECT sku,
       attributes -> 'tags' AS tags_json,
       attributes ->> 'color' AS color_text,
       attributes #>> '{shipping,weight_grams}' AS nested_weight_text
FROM store.products
WHERE sku = 'MUG-01';
~~~

The final expression returns the nested shipping weight as text. Paths should match the actual document structure.

To test that a key exists, use the question-mark operator:

~~~sql
SELECT sku
FROM store.products
WHERE attributes ? 'color';
~~~

Key existence is not the same as a key whose JSON value is null. Use -> or JSONPath checks when the difference matters.

## Filter with containment and add an index

The containment operator @> checks whether the left JSONB value contains the right structure:

~~~sql
SELECT sku, name
FROM store.products
WHERE attributes @> '{"tags":["travel"]}'::jsonb;
~~~

A default GIN index supports common JSONB containment and key-existence searches:

~~~sql
CREATE INDEX products_attributes_gin_idx
ON store.products
USING gin (attributes);
~~~

An index adds storage and slows writes, so create one only when the workload uses the operators it supports. Confirm that PostgreSQL can use it with EXPLAIN on representative data.

## Update one JSON path

jsonb_set returns a new JSONB value with a path replaced. The final true argument creates the last path element when it is missing:

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

This update replaces the stored JSONB document value. Validate input shape and allowed keys in the application when those rules are part of the product model. Avoid putting relational constraints into unstructured documents.

## Review questions

1. What does JSONB store, and what textual details does it not preserve?
2. When is a regular relational column a better choice than JSONB?
3. What is the difference between -> and ->>?
4. What does the question-mark operator test?
5. What does the @> operator mean for JSONB?
6. What index type is commonly used for JSONB containment?
7. What does the final true argument to jsonb_set request?
8. Why should an application validate JSON document shape?

## References

- [PostgreSQL: JSON types](https://www.postgresql.org/docs/18/datatype-json.html)
- [PostgreSQL: JSON functions and operators](https://www.postgresql.org/docs/18/functions-json.html)
- [PostgreSQL: GIN indexes](https://www.postgresql.org/docs/18/gin.html)

---

| [Previous: Views and generated data](11-views-and-generated-data.md) | [Notes index](../README.md) | [Next: PL/pgSQL functions and triggers](13-plpgsql-functions-and-triggers.md) |
