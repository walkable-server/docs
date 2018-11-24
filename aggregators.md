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
Count number of all farmers whose name starting with "a":

```clojure
`[(:farmers/total {:filters [:like :farmer/name "a%"]})]
```
{% mdtab title="SQL output" %}
Parameterized query:
```sql
SELECT COUNT(*) AS `farmers/total` FROM `farmer`
WHERE `farmer`.`name` LIKE ?
```
parameters:
```clojure
["a%"]
```
{% endmdtabs %}

## Aggregators against idents vs aggregators against joins

In the above examples, aggregators are used against idents. How
about aggregators with joins?

{% mdtabs title="Floor-plan" %}
```clojure
{:idents
 {:person/by-id :person/id}
 :joins
 {:person/pet-count ;; the join path you would use with a normal join:
  [:person/id :person-pet/person-id :person-pet/pet-id :pet/id]}
 :aggregators
 {:person/pet-count [:count-*]}}
```

{% mdtab title="Query" %}
```clojure
[{[:person/by-id 1]
  [:person/id
   :person/name
   :person/pet-count]}]
```

{% mdtab title="Data" %}
Table "person":

| id | name |
| :---: | :--- |
| 1 | Mary |
| 2 | John |

Table "pet":

| id | name |
| :--- | :--- |
| 10 | Tom |
| 20 | Jerry |

Table "person\_pet":

| person\_id | pet\_id |
| :---: | :---: |
| 1 | 10 |
| 1 | 20 |


{% mdtab title="Result" %}
```clojure
{[:person/by-id 1]
 {:person/id        1
  :person/name      "Mary"
  :person/pet-count 2}}
```

{% endmdtabs %}

## Common SQL aggregators

`:avg`, `:max`, `:min`, `:count`, `:count-*`

## Aggregators vs Pseudo-columns

An aggregator is meant to produce a single value, while pseudo columns
are for adding one or more columns to all rows.

For instance, if you want the value of:

```sql
max (`table_a`.`column_x`)
```

you will need an aggregator:

``` clojure
;; floor-plan
{:aggregators {:your/aggregator [:max :table-a/column-x]}}
```

In contrast, if you want the value of:

```sql
max (`table_a`.`column_x`, `table_a`.`column_y`)
```

you will need a pseudo-column:

``` clojure
;; floor-plan
{:pseudo-columns {:your/other-column [:max :table-a/column-x :table-a/column-y]}}
```

## More freedom with S-expressions

You're free to use whatever s-expression in your aggregator, as long
as the final result returned by SQL is a single value.

``` clojure
;; floor-plan
{:aggregators {:your/aggregator [:/ [:+ [:* 10 [:max :table-a/column-x]]
                                        [:max :table-a/column-y]]
                                     2]}}
```
