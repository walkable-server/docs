# Introduction

<table>
<tbody>
<tr style="border: none;">
<td style="border: none;text-align:left"><img src="assets/walkable.png" alt=""></td>
<td style="border: none;text-align:left"><strong>Walkable</strong> is a serious way to fetch data from SQL for Clojure: Datomic® pull (Graphql-ish) syntax, Clojure flavored filtering and more.</td>
</tr>
</tbody>
</table>

> Data dominates. If you’ve chosen the right data structures and
> organized things well, the algorithms will almost always be
> self-evident. Data structures, not algorithms, are central to
> programming. -- Rob Pike


> Bad programmers worry about the code. Good programmers worry about
> data structures and their relationships. -- Linus Torvalds

Ever imagined sending queries like this to your SQL database?

{% mdtabs title="Query" %}
```clojure
[{[:person/by-id 1]
  [:person/id
   :person/name
   :person/age
   {:person/pet [:pet/name :pet/favorite-location]}]}]
```


{% mdtab title="Data" %}
Table "person":

| id | name | age |
| :---: | :--- | :---: |
| 1 | Mary | 20 |
| 2 | John | 15 |

Table "pet":

| id | name | favorite\_location |
| :--- | :--- | :--- |
| 10 | Tom | garden |
| 20 | Jerry | kitchen |

Table "person\_pet":

| person\_id | pet\_id |
| :---: | :---: |
| 1 | 10 |
| 2 | 20 |


{% mdtab title="Result" %}
```clojure
{[:person/by-id 1]
 {:person/id   1
  :person/name "Mary"
  :person/age  20
  {:person/pet {:pet/name              "Tom"
                :pet/favorite-location "garden"}}}}
```

{% endmdtabs %}

or a bit more sophisticated:

{% mdtabs title="Query" %}
```clojure
`[{(:articles/all {:filters [:and [:= false :article/hidden]
                             {:article/author [:= :user/username "lucy"]}]})
   [:article/title
    :article/created-date
    {:article/author [:user/id :user/username :user/karma]}]}]
```


{% mdtab title="Data" %}
Table "article":

| id | title | author\_id | hidden | created\_date |
| :---: | :--- | :---: | :---: | :--- |
| 1 | Hello world | 10 | false | 2018-10-11 |
| 2 | Welcome | 20 | false | 2018-11-10 |
| 3 | Unfinished | 20 | true | 2018-09-20 |

Table "user":

| id | username | karma |
| :---: | :--- | :---: |
| 10 | mark | 21 |
| 20 | lucy | 42 |


{% mdtab title="Result" %}
```clojure
[{:articles/all
  [{:article/title        "Welcome"
    :article/created-date "2018-11-10"
    {:article/author {:user/id       20
                      :user/username "lucy"
                      :user/karma    42}}}]}]
```

{% endmdtabs %}

Yes, you can. Have your data fetched in your Clojure mission critical app with confidence. Even more, build the query part of a fulcro server or REST api in minutes today! Call it from your Clojurescript app without worrying about SQL injection.

You can learn about the above query language [here](query_language.md)

{% hint style="info" %}
### **Walkable is NOT about om.next**

People may have the impression that Walkable \(and Pathom\) is specific to om.next. That is NOT the case! Walkable requires a query language that is expressive and based off data structure. Om.next query language happens to satisfy that.

Walkable's goal is to become the ultimate SQL library for Clojure.
{% endhint %}

{% hint style="danger" %}
### For plain-SQL enthusiasts or ORM fans

I urge you to challenge your assumptions by implementing your own version of the Realworld API Spec with your favorite ORM \(Django, Rails, Korma, Toucan, whatever\) and compare it with the implementation using Walkable [here](https://github.com/walkable-server/realworld/).
{% endhint %}

{% hint style="info" %}
#### Who should use Walkable?

TL;DR: Anyone who have both Clojure and SQL in their toolkit.

To be more specific, anyone at any level of enthusiasm:

* Any Clojure developers who need to pull data from an SQL database \(maybe for a web/mobile app, maybe for doing some serious calculation, maybe just to migrate away to Datomic :D \)
* Clojure enthusiasts who have happen to have ORM-powered apps \(ie Korma, Django, Rails, most PHP systems, etc.\) in production but feel overwhelmed by the tedious task of developing new models \(or dreaded by the error-prone nature of modifying them.\)
* Clojure extremists who use SQL and think Clojure must be used in both backend side and frontend side, and the two sides must talk to each other using EDN data full of namespaced keywords instead of some string-based query language like GraphQL. Think not only om.next/fulcro but also reframe/reagent, hoplon, rum/prum, keechma, qlkit, quiescent etc.
{% endhint %}

## Special thanks to:

* Rich Hickey & Cognitect™ team for Clojure and Datomic®
* David Nolen for bringing many fresh ideas to the community including om.next
* James Reeves for Duct framework. The best development experience I've ever had
* Tony Kay for his heroic work on fulcro that showed me how great things can be done
* Wilker Lucio for pathom and being very supportive
* Sean Corfield for clojure.java.jdbc which we all use extensively
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
