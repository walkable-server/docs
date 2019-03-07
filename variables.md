Variables is where part of an sql string can be compiled in advance
with placeholders in it, only the missing pieces are produced at run
time and filled into placeholders. Variables is also the way to allow
dynamic queries because the values of such variables are evaluated at
run time.

## Variable getters

To use variables, place a Clojure symbol inside any [Walkable
S-expression](s-expressions.md), which means in a [pseudo column
definition](s-expressions.md#pseudo-columns), an
[aggregator](aggregators.md) or an [extra-condition
constraint](filters.md#filters-in-extra-conditions-floor-plan).

```clojure
;; floor-plan
{:pseudo-columns {:person/age [:- 'current-year :person/yob]}}
```

with the above definition, the pseudo column `:person/age` will be the
value of the variable `current-year` minus the true column
`:person/yob`.

Of course, you must provide how that variable is computed:

```clojure
;; also in floor-plan
{:variable-getters
    [{:key 'current-year
      :fn  (fn [env] 2019)}]}
```

That's a boring function that returns a hard-coded value (`2019`), but
you can be more creative than I was. Do you remember the weird `env`
hashmap we've injected when we build the parser? Such getter functions
will be passed that `env` as its argument.

You can have as many variables in your expresions as you like, as long
as you tell Walkable how to compute all of them:


```clojure
;; also in floor-plan
{:variable-getters
    [{:key 'foo
      :fn  (fn [env] ...)}
     {:key 'bar
      :fn  (fn [env] ...)}
     ...]}
```

A variable's value can be cached throughout the request by the keyword
`:cached?`:

```clojure
;; floor-plan
{:variable-getters
    [{:key 'current-year
      :cached? true
      :fn  (fn [env] 2019)}]}
```

It's totally fine to use namespaced symbols for your variables:

```clojure
{:pseudo-columns {:person/age [:- 'app/current-year :person/yob]}}
```

## Variable getter graphs

Sometimes the computation of several variables share a step that you
don't want to repeat. You can pack those variables inside a
[Plumbing](https://github.com/plumatic/plumbing/) graph.

```clojure
(require '[plumbing.core :refer [fnk]])

;; floor-plan
{:variable-getter-graphs
  [{:graph {:x (fnk [env] ...)
            :y (fnk [env x] ...)
            :z (fnk [x y] ...)}]}
```

Please refer to Plumbing documentation for usage. Basically, for each
keyword in the graph provided, you will have an equivalant variable
(denoted as a Clojure symbol). For instance, for the above graph
you'll get three variables `'x`, `'y` and `'z`.

You can't use namespaced keywords in graphs. However, you can make the
equivalent symbols share a namespace. Specify a string for it:

```clojure
;; floor-plan
{:variable-getter-graphs
  [{:graph {:x (fnk [env] ...)
            :y (fnk [env x] ...)
            :z (fnk [x y] ...)}

    :namespace "foo"}]}
```

In this case, for the above graph you'll get three variables `'foo/x`,
`'foo/y` and `'foo/z`.

By default, all variables of the same graph will be computed once
whenever at least one of them gets mentioned. This graph's behavior is
called "eager evaluation". If the graph is big and computation is
expressive, you may want it to be "lazy" which means only required
variables get computed:

```clojure
;; floor-plan
{:variable-getter-graphs
  [{:graph {... ...}
    :lazy? true}]}
```

However, lazy evaluation is not available in Clojurescript version of plumbing.

Just like normal variable getters, each variable getter graph can be
cached with the `:cached?` keyword:

```clojure
;; floor-plan
{:variable-getter-graphs
  [{:graph {... ...}
    :cached? true}]}
```

As you can see, the value for `:variable-getter-graphs` is a
vector. You can have as many such graphs as you please:

```clojure
;; floor-plan
{:variable-getter-graphs
  [{:graph {... ...}
    :lazy? true
    :cached? true}
   {:graph {... ...}
    :lazy? true}
   {:graph {... ...}
    :cached? true}
   ...]}
```
