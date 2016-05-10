+++
date = "2016-05-09T01:14:12-04:00"
draft = true
title = "Useful Leiningen Plugins"

+++

INTRO


### Alembic

From beginner to expert, most Clojure developers spend their time in the REPL. The quick iteration and test time is a big sell for Clojure development in general. But, the fast iteration goes away as soon as you need to pull in a separate dependency. After you add the dependency to your project.clj, you then need to stop and start the REPL. All of this breaks your train of thought. Worse, if you don't have the appropriate tooling set up, you'll lose the history of your REPL.

[Alembic](https://github.com/pallet/alembic) fixes that.

With Alembic as a dependency in your project, you can easily reload your `project.clj` with

``` clojure
(use 'alembic.still)
(load-project)
```

You can also use Alembic to try out any libraries without adding them to your `project.clj`, by running

``` clojure
(use 'alembic-still)
(distill '["5.5.0"])
```

You can add Alembic to every REPL you start by adding `{:repl {:plugins [[lein-ancient "0.6.2"]]}}` to `~/.lein/profiles.clj`.

### Lein Ancient
