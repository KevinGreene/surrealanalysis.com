+++
date = "2016-05-09T01:14:12-04:00"
draft = false
title = "Useful Leiningen Plugins"

+++

Clojure has a great development flow, but a few tools aren't included by default. By adding a few dependencies and plugins to `~/.lein/profiles.clj`, you can make your development workflow smoother, quicker, and more effective.

<!--more-->


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
(distill '[cheshire "5.5.0"])
```

You can add Alembic to every REPL by adding the following to `~/.lein/profiles.clj`.

```
{:repl {:dependencies [[alembic "0.3.2"]]}}
``` 

### Lein Ancient

As you come back to older projects, it can be useful to update them to the latest library. Instead of going to [Clojars)[https://clojars.org/] for each library you use, instead consider using [lein-ancient](https://github.com/xsc/lein-ancient).

By using `lein-ancient`, you can determine the outdated libraries in your `project.clj`. Running `lein ancient` will produce output similar to

```
[org.webjars/font-awesome "4.6.2"] is available but we use "4.6.1"
[org.webjars.bower/tether "1.3.2"] is available but we use "1.3.1"
[org.clojure/tools.cli "0.3.5"] is available but we use "0.3.4"
[clj-http "3.0.1"] is available but we use "2.1.0"
[lein-figwheel "0.5.3-1"] is available but we use "0.5.2"
```

From here, you can update any libraries as you see fit. As always, be careful blindly upgrading dependencies, and be sure to test extensively after changing any of them.

You can add lein-ancient to every project by adding the following to `~/.lein/profiles.clj`.

```
{:user {:plugins [[lein-ancient "0.6.10"]]}}
``` 

### Eastwood

With ongoing projects, it can be easy to let code quality slip. Luckily, great linters like [Eastwood](https://github.com/jonase/eastwood) exist to let you know when you stray from best practices.

With Eastwood as a plugin, you can run `lein eastwood '{:linters [:all]}'`. This produces a list of violations, that you can then work to correct. You can customize which linters you care about as described [here](https://github.com/jonase/eastwood#whats-there).

This will produce an output like:

```
== Eastwood 0.2.3 Clojure 1.8.0 JVM 1.8.0_77
Directories scanned for source files:
  env/dev/clj test/clj src/clj src/cljc test
Entering directory `/Users/kevingreene/programming/daily-cider'
src/cljc/daily_cider/validation.cljc:1:1: non-clojure-file: Non-Clojure file 'src/cljc/daily_cider/validation.cljc'.  It will not be linted.
test/cljs/daily_cider/core_test.cljs:1:1: non-clojure-file: Non-Clojure file 'test/cljs/daily_cider/core_test.cljs'.  It will not be linted.
test/cljs/daily_cider/doo_runner.cljs:1:1: non-clojure-file: Non-Clojure file 'test/cljs/daily_cider/doo_runner.cljs'.  It will not be linted.
== Linting daily-cider.dev-middleware ==
== Linting daily-cider.env ==
== Linting daily-cider.config ==
== Linting daily-cider.layout ==
src/clj/daily_cider/layout.clj:10:1: non-dynamic-earmuffs: #'daily-cider.layout/*app-context* should be marked dynamic
== Linting daily-cider.middleware ==
== Linting daily-cider.routes.home ==
src/clj/daily_cider/routes/home.clj:1:1: unused-fn-args: Function arg request__16360__auto__ never used
src/clj/daily_cider/routes/home.clj:1:1: unused-fn-args: Function arg request__16360__auto__ never used
== Linting daily-cider.processor ==
src/clj/daily_cider/processor.clj:1:1: unused-namespaces: Namespace clojure.java.shell is never used in daily-cider.processor
src/clj/daily_cider/processor.clj:1:1: unused-namespaces: Namespace luminus.logger is never used in daily-cider.processor
src/clj/daily_cider/processor.clj:1:1: unused-namespaces: Namespace clojure.java.io is never used in daily-cider.processor
== Linting daily-cider.routes.cider ==
src/clj/daily_cider/routes/cider.clj:1:1: unused-namespaces: Namespace clojure.java.io is never used in daily-cider.routes.cider
src/clj/daily_cider/routes/cider.clj:1:1: unused-fn-args: Function arg request__16360__auto__ never used
src/clj/daily_cider/routes/cider.clj:1:1: unused-fn-args: Function arg request__16360__auto__ never used
== Linting daily-cider.handler ==
== Linting daily-cider.figwheel ==
== Linting daily-cider.core ==
src/clj/daily_cider/core.clj:1:1: unused-namespaces: Namespace daily-cider.processor is never used in daily-cider.core
== Linting user ==
env/dev/clj/user.clj:1:1: unused-namespaces: Namespace daily-cider.figwheel is never used in user
== Linting daily-cider.test.handler ==
== Linting daily-cider.test.processor ==
test/daily_cider/test/processor.clj:1:1: unused-namespaces: Namespace clojure.string is never used in daily-cider.test.processor
== Warnings: 15 (not including reflection warnings)  Exceptions thrown: 0
Subprocess failed
```


You can add Eastwood to every project by adding the following to `~/.lein/profiles.clj`.

```
{:user {:plugins [[jonase/eastwood "0.2.3"]]}}
``` 
