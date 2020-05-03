---
title: "HTTP/2 Multiplexing explained by example"
excerpt: "How can http/2 multiplexing affect website loading time"
header:
  teaser: "assets/images/http2-multiplexing/teaser.png"
tags:
  - http
  - http/2
  - multiplexing
  - TCP
  - networking
  - javascript
  - nodejs
  - node
---

## Intro

Multiplexing is one of the new things coming with http/2. Let's see what is is and how it can affect website loading times.

> Multiplexing is a method by which multiple digital signals are combined into one signal over a shared medium. The aim is to share a scarce resource. For example, in telecommunications, several telephone calls may be carried using one wire.[^1]

[^1]: <https://en.wikipedia.org/wiki/Multiplexing>

  <img class="align-center" src="{{ site.url }}{{ site.baseurl }}/assets/images/http2-multiplexing/1.png" alt="">

In case of http/2, there could be multiple requests flowing over 1 tcp connection. They are flowing independently, meaning requests does not have to end in any particular order. In case of http/1, single tcp connection can have only one active request going through it.

## Note about http/1.1

Pipelining introduced in http/1.1, is similar to multiplexing but not as powerful. In this approach requests have to return back to the client with the same order they were send to the server.
Basically this can cause situation when requests which ended faster will be waiting for even one slow request to finish.

Pipelining didn't get enough traction and has it own set of problems, thus it is not being used in any of modern browser.
Instead http/2 multiplexing is taking place of what pipelining was meant to be.

## Dem puppies

To showcase how multiplexing works in real life, I've created nodejs server, which is serving website files using http/1 and http/2

- website consist of 12 picture of puppies
- each picture is around ~~300KB in size
- internet speed was limited to better visualise differences, this was done in chrome by picking so called `Fast 3G` in developer tools.

Let's jump into scenarios!

## Scenario with no server-side delay

In this scenario there is no latency or blocking added on server-side.
Meaning the time needed to retrieve puppy image is generally affected only by network bandwidth `Fast 3G`.

<figure>
  <figcaption>http/1</figcaption>
  <video controls autoplay loop muted width="100%">
    <source src="/assets/images/http2-multiplexing/http1.mp4" type="video/mp4">
  </video>
</figure>

<figure>
  <figcaption>http/2</figcaption>
  <video controls autoplay loop muted width="100%">
    <source src="/assets/images/http2-multiplexing/http2.mp4" type="video/mp4">
  </video>
</figure>

Wow the timings are very similar. Actually time which took to load all picture completely is exactly the same `19.35ms`.

If You take a closer look, there are some differences:

- http/2
  - all puppies are started to load at the same time
- http/1
  - there are only 6 puppies that can be loading at the same time
  - Initially only 6 out of 12 puppies are being loaded, and then additional are being loaded only when one of initial ones is already loaded.

The reason is that, there is a limitation to how many tcp connection can be opened. Popular browsers allows to open up to 6 parallel tcp connection to the same origin.

Timing are similar, because, the real bottleneck here is `Fast 3G` bandwidth, unfortunately there is no magic in http/2 that could make internet faster.

Although this is not visible in speed, http2 is more efficient here as it is using only one tcp connections to load 12 images, with http/1, each request retrieving call has its own tcp connection and creating such creates some overhead.
{: .notice--warning}

## Scenario with request being slow on server-side

In second scenario, there is a server-side delay of 10 seconds added, when retrieving pictures for first 6 puppies.

<figure>
  <figcaption>http/1</figcaption>
  <video controls autoplay loop muted width="100%">
    <source src="/assets/images/http2-multiplexing/http1_delay.mp4" type="video/mp4">
  </video>
</figure>

<figure>
  <figcaption>http/2</figcaption>
  <video controls autoplay loop muted width="100%">
    <source src="/assets/images/http2-multiplexing/http2_delay.mp4" type="video/mp4">
  </video>
</figure>

Here we can see that it took quite a lot more time to see all puppies.
Http/2 is not affected by first 6 calls being slow. Since  retrieving data for all puppies is starting at the same time, `power of internet` goes where it can and retrieve data where it can, in this case generally bandwidth is used first for retrieving 6-12 puppy and then after 10 seconds delay to retrieve first 6 puppies.

## Source code

Source code from above examples are available on [github.](https://github.com/UnderNotic/http-multiplexing){:target="\_blank"}

Check it out and have fun.
