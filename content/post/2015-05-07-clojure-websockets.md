+++
date = "2015-05-07T01:18:26-04:00"
draft = false
title = "Clojure Websockets with http-kit"

+++


As I continue to evaluate Clojure, new opportunities present themselves. In the
past, I've done highly concurrent processing with Node.js
or a JVM solution on top of Jetty. While these work, I never found them
particularly enjoyable to use.

Enter Clojure.

<!--more-->

Recently I worked with some colleagues on an app designed around spectating runners.
To quickly create an MVP, we needed a server that would broadcast coordinates periodically to all users who cared.
Websockets seemed like the way to go,
and because various members on the team had mixed experiences with [socket.io](http://socket.io/),
we went with [http-kit](http://www.http-kit.org/), the Clojure solution for websockets.

The actual definition of the server is very easy to setup:

```clojure
(defonce server (atom nil))

(defn app [request]
  (with-channel request channel
    (on-close channel process-close)
    (on-receive channel (fn [data]
                          (process-message channel data)))))

(defn -main []
  (reset! server (run-server #'app {:port 8080})))
```

After this, all logic lives in the `process-message` function.

For initial messages, we need to create a map of ids to channels we need to broadcast on.

Because following is bidirectional in nature, we add the channel we just created to the list of channels listening to that ID.

```clojure
(def channel-map (atom {}))

(defn process-message [channel message]
  (let [json-data (json/read-str message :key-fn keyword)]
    (if (:initialRequest json-data)
      (process-initial-message json-data channel)
      (process-geo-message json-data))))

(defn process-initial-message [json-data channel]
  (doseq [follow-id-string (:following json-data)]
    (let [follow-id (keyword (str follow-id-string))]
      (if (follow-id @channel-map)
        (swap! channel-map assoc follow-id
               (conj (get-in @channel-map [follow-id]) channel))
        (swap! channel-map assoc follow-id [channel])))))
```

Each time we receive a message after the initial request,
we simply broadcast whatever we receive to the channels registered as listening to the specific ID.

```clojure
(defn process-geo-message [json-data]
  (let [user-id (:userId json-data)
        channels ((keyword (str user-id)) @channel-map)]
    (doseq [channel channels]
      (send! channel
        (json/write-str
          {:userId user-id
           :lat (:lat json-data)
           :lng (:lng json-data)}))))))
```

Combine all of the previous code snippets, and the result is
the MVP for a server that facilitates realtime bidirectional communication between several clients over websockets.

This is by no means the professional, final solution. It:

 * Has no authentication
 * Relies purely on user input to maintain the ID
 * Doesn't clean up when users disconnect

 That said, this is still a working solution to our problem, and let us get where we needed to be in as much time as it has taken me to type up this blog post.
 In addition to demonstrating Clojure is a viable solution for websockets,
 it also demonstrated that Clojure is a viable solution for quickly prototyping an application.

 Feel free to test the above application with the following code,
 which utilizes [gniazdo](https://github.com/stylefruits/gniazdo), a Clojure websocket client
 to generate and test several user-ids with several longitude / latitude combinations.
 
```clojure

 (ns spectate-server.client
  (:require [clojure.tools.logging :as log])
  (:require [clojure.data.json :as json])
  (require [gniazdo.core :as ws]))

(def url "ws://localhost:8080")

(def id-channel-map (atom {}))

(def number-of-users 100)

;; Converts a keyword to an integer
(defn parse-keyword-int [k]
  (Integer. (re-find  #"\d+" (str k)))

(defn on-receive [user-id msg]
  (log/info "User " user-id "Received " msg))

(defn createSocket [user-id]
  (ws/connect url
              :on-receive (partial on-receive user-id)))

(defn create-initial-msg [user-id following-ids]
  (json/write-str {
                   :initialRequest true
                   :following following-ids
                   :userId user-id}))

(defn create-geo-msg [user-id lat lng]
  (json/write-str {:userId user-id
                   :lat lat
                   :lng lng}))

(defn -main []
  (dotimes [user-id number-of-users]
    (swap! id-channel-map assoc (keyword (str user-id))
           (createSocket user-id)))

  (doseq [[user-id-key socket] @id-channel-map]
    (let [user-id (parse-keyword-int user-id-key)]
      (ws/send-msg socket (create-initial-msg user-id [user-id (inc user-id)]))))

  (dotimes [x 1000]
    (let [user-id (rand-int number-of-users) user-id-key (keyword (str user-id))
          socket (user-id-key @id-channel-map)]
      (ws/send-msg socket (create-geo-msg user-id (rand 100) (rand 100)))
      )))
```
