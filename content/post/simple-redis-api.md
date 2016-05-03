+++
date = "2016-04-22T01:03:14-04:00"
draft = false
title = "A Simple Redis Slack Bot"

+++

Though I'm many years late to the game, I just started playing with Ruby, and I've been enjoying it quite a bit. After creating an API with Sinatra for an internal testing tool at Spantree, I was ready to try it out on a side project.

<!--more-->

Because I'm a remote employee, I depend heavily on Slack. Slack keeps me connected to the community at my company, the cities I'm in, and the technologies I use. As a result, a lot of my side projects end up being enhancements to Slack. While using Slack, I often want to keep track of a simple number, or a simple value. When I want to do this in an application, my default is Redis, so why not use that while Slacking?

Enter [a simple Redis Slackbot](https://github.com/KevinGreene/redis-api), a project I threw together one evening. Just 36 lines of code, and we have a functioning Slackbot.

The code is a fairly straightforward wrapper around the Redis API, but I like one bit of code more than the others:

``` ruby
  message = params["text"].split
  result = redis.send(*message)
```

This splits the message based on whitespace and calls the first element in the message with the rest of the elements in the message as arguments. For example, if the user typed `/redis set this 1` then the second line of code is functionally `result = redis.set("this", 1)`.

It's far from the prettiest code I've ever written. It's untested, imperative, and doesn't have much to it. But it works. It went from conception to implementation to deployed in less than a day. And it serves a useful purpose, which is about as much as I can ask from any side project.
