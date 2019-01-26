Variables is where part of an sql string can be compiled in advance
with placeholders in it, only the missing pieces are produced at run
time and filled into placeholders. Variables is also the way to allow
dynamic queries because the values of such variables are evaluated at
run time.

To use variables, place a Clojure symbol inside any [Walkable
S-expression](s-expressions.md), which means in a pseudo column
definition, or an extra-condition constraint.

```clj
;; floor-plan
{:pseudo-columns {:person/age [:- 'current-year :person/yob]}}
```

with the above definition, the pseudo column `:person/age` will be the
value of the variable `current-year` minus the true column
`:person/yob`.

Of course, you must provide how that variable is computed:

``` clj
;; also in floor-plan
{:variable-getters {'current-year (fn [env] 2019)}
```

That's a boring function that returns a hard-coded value (`2019`), but
you can be more creative than I was. Do you remember the weird `env`
hashmap we've injected when we build the parser?
