## 1. Syntactic sugars

### Composite key for `:idents`, `:extra-conditions`, `:joins` and `:cardinality`.

If two or more dispatch key share the same configuration, it's handy to have them in the same entry. For example:

instead of:

```clojure
;; floor-plan
{:idents {:people/all "person"
          :my-friends "person"}}
```

this is shorter:

```clojure
;; floor-plan
{:idents {[:people/all :my-friends]
          "person"}}
```

This also applies to keys under `:extra-conditions`, `:joins` and `:cardinality`.

## 2 :required-columns

{% hint style="info" %}
You need to understand Pathom plugins to make use of this.
{% endhint %}

Automatically fetch some columns of the same level whenever a namespace keyword is asked for. This is useful when you want to derive a property from some SQL columns using Clojure code \(to be specific, as Pathom plugins\)

Please see [example.clj](https://github.com/walkable-server/walkable/tree/ab05c4706867ea7cce2daa6b903ee23834e1cf7f/dev/src/walkable_demo/handler/example.clj) for examples. Things to look at:

* `derive-attributes` which calculates `:pet/age` and `:person/age` from `:pet/yob` and `:person/yob` respectively.
* required inputs for `:pet/age` and `:person/age` in `:required-columns`:

```clojure
;; floor-plan
{:required-columns {:pet/age    #{:pet/yob}
                    :person/age #{:person/yob}}}
```
