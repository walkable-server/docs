# Aggregators

Sometimes you don't want columns of a table but the total number of
records in it. Use `:aggregators` in that case.

{% mdtabs title="Floor plan" %}
```clojure
{:idents
 {:farmers/total "farmer"}
 :aggregators
 {:farmers/total [:count-*]}}
```
{% mdtab title="Query" %}
```clojure
[:farmers/total]
```
{% mdtab title="SQL output" %}
```sql
SELECT COUNT(*) AS `farmers/total`  FROM `farmer`
```
{% endmdtabs %}

## With filters and pagination

Aggregators work well with filters (and pagination, too!)

{% mdtabs title="Floor plan" %}
```clojure
{:idents
 {:farmers/total "farmer"}
 :aggregators
 {:farmers/total [:count-*]}}
```
{% mdtab title="Query" %}
```clojure
`[(:farmers/total {:filters [:= :farmer/name "mary"]})]
```
{% mdtab title="SQL output" %}
```sql
SELECT COUNT(*) AS `farmers/total`  FROM `farmer`
WHERE `farmer`.`name` = ?
```
parameters:
```clojure
["mary"]
```
{% endmdtabs %}

## Aggregators against idents vs aggregators against joins

In the above examples, aggregators are used against idents. How
about aggregators with joins?

{% mdtabs title="Floor plan" %}
```clojure
{:idents           {:world/all   "human"}
 :joins            {:human/follower-count
                    [:human/number :follow/human-1 :follow/human-2 :human/number]}
 :aggregators      {:human/follower-count [:count-*]}}
```
{% mdtab title="Query" %}
```clojure
[{:world/all [:human/name :human/follower-count]}]
```
{% mdtab title="Data" %}
Todo
{% mdtab title="Result" %}
```clojure
;; Todo
```
{% endmdtabs %}

## Common SQL aggregators

`:avg`, `:max`, `:min`, `:count`, `:count-*`

## More freedom with S-expressions

## Aggregators vs Pseudo-columns

Aggregators are for single values, while pseudo columns are for adding
one or more columns to all rows.
