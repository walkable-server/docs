---
description: >-
  Emitter is part of floor-plan that tells Walkable how to generate SQL strings
  in respect of your database.
---

# Emitter

Emitter is a `hash-map` with the following keys `:quote-marks`, `:transform-table-name`,  `:transform-column-name`, `:rename-tables`, `:rename-columns`, `:rename-keywords`, `:wrap-select-strings`.

There are some pre-defined emitters in `walkable.sql-query-builder.emitter` namespace, namely `sqlite-emitter`, `mysql-emitter` and `postgres-emitter`. You may make your own emitter from scratch \(which means providing all the above keywords\), but it's better to start from one of the pre-defined emitters and override your custom keys.

### :quote-marks

Different SQL databases use different strings to denote quotation.

For instance, MySQL use a pair of backticks:

```sql
SELECT * FROM `table`
```

while PostgreSQL use quotation marks:

```sql
SELECT * FROM "table"
```

You need to provide the `:quote-marks` as a vector of two strings

{% tabs %}
{% tab title="Postgres" %}
```text
;; emitter for postgres
{:quote-marks ["\"", "\""]}
```
{% endtab %}

{% tab title="Mysql" %}
```text
;; emitter for mysql
{:quote-marks ["`", "`"]}
```
{% endtab %}
{% endtabs %}

### :transform-table-name and :transform-column-name

### :rename-tables, :rename-columns

### :rename-keywords

### :wrap-select-strings

```text

```



