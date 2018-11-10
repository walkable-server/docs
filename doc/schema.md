# Floor-plan

As you can see in **Usage** guide, we need to provide `floor-plan/compile-floor-plan` a map describing the `floor-plan`. You've got some idea about how such a `floor-plan` looks like as you glanced the example in **Overview** section. Now let's walk through each key of the `floor-plan` map in details.

{% hint style="info" %}
Notes:

* SQL snippets have been simplified for explanation purpose
* Backticks are used as quote marks
* For clarity, part of the floor-plan may not be included
{% endhint %}

## 1 :idents

Idents are the root of all queries. From an SQL dbms perspective, you must start your graph query from somewhere, and it's a table.

### 1.1 Keyword idents

Keyword idents can be defined as simple as:

```text
;; schema
{:idents {:people/all "person"}}
```

so queries like:

```text
[:people/all [:person/id :person/name]]
```

will result in an SQL query like:

```sql
SELECT `id`, `name` FROM `person`
```

### 1.2 Vector idents

These are idents whose key implies some condition. Instead of providing just the table, you provide the column \(as a namespaced keyword\) whose value match the ident argument found in the ident key.

For example, the following vector ident:

```text
;; dispatch-key: `:person/by-id`
;; ident arguments: `1`
[:person/by-id 1]
```

will require a floor-plan like:

```text
;; floor-plan
{:idents {:person/by-id :person/id}}
```

so queries like:

```text
;; query
[[:person/by-id 1] [:person/id :person/name]]
```

will result in an SQL query like:

```sql
SELECT `id`, `name` FROM `person` WHERE `person`.`id` = 1
```

## 2 :joins

Each join schema describes the "path" from the source table \(of the source entity\) to the target table \(and optionally the join table\).

Let's see some examples.

### Example 1: Join column living in source table

Assume table `cow` contains:

```text
|----+-------|
| id | color |
|----+-------|
| 10 | white |
| 20 | brown |
|----+-------|
```

and table `farmer` has:

```text
|----+------+--------|
| id | name | cow_id |
|----+------+--------|
|  1 | jon  |     10 |
|  2 | mary |     20 |
|----+------+--------|
```

and you want to get a farmer along with their cow using the query:

```text
;; query
[{[:farmer/by-id 1] [:farmer/name {:farmer/cow [:cow/id :cow/color]}]}]
```

> For the join `:farmer/cow`, table `farmer` is the source and table `cow` is the target.

then you must define the join "path" like this:

```text
;; floor-plan
{:joins {:farmer/cow [:farmer/cow-id :cow/id]}}
```

the above join path says: start with the value of column `farmer.cow_id` \(the join column\) then find the correspondent in the column `cow.id`.

Internally, Walkable will generate this query to fetch the entity whose ident is `[:farmer/by-id 1]`:

```sql
SELECT `farmer`.`name`, `farmer`.`cow_id` FROM `farmer` WHERE `farmer`.`id` = 1
```

the value of column `farmer`.`cow_id` will be collected \(for this example it's `10`\). Walkable will then build the query for the join `:farmer/cow`:

```sql
SELECT `cow`.`id`, `cow`.`color` FROM `cow` WHERE `cow`.`id` = 10
```

Finally, Walkable will combine the results of the above SQL queries and return the final result:

```text
{[:farmer/by-id 1] #:farmer{:number 1,
                            :name "jon",
                            :cow #:cow{:index 10,
                                       :color "white"}}}
```

### Example 2: A join involving a join table

Assume the following tables:

source table `person`:

```text
|----+------|
| id | name |
|----+------|
|  1 | jon  |
|  2 | mary |
|----+------|
```

target table `pet`:

```text
|----+--------|
| id | name   |
|----+--------|
| 10 | kitty  |
| 11 | maggie |
| 20 | ginger |
|----+--------|
```

join table `person_pet`:

```text
|-----------+--------+---------------|
| person_id | pet_id | adoption_year |
|-----------+--------+---------------|
|         1 |     10 |          2010 |
|         1 |     11 |          2011 |
|         2 |     20 |          2010 |
|-----------+--------+---------------|
```

you may query for a person and their pets along with their adoption year

```text
[{[:person/by-id 1] [:person/name {:person/pets [:pet/name :person-pet/adoption-year]}]}]
```

then the `:joins` part of our floor-plan is as simple as:

```text
;; schema
{:joins {:person/pets [:person/id :person-pet/person-id
                       :person-pet/pet-id :pet/id]}}
```

Walkable will issue an SQL query for `[:person/by-id 1]`:

```sql
SELECT `person`.`name` FROM `person` WHERE `person`.`id` = 1
```

and another query for the join `:person/pets`:

```sql
SELECT `pet`.`name`, `person_pet`.`adoption_year`
FROM `person_pet` JOIN `pet` ON `person_pet`.`pet_id` = `pet`.`id` WHERE `person_pet`.`person_id` = 1
```

and our not-so-atonishing result:

```text
;; result
{[:person/by-id 1] #:person{:id 1,
                            :name "jon",
                            :pets [{:pet/id 10,
                                    :pet/name "kitty"
                                    :person-pet/adoption-year 2010}
                                   {:pet/id 11,
                                    :pet/name "maggie"
                                    :person-pet/adoption-year 2011}]}}
```

### Example 3: Join column living in target table

No big deal. This is no more difficult than example 1.

Assume table `farmer` contains:

```text
|----+-------|
| id | name  |
|----+-------|
| 1  | jon   |
| 2  | mary  |
|----+-------|
```

and table `cow` has:

```text
|----+-------+----------|
| id | name  | owner_id |
|----+-------+----------|
| 10 | white |     1    |
| 20 | brown |     2    |
|----+-------+----------|
```

The `floor-plan` for this example can be a good exercise for the reader of this documentation. \(Sorry, actually I'm just too lazy to type it here :D \)

## 3 :reversed-joins

A handy way to avoid typing a join whose path is just reversed version of another.

The floor-plan for such a join is straightforward:

```text
;; floor-plan
{:joins          {:farmer/cow [:farmer/cow-id :cow/id]}
 :reversed-joins {:cow/owner :farmer/cow}}
```

so you can go both ways:

```text
;; queries:

;; - find the cow of a given farmer
[{[:farmer/by-id 1] [:farmer/name {:farmer/cow [:cow/id :cow/color]}]}]

;; - find the owner of a given cow
[{[:cow/by-id 10] [:cow/id :cow/color {:cow/owner [:farmer/name]}]}]
```

Also, another reason to use `:reversed-joins` is that it helps with semantics.

## 4 :columns

A set of available columns must be provided at compile time so Walkable can pre-compute part of SQL query strings.

```text
;; floor-plan
{:columns #{:farmer/name
            :cow/color}}
```

Walkable will automatically include columns found in `:joins` paths so you don't have to.

{% hint style="warning" %}
Please note: keywords not found in the column set will be ignored. That means if you forget to include any of them, you can't use the missing one in a query's property or filter.
{% endhint %}

{% hint style="info" %}
The rationale for having a pre-defined set of columns is that your query resolver doesn't have to limit itself to an SQL database as a single source of data. If Walkable can't match a keyword to a column, an ident or a join, it will be passed down to the next plugin in the Pathom plugin chain.
{% endhint %}

On the other hand, you don't have to include every single column in your database if you know you will never use some of them in a query's property or filter.

## 5 :cardinality

Idents and joins can have cardinality of either `:many` \(which is default\) or `:one`. You declare that by their dispatch keys:

```text
;; floor-plan
{:cardinality {:person/by-id :one
               ;; you can skip all `:many`!
               :people/all   :many}}
```

## 6 :extra-conditions

Please see documentation for [Filters](filters.md)

## 7 :emitter

Please see documentation for [Emitters](filters.md)

## 8 :required-columns

{% hint style="info" %}
You need to understand Pathom plugins to make use of this.
{% endhint %}

Automatically fetch some columns of the same level whenever a namespace keyword is asked for. This is useful when you want to derive a property from some SQL columns using Clojure code \(to be specific, as Pathom plugins\)

Please see [example.clj](https://github.com/walkable-server/walkable/tree/ab05c4706867ea7cce2daa6b903ee23834e1cf7f/dev/src/walkable_demo/handler/example.clj) for examples. Things to look at:

* `derive-attributes` which calculates `:pet/age` and `:person/age` from `:pet/yob` and `:person/yob` respectively.
* required inputs for `:pet/age` and `:person/age` in `:required-columns`:

```text
;; floor-plan
{:required-columns {:pet/age    #{:pet/yob}
                    :person/age #{:person/yob}}}
```

## 9 :pseudo-columns

Please see [dev.clj](https://github.com/walkable-server/walkable/tree/ab05c4706867ea7cce2daa6b903ee23834e1cf7f/dev/src/dev.clj) for examples.

