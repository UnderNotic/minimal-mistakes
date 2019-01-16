---
title: "Exposing local server to the public internet"
excerpt: "Your server visible everywhere"
header:
 teaser:
tags: 
  - raspberrypi
  - raspberry
  - router
  - internet
  - webhook
  - server
---

## Intro

In days when we have cloud operator exposing local servers to internet is not a common thing to do.
On other hand with IOT beign a thing, sometimes we want to have some device sitting in our private home network behind router nat accessible from internet.

## Router port forwarding

In general typical home network consist of router, which is our gate to internet. Everything behind router sits in private network, to expose a server in this private network, router has to be configured to forward all trafic comming to it.

Keep in mind that this approach requires public ip assigned to our router from ISP.

In tp-link world it look like this:
- add virtual server entry
- specify service port -> port that will be called from outside
- specify internal port -> port of our local server
- ip address -> local ip address to the server
- protocol -> tcp, udp or both can be chosen
- status -> turn off forwarding without deleting entry
- common service port -> configuration templates

![image-center](/assets/images/exposing-local-server-to-the-public-internet/new_forwarding.png){: .align-center }{:style="width: 100%"}

![image-center](/assets/images/exposing-local-server-to-the-public-internet/virtual-servers.png){: .align-center }{:style="width: 100%"}


Now check your public ip here https://www.whatismyip.com/what-is-my-public-ip-address/ and access you server.

## Aternatives
Alternatively there are tools that do all this trickery and more  and helps you expose your local server to the world even if you don't have public ip. If interested check [here](https://ngrok.com/)