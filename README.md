# Introduction

|  |  |
| :--- | :--- |
| ![](.gitbook/assets/walkable%20%281%29.png) | **Walkable** is a serious way to fetch data from SQL for Clojure: Datomic® pull \(Graphql-ish\) syntax, Clojure flavored filtering and more. |

> Data dominates. If you’ve chosen the right data structures and organized things well, the algorithms will almost always be self-evident. Data structures, not algorithms, are central to programming. -- Rob Pike
>
> Bad programmers worry about the code. Good programmers worry about data structures and their relationships. -- Linus Torvalds

Ever imagined sending queries like this to your SQL database?

{% tabs %}
{% tab title="Query" %}
```text
[{[:person/by-id 1]
  [:person/id
   :person/name
   :person/age
   {:person/pet [:pet/name :pet/favorite-location]}]}]
```
{% endtab %}

{% tab title="Data" %}
### Table "person":

| id | name | age |
| :---: | :--- | :---: |
| 1 | Mary | 20 |
| 2 | John | 15 |

### Table "pet":

| id | name | favorite\_location |
| :--- | :--- | :--- |
| 10 | Tom | garden |
| 20 | Jerry | kitchen |

### Table "person\_pet":

| person\_id | pet\_id |
| :---: | :---: |
| 1 | 10 |
| 2 | 20 |
{% endtab %}

{% tab title="Result" %}
```text
{[:person/by-id 1]
 {:person/id   1
  :person/name "Mary"
  :person/age  20
  {:person/pet {:pet/name              "Tom"
                :pet/favorite-location "garden"}}}}
```
{% endtab %}
{% endtabs %}

or a bit more sophisticated:

{% tabs %}
{% tab title="Query" %}
```text
`[{(:articles/all {:filters [:and [:= false :article/hidden]
                             {:article/author [:= :user/username "lucy"]}]})
   [:article/title
    :article/created-date
    {:article/author [:user/id :user/username :user/karma]}]}]
```
{% endtab %}

{% tab title="Data" %}
### Table "article":

| id | title | author\_id | hidden | created\_date |
| :---: | :--- | :---: | :---: | :--- |
| 1 | Hello world | 10 | false | 2018-10-11 |
| 2 | Welcome | 20 | false | 2018-11-10 |
| 3 | Unfinished | 20 | true | 2018-09-20 |

### Table "user":

| id | username | karma |
| :---: | :--- | :---: |
| 10 | mark | 21 |
| 20 | lucy | 42 |
{% endtab %}

{% tab title="Result" %}
```text
[{:articles/all
  [{:article/title        "Welcome"
    :article/created-date "2018-11-10"
    {:article/author {:user/id       20
                      :user/username "lucy"
                      :user/karma    42}}}]}]
```
{% endtab %}
{% endtabs %}

Yes, you can. Have your data fetched in your Clojure mission critical app with confidence. Even more, build the query part of a fulcro server or REST api in minutes today! Call it from your Clojurescript app without worrying about SQL injection.

You can learn about the above query language [here](doc/query_language.md)

{% hint style="info" %}
### **Walkable is NOT about om.next**

People may have the impression that Walkable \(and Pathom\) is specific to om.next. That is NOT the case! Walkable requires a query language that is expressive and based off data structure. Om.next query language happens to satisfy that.

Walkable's goal is to become the ultimate SQL library for Clojure.
{% endhint %}

{% hint style="warning" %}
### For plain-SQL enthusiasts or ORM fans

I urge you to challenge your assumptions by implementing your own version of the Realworld API Spec with your favorite ORM \(Django, Rails, Korma, Toucan, whatever\) and compare it with the implementation using Walkable [here](https://github.com/walkable-server/realworld/).
{% endhint %}

## Overview

Basically you define your floor-plan like this:

```text
{:idents           { ;; query with `[:person/by-id 1]` will result in
                    ;; `FROM person WHERE person.id = 1`
                    :person/by-id :person/id
                    ;; just select from `person` table without any constraints
                    :people/all "person"}
 :columns          #{:person/name :person/yob}
 :extra-conditions { ;; enforce some constraints whenever this join is asked for
                    :pet/owner [:and
                                [:= :person/hidden true]
                                ;; yes, you can nest the conditions whatever you like
                                [:or [:= :person/id 5]
                                 ;; a vector implies an `AND` for the conditions inside
                                     [[:in :person/yob
                                       1970
                                       1971
                                       ;; yes, you can!
                                       [:cast "1972" :integer]]
                                      [:like :person/name "john"]]
                                 ;; you can even filter by properties of a join, not just
                                 ;; the item itself
                                 {:person/pet [:or [:= :pet/color "white"]
                                                   [:= :pet/color "green"]]}
                                 ]]}
 :joins            { ;; will produce:
                    ;; "JOIN `person_pet` ON `person`.`id` = `person_pet`.`person_id` JOIN `pet` ON `person_pet`.`pet_id` = `pet`.`id`"
                    :person/pet [:person/id :person-pet/person-id :person-pet/pet-id :pet/id]
                    ;; will produce
                    ;; "JOIN `person` ON `pet`.``owner` = `person`.`id`"
                    :pet/owner [:pet/owner :person/id]}
 :cardinality      {:person/by-id :one
                    :person/pet   :many}}
```

then you can make queries like this:

```text
'[{(:people/all {:limit    5
                 :offset   10
                 ;; remember the extra-conditions above? you can use the same syntax here:
                 :filters [:or [:= :person/id 1]
                               [:in :person/yob 1999 2000]]
                 ;; -> you've already limited what the user can access, so let them play freely
                 ;; with whatever left open to them.
                 :order-by [:person/id
                            :person/name :desc
                            ;; Note: sqlite doesn't support `:nils-first`, `:nils-last`
                            :person/yob :desc :nils-last]})
   [:person/id :person/name]}]
```

As you can see the filter syntax is in pure Clojure. It's not just for aesthetic purpose. The generated SQL will always get parameterized so it's safe from injection attacks. For instance:

```text
[:or [:like :person/name "john"]
     [:in :person/id 3 4 7]]
```

will result in

```text
(jdbc/query ["SELECT <...> WHERE person.name LIKE ? OR person.id IN (3, 4, 7)"
             "john"]
```

You can express pretty much anything with those S-expressions. Learn more about them [here](doc/s-expressions.md).

{% hint style="info" %}
#### Who should use Walkable?

TL;DR: Anyone who have both Clojure and SQL in their toolkit.

To be more specific, anyone at any level of enthusiasm:

* Any Clojure developers who need to pull data from an SQL database \(maybe for a web/mobile app, maybe for doing some serious calculation, maybe just to migrate away to Datomic :D \)
* Clojure enthusiasts who have happen to have ORM-powered apps \(ie Korma, Django, Rails, most PHP systems, etc.\) in production but feel overwhelmed by the tedious task of developing new models \(or dreaded by the error-prone nature of modifying them.\)
* Clojure extremists who use SQL and think Clojure must be used in both backend side and frontend side, and the two sides must talk to each other using EDN data full of namespaced keywords instead of some string-based query language like GraphQL. Think not only om.next/fulcro but also reframe/reagent, hoplon, rum/prum, keechma, qlkit, quiescent etc.
{% endhint %}

## Installation

```text
;; no stable version yet
[walkable "1.1.0-SNAPSHOT"]
```

Walkable is a plugin for [Pathom](https://github.com/wilkerlucio/pathom/).

{% hint style="info" %}
Pathom is a Clojure library designed to provide a collection of helper functions to support Clojure\(script\) graph parsers using om.next graph syntax.

Don't worry if you don't know how Pathom works yet: Understanding Pathom is not required unless you use advanced features.
{% endhint %}

Walkable comes with two versions: synchronous \(for Clojure\) and asynchronous \(for both Clojure and Clojurescript\).

First of all, you need to build pathom "parser" with walkable's `sqb/pull-entities` \(or `sqb/async-pull-entities`\)

```text
(require '[com.wsscode.pathom.core :as p])
(require '[walkable.sql-query-builder :as sqb])

;; sync version
(def sync-parser
  (p/parser
    {::p/plugins
     [(p/env-plugin
        {::p/reader
        ;; walkable's main worker
         [sqb/pull-entities
         ;; pathom's entity reader
          p/map-reader]})]}))

;; async version
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

Then you need to define your floor-plan and compile it

```text
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

Details about the floor-plan is [here](https://github.com/walkable-server/walkable-server.github.io/tree/ee970dafd13b3f6a6f6b8fa3001e171b44b85f26/doc/floor-plan.md).

Ready! It's time to run your graph queries:

Sync version:

```text
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

```text
(my-run-query my-db ["select * from fruit where color = ?" "red"])
;; => [{:id 1, :color "red"} {:id 3, :color "red"} ...]

(my-run-query my-db ["select * from fruit"])
;; => [{:id 1, :color "red"} {:id 2, :color "blue"} ...]
```

Async version, Clojure JVM:

```text
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

For Nodejs, you'll need to convert between Javascript and Clojure data structure. The file [dev.cljs](https://github.com/walkable-server/walkable/tree/ab05c4706867ea7cce2daa6b903ee23834e1cf7f/dev/src/dev.cljs) has examples using sqlite3 node module.

{% hint style="info" %}
Please see the file dev.clj \(or its nodejs version dev.cljs\) for executable examples. Consult config.edn for SQL migrations for those examples.
{% endhint %}

## Special thanks to:

* Rich Hickey & Cognitect™ team for Clojure and Datomic®
* David Nolen for bringing many fresh ideas to the community including om.next
* James Reeves for Duct framework. The best development experience I've ever had
* Tony Kay for his heroic work on fulcro that showed me how great things can be done
* Wilker Lucio for pathom and being very supportive
* Sean Corfield for clojure.jdbc which we all use extensi
* Bozhidar Batsov and CIDER team!!!

## Performance

Walkable comes with some optimizations:

* A compile phase \(`floor-plan/compile-floor-plan`\) that pre-computes many parts of final SQL query strings.
* Reduce roundtrips between Clojure and SQL server by combining similar queries introduced by the same om.next join query. \(aka N+1 problem\)

More optimization will be added. Check github issues for progress.

## Limitation

* Currently Walkable only takes care of reading from the database, NOT

  making mutations to it. I think it varies from applications to

  applications. If you can think of any pattern of doing it, please

  open an issue.

## Support

I'm available for questions regarding walkable on `#walkable` clojurians slack channel. I'm also on `#fulcro` and [Clojureverse](https://clojureverse.org/)

## Legal

Copyright © 2018 Hoàng Minh Thắng

Datomic® is a registered trademark of Cognitect, Inc.

