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

All this make etcd good candidate to store data used in service discovery.

[^1]: <https://coreos.com/etcd/docs/latest/learning/api_guarantees.html/>
[^2}: <https://kubernetes.io/>


### Client Side Service Discovery
In client‑side discovery, the client is responsible for determining the network locations of available service instances and load balancing requests across them. The client queries a service registry (etcd), which is a database of available service instances. The client then uses a load‑balancing algorithm to select one of the available service instances and makes a request.[^3]
Client in above context is usually a library that knows how to get information what services are currently avaliable in the system.
Drawback is that you have to write this client for every programming language used in microservices that need to talk to some other microservice.

### Server Side Service Discovery

The client makes a request to a service via a load balancer. The load balancer queries the service registry and routes each request to an available service instance. As with client‑side discovery, service instances are registered and deregistered with the service registry.
Here load balancer is just another microservice but only that microservice is aware of service registry.
As oppose to client side service discovery this solution is language agnostic but on the other hand requires additional network hop.

[^3]: <https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/>
### The Heartbeat

I had written example service discovery application showcasing client side service discovery with etcd as a service registry.
Solution consists of 3 projects:



https://github.com/UnderNotic/etcd_spike

### Load balancing

### Etcd Grcp Client library.

Todo support for certificates.


https://coreos.com/etcd/docs/latest/learning/api_guarantees.html