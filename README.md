## yang <img src="doc/img/yang-logo.png" width="42px">

.. of Clojure's yin

[![<! release](https://img.shields.io/badge/dynamic/json.svg?label=release&url=https%3A%2F%2Fclojars.org%2Ftolitius%2Fyang%2Flatest-version.json&query=version&colorB=blue)](https://github.com/tolitius/yang/releases)
[![<! clojars](https://img.shields.io/clojars/v/tolitius/yang.svg)](https://clojars.org/tolitius/yang)

- [why](#why)
- [show me](#show-me)
  - [lang](#lang)
  - [time](#time)
  - [codec](#codec)
  - [network](#network)
  - [schedule](#schedule)
  - [exceptions](#exceptions)
  - [io](#io)
  - [java](#java)  
    - [from java](#from-java)  
    - [from clojure](#from-clojure)  
- [license](#license)

## why

.. not to carry often useful functions from project to project

some of these have a faint "missing from Clojure" feeling<br/>
some just very useful but have no such feeling

one thing they all have in common: **no external dependencies**

## show me

ok. here are a few examples, there are many more inside:

### lang

```clojure
=> (require '[yang.lang :as l])

=> (l/fmk {"a" 42 "b" 34} keyword)
{:a 42, :b 34}

=> (l/fmv {"a" 42 "b" 34} inc)
{"a" 43, "b" 35}

=> (l/dissoc-in {:foo {:bar {:a 42 :b "don't need this"}}}
                [:foo :bar :b])
{:foo {:bar {:a 42}}}

=> (l/dash-keys {:a_foo 42 :b_bar 34})
{:a-foo 42, :b-bar 34}

=> (l/and-> 5 number? pos?)
true
=> (l/and-> nil number? pos?)
false

=> (l/group-by-ns {:a/one :a-one :b/one :b-one :a/two :a-two :b/two :b-two})
{:a {:one :a-one, :two :a-two},
 :b {:one :b-one, :two :b-two}}

=> (l/join [{:a 20, :b 34} {:a 31, :b 27} {:a 28, :b 42}]
           [{:a 31, :b 27} {:a 12, :b 4} {:a 28, :b 42}]
           :a)
[{:a 31, :b 27} {:a 28, :b 42}]

=> (l/merge-maps {:a {:b {:c 12}} :d 21 :z 34}
                 {:a {:b {:c 42}} :d 25})
{:a {:b {:c 42}}, :d 25, :z 34}

=> (l/gzip-edn {:a 42 :b 28 :c [{:z #{:a :b 42}}]})
#object["[B" 0x2aafa84f "[B@2aafa84f"]

=> (l/gunzip-edn *1)
{:a 42, :b 28, :c [{:z #{:b 42 :a}}]}
```

### time

```clojure
=> (require '[yang.time :as t])

=> (def dates [{:date (t/now-utc)} {:date (t/now-utc)} {:date (t/now-utc)}])
#'user/dates

;; DESC in time
=> (sort-by :date t/time> dates)
({:date #object[java.time.Instant 0x2f75a9b1 "2020-07-13T19:56:11.794186Z"]}
 {:date #object[java.time.Instant 0x9cc0505 "2020-07-13T19:56:11.794174Z"]}
 {:date #object[java.time.Instant 0x26cdd4af "2020-07-13T19:56:11.794141Z"]})

;; measure things
=> (t/measure "42 sum" println (reduce + (range 42)))
"42 sum" took: 79,319 nanos
861
```

### codec

```clojure
=> (require '[yang.codec :as c])

=> (c/base64-encode (-> "distance from you to mars is 69,561,042" .getBytes))
"ZGlzdGFuY2UgZnJvbSB5b3UgdG8gbWFycyBpcyA2OSw1NjEsMDQy"

=> (-> "ZGlzdGFuY2UgZnJvbSB5b3UgdG8gbWFycyBpcyA2OSw1NjEsMDQy"
       c/base64-decode
       String.)
"distance from you to mars is 69,561,042"
```

### network

```clojure
=> (require '[yang.network :as n])

=> n/hostname
"tweedledee/10.143.34.42"
```

```clojure
=> (n/uri->map "postgresql://192.168.10.42:4242/planets")

{:path "/planets",
 :user-info nil,
 :fragment nil,
 :authority "192.168.10.42:4242",
 :port 4242,
 :host "192.168.10.42",
 :scheme-specific-part "//192.168.10.42:4242/planets",
 :query nil,
 :scheme "postgresql"}
```

```clojure
=> (n/jdbc-uri->map "jdbc:postgresql://192.168.10.42:4242/planets")

{:path "/planets",
 :user-info nil,
 :fragment nil,
 :authority "192.168.10.42:4242",
 :port 4242,
 :dbname "planets",
 :host "192.168.10.42",
 :scheme-specific-part "//192.168.10.42:4242/planets",
 :query nil,
 :scheme "postgresql"}
```

### schedule

```clojure
=> (require '[yang.scheduler :as s])
```

schedule functions to run on intervals:

```clojure
=> (def hh (s/every 1000 #(println "hey humans!")))
#'user/hh

hey humans!
hey humans!
hey humans!
hey humans!
hey humans!
hey humans!
hey humans!
hey humans!
hey humans!

user=> (s/stop hh)
true
```

start/stop a farm of threads running a function:

```clojure

=> (defn f []
     (println (s/thread-name))
     (Thread/sleep 5000))
#'user/f

;; schedule a function "f" to run with 42 threads:

=> (def farm (s/run-fun f 42))
yang-runner-0
yang-runner-1
yang-runner-2
...
yang-runner-39
yang-runner-40
yang-runner-41

;; stop the farm of threads from calling "f":

=> (-> farm :running? (reset! false))
false

=> farm
{:pool ThreadPoolExecutor [Running, pool size = 42, active threads = 0, queued tasks = 0, completed tasks = 42]"],
 :running? #atom[false 0x340b4f07]}
```

schedule to run a function `n` times (on a different thread):

```clojure
=> (sc/ftimes 5 #(println "lotery numbers are:" (repeatedly 5 (fn [] (rand-int 42)))))
lotery numbers are: (31 2 27 29 28)
lotery numbers are: (3 28 40 15 1)
lotery numbers are: (13 26 18 19 21)
lotery numbers are: (37 18 18 23 17)
lotery numbers are: (7 16 20 35 8)
```

### exceptions

```clojure
=> (require '[yang.exception :as ex])

=> (ex/set-default-exception-handler)
```

### io

```clojure
=> (require '[yang.io :as io])

=> (io/file->bytes "src/yang/io.clj")
#object["[B" 0x6339e604 "[B@6339e604"]
```

### java

#### from Java

```java
jshell> import tolitius.Yang;

jshell> var m = Map.of("foo", 42, "nested", Map.of("bar", 34), "zoo", 28)
m ==> {foo=42, nested={bar=34}, zoo=28}

jshell> var edn = Yang.mapToEdn(m)
edn ==> {:foo 42, :nested {:bar 34}, :zoo 28}

// so now it can be injected in any Clojure lib that is called from Java and expects EDN
```

#### from Clojure

```clojure
=> (require '[yang.java :as j])

=> (def m (java.util.HashMap. {"crux.id/foo" {"crux.db/bar" "baz"} "crux.asnwer/life" 42}))

=> (j/map->edn m)
{:crux.id/foo #:crux.db{:bar "baz"}, :crux.asnwer/life 42}
```

## license

Copyright © 2020 tolitius

Distributed under the Eclipse Public License either version 1.0 or (at
your option) any later version.
