Walkable comes with a [Duct](https://github.com/duct-framework/duct) setup as its development environment which can be found in `dev` directory.

{% hint style="info" %}
Duct is a highly modular framework for building server-side applications in Clojure using data-driven architecture.

A more detailed guide for Duct can be found at:

[https://github.com/duct-framework/docs/blob/master/GUIDE.rst](https://github.com/duct-framework/docs/blob/master/GUIDE.rst)
{% endhint %}

## Leiningen profiles

Walkabe source code comes with three Leiningen profiles for three supported sql flavors: `postgres`, `mysql` and `sqlite`. You **must** specify one of them whenever you start a REPL server or run tests.

## Development environment

To begin developing, start with a REPL.

{% mdtabs  title="Postgres" %}
```clojure
lein with-profile postgres repl
```


{% mdtab title="Mysql" %}
```clojure
lein with-profile mysql repl
```


{% mdtab title="Sqlite" %}
```clojure
lein with-profile sqlite repl
```

{% endmdtabs %}

Then load the development environment.

```clojure
user=> (dev)
:loaded
```

Run `go` to initiate the system.

```clojure
dev=> (go)
:duct.server.http.jetty/starting-server {:port 3000}
:initiated
```

By default this creates a web server at [http://localhost:3000](http://localhost:3000).

When you make changes to your source files, use `reset` to reload any modified files and reset the server.

```clojure
dev=> (reset)
:reloading (...)
:resumed
```

## Testing

Testing is fastest through the REPL, as you avoid environment startup time.

```clojure
dev=> (test)
...
```
