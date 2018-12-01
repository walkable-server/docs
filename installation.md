# Installation

## Project dependencies

```clojure
[walkable "1.1.0-rc1"]

```

Walkable is a plugin for [Pathom](https://github.com/wilkerlucio/pathom/).

{% hint style="info" %}
Pathom is a Clojure library designed to provide a collection of helper functions to support Clojure\(script\) graph parsers using om.next graph syntax.

Don't worry if you don't know how Pathom works yet: Understanding Pathom is not required unless you use advanced features.
{% endhint %}

Walkable comes with two versions: synchronous \(for Clojure\) and asynchronous \(for both Clojure and Clojurescript\).

First of all, you need to build pathom "parser" with walkable's `sqb/pull-entities` \(or `sqb/async-pull-entities`\)

```clojure
(require '[com.wsscode.pathom.core :as p])
(require '[walkable.sql-query-builder :as sqb])

```

{% mdtabs title="Sync version" %}

```clojure
(def sync-parser
  (p/parser
    {::p/plugins
     [(p/env-plugin
        {::p/reader
        ;; walkable's main worker
         [sqb/pull-entities
         ;; pathom's entity reader
          p/map-reader]})]}))
```
{% mdtab title="Async version" %}

``` clojure
(def async-parser
  (p/async-parser
    {::p/plugins
     [(p/env-plugin
        {::p/reader
        ;; walkable's main worker
         [sqb/async-pull-entities
         ;; pathom's entity reader
          p/map-reader]})]}))
```
{% endmdtabs %}

Then you need to define your floor-plan and compile it

```clojure
;; both sync and async versions
(require '[walkable.sql-query-builder.floor-plan :as floor-plan])
(def compiled-floor-plan
  (floor-plan/compile-floor-plan
    {:emitter     ...
     :columns     ...
     :idents      ...
     :joins       ...
     ...          ...}))
```

Details about the floor-plan is [here](https://walkable.gitbook.io/walkable/floor-plan).

Ready! It's time to run your graph queries:

Sync version:

```clojure
(require '[clojure.java.jdbc :as jdbc])

(let [my-query     [{:people/all [:person/name]}]
      my-db        {:dbtype   "mysql"
                    :dbname   "clojure_test"
                    :user     "test_user"
                    :password "test_password"}
      my-run-query jdbc/query]
  (sync-parser {::sqb/sql-db     my-db
                ::sqb/run-query  my-run-query
                ::sqb/floor-plan compiled-floor-plan}
               my-query))
```

where `my-run-query` and `my-db` is any pair of a function plus a database instance \(even a pair of mock ones!\) that work together like this:

```clojure
(my-run-query my-db ["select * from fruit where color = ?" "red"])
;; => [{:id 1, :color "red"} {:id 3, :color "red"} ...]

(my-run-query my-db ["select * from fruit"])
;; => [{:id 1, :color "red"} {:id 2, :color "blue"} ...]
```

Async version, Clojure JVM:

```clojure
(require '[clojure.java.jdbc :as jdbc])
(require '[clojure.core.async :refer [go promise-chan put! >! <!]])

(let [my-query     [{:people/all [:person/name]}]
      my-db        {:dbtype   "mysql"
                    :dbname   "clojure_test"
                    :user     "test_user"
                    :password "test_password"}
      my-run-query (fn [db q]
                     (let [ch (promise-chan)]
                       (let [result (jdbc/query db q)]
                         (put! ch rersult))
                       ch))]
  (go
    (println
      (<! (async-parser
            {::sqb/sql-db     my-db
             ::sqb/run-query  my-run-query
             ::sqb/floor-plan compiled-floor-plan}
            my-query)))))
```

As you can see, `my-run-query` and `my-db` are similar to those in sync version, except that `my-run-query` doesn't return the result directly but in a channel.

For Nodejs, you'll need to convert between Javascript and Clojure data structure. The file [dev.cljs](https://github.com/walkable-server/walkable/blob/master/dev/src/common/dev.cljs) has examples using sqlite3 node module.

{% hint style="info" %}
Please see the file dev.clj \(or its nodejs version dev.cljs\) for executable examples. Consult config.edn for SQL migrations for those examples.
{% endhint %}
