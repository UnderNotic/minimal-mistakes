---
title: "Service discovery with etcd"
excerpt: "Managing state in distributed systems"
header:
 teaser: "assets/images/service-discovery-with-etcd.png"
tags: 
  - etcd
  - microservices
  - distributed
  - distributed systems
  - database
  - .net
  - csharp
--- 

### Overview
Let start right off the bat with quick overview what `etcd` is:
- distributed key/value store with failover mechanism
- heavily uses disk but also use in memory cache
- AP regarding CAP theorem
- sequential consistency ( the strongest consistency guarantee available from distributed systems)
- data is persisted, after etcd restart, previously added data is still there
- max request size 1MB
- publish/subscribe 
- get/put
- leases (TTL)
- uses grcp for communication
- uses RAFT (leader election)
- usually used for handling state in distributed systems (service discovery, shared configuration)
- used in Kubernetes

[^1]: <https://coreos.com/etcd/docs/latest/learning/api_guarantees.html/>
[^2}: <https://kubernetes.io/>


### Client Side Service Discovery

### The Heartbeat

### Load balancing

### Etcd Grcp Client library.

Todo support for certificates.


https://coreos.com/etcd/docs/latest/learning/api_guarantees.html