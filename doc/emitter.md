---
description: >-
  Emitter is part of floor-plan that tells Walkable how to generate SQL strings
  in respect of your database.
---

# Emitter

Emitter is a `hash-map` with the following keys `:quote-marks`, `:transform-table-name`, `:transform-column-name`, `:rename-tables`, `:rename-columns`, `:rename-keywords`, `:wrap-select-strings`.

## Pre-defined emitters

There are some pre-defined emitters in `walkable.sql-query-builder.emitter` namespace, namely `sqlite-emitter`, `mysql-emitter` and `postgres-emitter`. You may make your own emitter from scratch \(which means providing all the above keywords\), but it's better to start from one of the pre-defined emitters and override your custom keys:

```text
(def your-emitter (merge emitter/postgres-emitter {...your customizations...}))
```

## :quote-marks

Different SQL databases use different strings to denote quotation.

For instance, MySQL use a pair of backticks:

```sql
SELECT * FROM `table`
```

while PostgreSQL use quotation marks:

```sql
SELECT * FROM "table"
```

You need to provide the `:quote-marks` as a vector of two strings. For example:

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

{% hint style="warning" %}
It's unlikely that you want to override `:quote-marks`. Keep the one from pre-defined emitters.
{% endhint %}

## :transform-table-name and :transform-column-name

By default the pre-defined emitters will replace dashes `-` with underscores `_`in table names and column names. For example, the column `:human-being/full-name` will be understood as column `full_name` in table `human_being`. You can override that behavior by providing your own function\(s\).

For example if all your real table names are upper-cased \(while we all like our keywords being lower-cased\):

{% tabs %}
{% tab title="Emitter" %}
```text
(def your-emitter
  (merge emitter/postgres-emitter
    {:transform-table-name (fn [table-name] (clojure.string/upper-case table-name))}))
```
{% endtab %}

{% tab title="Keyword" %}
`:person/name`, `:person/age`
{% endtab %}

{% tab title="Real table in database" %}
"PERSON", as in this sql query:

```text
SELECT "PERSON"."name", "PERSON"."age" FROM "PERSON";
```
{% endtab %}
{% endtabs %}

## :rename-tables, :rename-columns

Sometimes you want to specify the exact name, let alone `transform-*-name` functions:

{% tabs %}
{% tab title="Emitter" %}
```text
(def your-emitter
  (merge emitter/postgres-emitter
    {:rename-tables {"person" "people"}}))
```
{% endtab %}

{% tab title="Keyword" %}
`:person/name`, `:person/age`
{% endtab %}

{% tab title="Real table in database" %}
"people", as in this sql query:

```text
SELECT "people"."name", "people"."age" FROM "people";
```
{% endtab %}
{% endtabs %}

## :rename-keywords

Maybe matching just the table name or the column name is not enough. You want to match the exact pair of table + column name.

{% tabs %}
{% tab title="Emitter" %}
```text
(def your-emitter
  (merge emitter/postgres-emitter
    {:rename-keywords {:person/type :person/typo}}))
```
{% endtab %}

{% tab title="Keywords" %}
`:person/name`, `:person/type`
{% endtab %}

{% tab title="Real table+column in database" %}
"person"."typo", as in this sql query:

```text
SELECT "person"."name", "person"."typo" FROM "person";
```
{% endtab %}
{% endtabs %}

## :wrap-select-strings

Specific to Sqlite. It's unlikely that you want to override this. \(Doc comming soon\)

