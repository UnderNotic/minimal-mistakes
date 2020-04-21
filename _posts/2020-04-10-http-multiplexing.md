---
title: "http multiplexing"
excerpt: "http multiplexing"
header:
 teaser: "assets/images/http2-multiplexing/teaser.png"
tags:
  - http
  - multiplexing
---

## Http/2 multiplexing
One  of the features coming with http/2 is multiplexing let's see how this can affect website loading times.

> Multiplexing is a method by which multiple digital signals are combined into one signal over a shared medium. The aim is to share a scarce resource. For example, in telecommunications, several telephone calls may be carried using one wire. - Wikipedia

In this case we can have multiple requests flowing over 1 tcp connection. They are flowing independently, meaning requests does not have to end in any particular order. In case of http/1, single tcp connection can have only one active request going through it.
# pic1


# Note about http 1.1 pipelining
Http/1.1 introduced pipeling, which is somehow a middleground between http/1 and http/2 multiplexing.
It didn't get enough traction and has it own set of problems, thus it is not being used in any of modern browser. 
Instead  http/2 multiplexing is taking place what pipeling was meant to be. 

https://stackoverflow.com/questions/34478967/what-is-the-difference-between-http-1-1-pipelining-and-http-2-multiplexing
https://stackoverflow.com/questions/30477476/why-is-pipelining-disabled-in-modern-browsers

# Dem puppies (info about example in nodejs)

# Scenario with slow internet (fast 3G)
##GIF #1


Queueing. The browser queues requests when:
There are higher priority requests.
There are already six TCP connections open for this origin, which is the limit. Applies to HTTP/1.0 and HTTP/1.1 only.
The browser is briefly allocating space in the disk cache

https://developers.google.com/web/tools/chrome-devtools/network/reference#timing-explanation

# Scenario with request being slow on server side (server-side 10 second delay for first 6 pictures)
##GIF #2


# Finish

https://github.com/UnderNotic/http-multiplexing


Enim voluptate aute sit non ad in proident adipisicing eu sit minim exercitation sit aute. Reprehenderit eiusmod eu veniam ullamco adipisicing ut velit irure ipsum exercitation et nostrud. Duis consequat exercitation commodo id non mollit do. Ut et veniam fugiat tempor magna reprehenderit excepteur minim elit. Dolore amet sunt proident commodo magna quis non excepteur anim nostrud deserunt deserunt nisi. Eiusmod laborum proident qui non adipisicing ullamco aliquip culpa occaecat in velit fugiat amet nulla.
