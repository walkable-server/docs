S-expressions is the way Walkable allow you to write arbitrary SQL
expressions in your
[paredit](https://github.com/clojure-emacs/cider)/[parinfer](https://github.com/shaunlebron/parinfer)-[powered](https://github.com/tpope/vim-fireplace)
[editors](https://cursive-ide.com/) without compromising security.

{% hint style="info" %}

Note about SQL examples:

* S-expressions can end up as SQL strings in either `SELECT`
  statements or `WHERE` conditions. For demonstrating purpose, the
  strings are wrapped in `SELECT ... as q` so the SQL outputs are
  executable, except ones with tables and columns.
* SQL output may differ when you `require` different implementations
  \(ie `(require 'walkable.sql-query-builder.impl.postgres)` vs
  `(require 'walkable.sql-query-builder.impl.sqlite)`\).

{% endhint %}

## Primitive types

{% mdtabs title="S-expression" %}
``` clojure
123
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT 123 AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q 123}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
nil
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT NULL AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q nil}]

```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
"hello world"
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT ? AS q" "hello world"])
```
{% mdtab title="result" %}
``` clojure
[{:q "hello world"}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
"hello\"; DROP TABLE users"
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT ? AS q" "hello\"; DROP TABLE users"])
```
{% mdtab title="result" %}
``` clojure
[{:q "hello\"; DROP TABLE users"}]
```
{% endmdtabs %}

## Columns

{% hint style="info" %}
Note

The examples just use backticks as quote marks. Depending on your
emitter configuration, Walkable will emit SQL strings using whatever
quote marks you specified.

{% endhint %}

{% mdtabs title="S-expression" %}
``` clojure
:my-table/a-column
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT `my_table`.`a_column` AS `my-table/a-column` FROM `my_table`"])
```
{% mdtab title="result" %}
``` clojure
[{:my-table/a-column 42}, ...other records...]

```
{% endmdtabs %}

## Comparison

Walkable comes with some comparison operators: `:=`, `:<`, `:>`,
`:<=`, `:>=`. They will result in SQL operators with the same name,
but also handle multiple arity mimicking their Clojure equivalents.

{% mdtabs title="S-expression" %}
``` clojure
[:= 1 2]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (1 = 2) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q false}]

```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:<= 1 2]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (1 <= 2) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q true}]

```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:< 1 2 3 1]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT ((1 < 2) AND (2 < 3) AND (3 < 1)) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q false}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:= 0]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT true AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q true}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:>= 1000]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT true AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q true}]
```
{% endmdtabs %}

String comparison operators: `=`, `like`, `match`, `glob`:

{% mdtabs title="S-expression" %}
``` clojure
[:= "hello" "hi"]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (? = ?) AS q" "hello" "hi"])
```
{% mdtab title="result" %}
``` clojure
[{:q false}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:like "abcd" "abc%"]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (? LIKE ?) AS q" "abcd" "abc%"])
```
{% mdtab title="result" %}
``` clojure
[{:q true}]
```
{% endmdtabs %}

Use them on some columns, too:

{% mdtabs title="S-expression" %}
``` clojure
[:= :my-table/its-column "hi"]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (`my_table`.`its_column` = ?) AS q FROM `my_table`" "hi"])
```
{% mdtab title="result" %}
``` clojure
[{:q true}]
```
{% endmdtabs %}

## Math

Basic math operators work just like their Clojure equivalents: `:+`, `:-`, `:*`, `:/`:

{% mdtabs title="S-expression" %}
``` clojure
[:+ 1 2 4 8]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (1 + 2 + 4 + 8) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q 15}]
```
{% endmdtabs %}

Feel free to mix them:

{% mdtabs title="S-expression" %}
``` clojure
[:+ [:*] [:* 2 4 7] [:/ 0.25]]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (1 + (2 * 4 * 7) + (1/0.25)) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q 61.0}]
```
{% endmdtabs %}

{% hint style="info" %}
`:*` with no argument result in `1`
{% endhint %}

## String manipulation

{% mdtabs title="S-expression" %}
``` clojure
[:str "hello " nil "world" 123]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (CONCAT(?, NULL, ?, 123) AS q" "hello " "world"])
```
{% mdtab title="result" %}
``` clojure
[{:q "hello world123"}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:subs "hello world"]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (CONCAT(?, NULL, ?, 123) AS q" "hello " "world"])
```
{% mdtab title="result" %}
``` clojure
[{:q "hello world123"}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:str "hello " nil "world" 123]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (CONCAT(?, NULL, ?, 123) AS q" "hello " "world"])
```
{% mdtab title="result" %}
``` clojure
[{:q "hello world123"}]
```
{% endmdtabs %}

## Conversion between types

Use the `:cast` operator:

{% mdtabs title="S-expression" %}
``` clojure
[:cast "2" :integer]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT CAST(? as INTEGER) AS q" "2"])
```
{% mdtab title="result" %}
``` clojure
[{:q 2}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:cast 3 :text]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT CAST(3 as TEXT) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q "3"}]
```
{% endmdtabs %}

## Logic constructs

`:and` and `:or` accept many arguments like in Clojure:

{% mdtabs title="S-expression" %}
``` clojure
[:and true true false]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (true AND true AND false) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q false}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:and]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (true) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q true}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:or]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (NULL) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q nil}]
```
{% endmdtabs %}

`:not` accepts exactly one argument:

{% mdtabs title="S-expression" %}
``` clojure
[:not true]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (NOT true) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q false}]
```
{% endmdtabs %}

Party time! Mix them as you wish:

{% mdtabs title="S-expression" %}
``` clojure
[:and [:= 4 [:* 2 2]] [:not [:> 1 2]] [:or [:= 2 3] [:= 4 4]]]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (((4)=((2)*(2))) AND (NOT ((1)>(2))) AND (((2)=(3)) OR ((4)=(4)))) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q true}]
```
{% endmdtabs %}

## Notes
Please note that Walkable S-expressions are translated directly to SQL
equivalent. Your DBMS may throw an exception if you ask for this:

{% mdtabs title="S-expression" %}
``` clojure
[:or 2 true]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (2 OR true) AS q"])
```
{% mdtab title="result" %}
``` clojure
ERROR:  argument of OR must be type boolean, not type integer
```
{% endmdtabs %}

Don't be surprised if you see `[:not nil]` is ... `nil`!

{% mdtabs title="S-expression" %}
``` clojure
[:not nil]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (NOT NULL) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q nil}]
```
{% endmdtabs %}

`nil` can not be checked with `:=`. Use `:nil?` instead

{% mdtabs title="S-expression" %}
``` clojure
[:= nil nil]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (NULL = NULL) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q nil}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:nil? nil]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (NULL IS NULL) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q true}]
```
{% endmdtabs %}

## Other constructs

`:when`, `:if`, `:case` and `:cond` look like in Clojure...

{% mdtabs title="S-expression" %}
``` clojure
[:when true "yay"] ;; or [:if true "yay"]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (CASE WHEN ( true ) THEN ( ? ) END) AS q" "yay"])
```
{% mdtab title="result" %}
``` clojure
[{:q "yay"}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:if [:= 1 2] "yep" "nope"]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (CASE WHEN ((1)=(2)) THEN ( ? ) ELSE ( ? ) END) AS q" "yay" "nope"])
```
{% mdtab title="result" %}
``` clojure
[{:q "nope"}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:case [:+ 0 1] 2 3]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (CASE (0+1) WHEN (2) THEN (3) END) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q nil}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:case [:+ 0 1] 2 3 4]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (CASE (0+1) WHEN (2) THEN (3) ELSE (4) END) AS q"])
```
{% mdtab title="result" %}
``` clojure
[{:q 4}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:cond [:= 0 1] "wrong" [:< 2 3] "right"]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT  (CASE WHEN ((0)=(1)) THEN ( ? ) WHEN ((2)<(3)) THEN ( ? ) END) AS q" "wrong" "right"])
```
{% mdtab title="result" %}
``` clojure
[{:q "right"}]
```
{% endmdtabs %}

...except the fact that you must supply real booleans to them, not just some truthy values.

{% mdtabs title="S-expression" %}
``` clojure
[:cond
 [:= 0 1]
 "wrong"

 [:> 2 3]
 "wrong again"

 true ;; <= must be literally `true`, not `:default` or something else
 "default"]

```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT  (CASE WHEN ((0)=(1)) THEN ( ? ) WHEN ((2)>(3)) THEN ( ? ) WHEN ( true ) THEN ( ? ) END) AS q" "wrong" "wrong again" "default"])
```
{% mdtab title="result" %}
``` clojure
[{:q "default"}]
```
{% endmdtabs %}

## Pseudo columns

In your floor-plan you can define so-called pseudo columns that look just like normal columns from client-side view:

```clojure
;; floor-plan
;; :person/yob is a real column
{:pseudo-columns {:person/age [:- 2018 :person/yob]}}
```

You can't tell the difference from client-side:

{% mdtabs title="Query for a real column" %}
``` clojure
[{[:person/by-id 9]
  [:person/yob]}]
```
{% mdtab title="Query for a pseudo column" %}
``` clojure
[{[:person/by-id 9]
  [:person/age]}]
```
{% endmdtabs %}

{% mdtabs title="Filter with a real column" %}
``` clojure
[{(:people/all {:filters [:= 1988 :person/yob]})
  [:person/name]}]
```
{% mdtab title="Filter with a pseudo column" %}
``` clojure
[{(:people/all {:filters [:= 30 :person/age]})
  [:person/name]}]
```
{% endmdtabs %}

Behind the scenes, Walkable will expand the pseudo columns to whatever
they are defined. You can also use pseudo columns in other pseudo
columns' definition, but be careful as Walkable **won't check circular
dependencies** for you.

Please note you can only use true columns from the same table in the
definition of pseudo columns. For instance, the following doesn't make
sense:

```clojure
;; floor-plan
{:pseudo-columns {:person/age [:- 2018 :pet/yob]}}
```

Your RDMS will throw an exception in that case anyway.

## Define your own operators

There are some convenient marcros to help you "import" SQL functions/operators: `walkable.sql-query-builder.expressions/import-functions` and `walkable.sql-query-builder.expressions/import-infix-operators`.

More complex operators may require implementing multimethod `walkable.sql-query-builder.expressions/process-operator` or even a harder one `walkable.sql-query-builder.expressions/process-unsafe-expression`.

Todo: more docs.

## Bonus: JSON in Postgresql

The following expressions work in Postgresql:

{% mdtabs title="S-expression" %}
``` clojure
[:= 1
 [:cast [:get-as-text [:jsonb {:a 1}] "a"] :integer]]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT ((1)=(CAST ((?::jsonb)->>( ? ) AS INTEGER))) AS q" "{\"a\" :1}" "a"])
```
{% mdtab title="result" %}
``` clojure
[{:q true}]
```
{% endmdtabs %}

{% mdtabs title="S-expression" %}
``` clojure
[:or [:= 2 [:array-length [:array 1 2 3 4] 1]]
 [:contains [:jsonb {:a 1 :b 2}]
  [:jsonb {:a 1}]]
 [:jsonb-exists [:jsonb {:a 1 :b 2}]
  "a"]]
```
{% mdtab title="SQL output" %}
``` clojure
(jdbc/query your-db ["SELECT (((2)=(array_length (ARRAY[1, 2, 3, 4], 1)))
OR ((?::jsonb)@>(?::jsonb))
OR (jsonb_exists (?::jsonb,  ? )))
AS q"
    "{\"a\":1,\"b\":2}" "{\"a\":1}" "{\"a\":1,\"b\":2}" "a"])
```
{% mdtab title="result" %}
``` clojure
[{:q true}]
```
{% endmdtabs %}
