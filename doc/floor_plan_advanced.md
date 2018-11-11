# Floor-plan \(advanced\)

## 1. Syntactic sugars

### Composite key for `:idents`, `:extra-conditions`, `:joins` and `:cardinality`.

If two or more dispatch key share the same configuration, it's handy to have them in the same entry. For example:

instead of:

```text
;; floor-plan
{:idents {:people/all "person"
          :my-friends "person"}}
```

this is shorter:

```text
;; floor-plan
{:idents {[:people/all :my-friends]
          "person"}}
```

This also applies to keys under `:extra-conditions`, `:joins` and `:cardinality`.

## 2. Lambda form for `:extra-conditions`

Instead of specifying a fixed filters set for an ident or a join, you; can use a function that returns such filters set. The function accepts `env` as its argument.

Please see [dev.clj](https://github.com/walkable-server/walkable/tree/ab05c4706867ea7cce2daa6b903ee23834e1cf7f/dev/src/dev.clj) for examples.

