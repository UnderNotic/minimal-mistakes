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

In days when we have cloud operator liek azure,aws
On other hand with iot beign a thing, sometimes we want to have some device sitting in our private home network behind router nat accessible from internet.

To configure github hooks, usually some work needs to be done on router side.
Our nodejs app will be informed, that repo has been updated
by post request send from github.
To make that happend:
- github needs to know what is `payload url`, which is location of the server
- our server needs to be avaialable from Internet, therafore this requires unblocking port (5000 by default) on local router (router forwarding)  
![image-center](/assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/new_forwarding.png){: .align-center }{:style="width: 100%"}
- our server needs to be avaialable from Internet, therafore this requires unblocking port on local router (router forwarding)   

![image-center](/assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/virtual-servers.png){: .align-center }{:style="width: 100%"}
- 

- 
![image-center](/assets/images/keeping-node-applications-up-to-date-by-syncing-with-github-repo/webhook.png){: .align-center }{:style="width: 100%"}