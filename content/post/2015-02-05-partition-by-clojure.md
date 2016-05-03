+++
date = "2015-02-05T01:14:12-04:00"
draft = false
title = "Partition By in Clojure"

+++

On every professional software engineering job I've had, data transformation always played a role. Most recently, at Spantree we needed to get a massive corpus of synonyms into the file format readable by [Elasticsearch](http://www.elasticsearch.org/).

<!--more-->

We received a 640MB text file, with over 5.4 million lines of the form

```
C0000005|...|Dog|...
C0000005|...|Canine|...
C0000039|...|Cat|...
C0000039|...|Feline|...
```

While a Groovy or Python solution was possible, this was definitely a problem that would require processing things line by line.

Instead of relying on keeping track of state line by line, this seemed like a great opportunity to try out [Clojure](http://clojure.org). The crux of the application boils down to two functions.

First, the processLine function. This takes in a given string, and returns a map with the given ID and word.

```clojure
(defn processLine
  [line]
  (let [splitLine (str/split line #"\|")]
    {:id (nth splitLine 0)
     :word (nth splitLine 3)}))
```

While the syntax is different, this is fairly similar to what would have been used in Groovy or Python.

After we read in a line, we need to group the lines based on their ID. This is when Clojure began to really show improvement in brevity and style. The function `partition-by` allows us to create sequences of groups based on the result of a given function. In this case, the given function is simply getting the ID of a line.

```clojure
(defn groupSynonymsById
  [allSynonyms]
  (partition-by #(:id %) allSynonyms))
```

With some basic wrappers like `importAllSynonyms` to load the file and map `processLine`, and `writeOutGroupedSynonyms` to create the formatting we want, the entire main method becomes

```clojure

(writeOutGroupedSynonyms (groupSynonymsById (importAllSynonyms)))
```

Nothing done was exclusive to Clojure, but the combination of built in functions and succinct syntax allowed us to produce a script with less than 15 lines of actual code. This will allow us to come back and understand the code with little effort, making any changes quick and painless.
