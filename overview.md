Basically you define your floor-plan like this:

```clojure
{:idents           { ;; query with `[:person/by-id 1]` will result in
                    ;; `FROM person WHERE person.id = 1`
                    :person/by-id :person/id
                    ;; just select from `person` table without any constraints
                    :people/all "person"}
 :true-columns    #{:person/name :person/yob}
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

```clojure
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

```clojure
[:or [:like :person/name "john"]
     [:in :person/id 3 4 7]]
```

will result in

```clojure
(jdbc/query ["SELECT <...> WHERE person.name LIKE ? OR person.id IN (3, 4, 7)"
             "john"]
```

You can express pretty much anything with those S-expressions. Learn more about them [here](s-expressions.md).
