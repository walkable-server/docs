# Pagination

Walkable supports SQL pagination you've been familiar with: `OFFSET`,
`LIMIT` and `ORDER BY`.

## Inside a query's parameters

You can specify how to paginate your query by providing any
combination of pagination keywords, namely: `:offset`, `:limit`,
`:order-by`.

* No pagination:
{% mdtabs title="Query" %}
```clojure
`[{(:farmers/all {})
   [:farmer/number :farmer/name]}]]
;; the same as:
`[{:farmers/all
   [:farmer/number :farmer/name]}]]
```
{% mdtab title="SQL output" %}
```sql
SELECT farmer.number, farmer.name FROM farmer
```
{% endmdtabs %}

* Pagination with `:limit` and `:order-by`:
{% mdtabs title="Query" %}
```clojure
`[{(:farmers/all {:limit 3 :order-by :farmer/number})
   [:farmer/number :farmer/name]}]]
```
{% mdtab title="SQL output" %}
```sql
SELECT farmer.number, farmer.name
 FROM farmer
 LIMIT 3
 ORDER BY farmer.number
```
{% endmdtabs %}

* Who says `:order-by` has to be simple?
{% mdtabs title="Query" %}
```clojure
`[{(:farmers/all {:order-by [:farmer/number :farmer/name :desc :nils-first]})
   [:farmer/number :farmer/name]}]]
```
{% mdtab title="SQL output" %}
```sql
SELECT farmer.number, farmer.name
FROM farmer
ORDER BY farmer.number, farmer.name DESC NULLS FIRST
```
{% endmdtabs %}

## Validator and default value

Pagination can be a bomb if you let client apps specify whatever
parameters. You can just tell Walkable how pagination parameters must
comply and default values when provided parameters fail that (or not
even provided at all).

Validators and respective default values can be declared for each
idents it in `:floor-plan`.

### Offset and limit

For `:offset` and `:limit`, value of `:default` must be a number while
`:validate` is a function that check if the supplied parameter
satisfies your constraint.

{% mdtabs title="Floor-plan" %}
```clojure
{:idents
 {:farmers/all "farmer"}

 :pagination-fallbacks
 {:farmers/all
  {:offset {:default  2
            :validate #(<= 0 % 100)}}}}
```
{% mdtab title="Query" %}
```clojure
`[{(:farmers/all {:offset 9999})
   [:farmer/number :farmer/name]}]]
```
{% mdtab title="SQL output" %}

Will use default value `2` because `9999` fails the validator:

```sql
SELECT farmer.number, farmer.name
FROM farmer
OFFSET 2
```
{% endmdtabs %}

{% hint style="info" %}
Walkable will check if the supplied argument is an integer first, so
you don't have to do it in your validator functions.
{% endhint %}

### Order-by

For `:order-by`, value of `:default` must be a valid order-by expression while
`:validate` is a function that check if all the supplied **columns**
satisfies your constraint. Usually you want a simple set for your validator.

{% mdtabs title="Floor-plan" %}
```clojure
{:idents
 {:farmers/all "farmer"}

 :pagination-fallbacks
 {:farmers/all
  {:order-by {:default  [:farmer/number :asc]
              :validate #{:farmer/number :farmer/yob}}}}}
```
{% mdtab title="Query" %}
```clojure
`[{(:farmers/all {:order-by [:farmer/number :desc :farmer/yob :nils-first]})
   [:farmer/number :farmer/name]}]]
```
{% mdtab title="SQL output" %}

Supplied columns in `:order-by` satisfy the validator.

```sql
SELECT farmer.number, farmer.name
FROM farmer
ORDER BY farmer.number DESC, farmer.yob NULLS FIRST
```
{% endmdtabs %}

{% mdtabs title="Floor-plan" %}
```clojure
{:idents
 {:farmers/all "farmer"}

 :pagination-fallbacks
 {:farmers/all
  {:order-by {:default  [:farmer/number :asc]
              :validate #{:farmer/number :farmer/yob}}}}}
```
{% mdtab title="Query" %}
```clojure
`[{(:farmers/all {:order-by [:farmer/number :desc :farmer/name :nils-first]})
   [:farmer/number :farmer/name]}]]
```
{% mdtab title="SQL output" %}

The column `:farmer/name` in `:order-by` fails the validator. Default
value will be used:

```sql
SELECT farmer.number, farmer.name
FROM farmer
ORDER BY farmer.number ASC
```
{% endmdtabs %}
