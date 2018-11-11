---
description: >-
  Emitter is part of floor-plan that tells Walkable how to generate SQL strings
  in respect of your database.
---

# Emitter

Emitter is a `hash-map` with the following keys `:quote-marks`, `:transform-table-name`,  `:transform-column-name`, `:rename-tables`, `:rename-columns`, `:rename-keywords`, `:wrap-select-strings`.

There are some pre-defined emitters in `walkable.sql-query-builder.emitter` namespace, namely `sqlite-emitter`, `mysql-emitter` and `postgres-emitter`. You may make your own emitter from scratch \(which means providing all the above keywords\), but it's better to start from one of the pre-defined emitters and override your custom keys.



