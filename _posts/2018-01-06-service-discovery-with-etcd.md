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

* distributed key/value store with failover mechanism
* heavily uses disk but also use in memory cache
* AP regarding CAP theorem
* sequential consistency ( the strongest consistency guarantee available from distributed systems)
* data is persisted, after etcd restart, previously added data is still there
* max request size 1MB
* publish/subscribe
* get/put
* leases (Time to live)
* uses grcp for communication
* uses RAFT (leader election)
* usually used for handling state in distributed systems (service discovery, shared configuration)
* used in Kubernetes

All this make etcd good candidate to store data used in service discovery.

[^1]: <https://coreos.com/etcd/docs/latest/learning/api_guarantees.html/>

[^2}: <https://kubernetes.io/>

### Client Side Service Discovery

In client‑side discovery, the client is responsible for determining the network locations of available service instances and load balancing requests across them.   
The client queries a service registry (etcd), which is a database of available service instances. The client then uses a load‑balancing algorithm to select one of the available service instances and makes a request.[^3]   
Client in above context usually uses a library(common project or package) that knows how to get information what services are currently avaliable in the system.   
Drawback is that you have to write this client for every programming language used in microservices that need to talk to some other microservice.

### Server Side Service Discovery

The client makes a request to another microservice via load balancer. Load balancer queries the service registry and routes each request to an available service instance.   
As with client‑side discovery, service instances are registered and deregistered with the service registry.   
Here load balancer is just another microservice but only that microservice is aware of service registry.
As oppose to client side service discovery this solution is language agnostic but on the other hand requires additional network hop.

[^3]: <https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/>

### Example project [Github repo](https://github.com/UnderNotic/etcd_spike){: .btn .btn--primary}{:target="_blank"}

I had written example service discovery application showcasing server side service discovery with etcd as a service registry.

Solution consists of 3 projects:

* Etcd.Client
  * grpc client to cummunicate with etcd
* Gateway
  * load-balancer in service side discovery
  * load-balancing incoming requests across available 'Node' instances
* Node
  * subscribes to etcd by doing periodic heartbeats
  * handles requests

![image-center](/assets/diagrams/etcd-discovery.svg){: .align-center }{:style="width: 850px"}

### The Heartbeat

`Node` reprents whatever service in the system that can scale horizontally and to whom requests are load-balanced.   
On startup node is taking whatever port is available starting from 8080, then it's putting lease in etcd, which is basically setting key-value pair with 10 seconds TTL (time to live), once it is set, it starts refreshing TTL every 5 seconds. If Node dies, hangs... setted up heatbeat key-value pair will be removed at the same time making this node unavailable and undiscoverable for the gateway.

Key is designed in the following way `heartbeat|{NODE_TYPE}|{url}`, so the gateway can make use of etcd range subscribe to watch only for keys that starts with `heartbeat`, also from that key, information about node type and url that it is hosted at can be retrieved.

```csharp
 private async void Run(string url)
       {

           var leaseId = await etcdClient.PutWithLease($"heartbeat|{NODE_TYPE}|{url}", url, (int)TTL.TotalSeconds);
           this.timer = new Timer(_ => etcdClient.KeepAliveLease(leaseId), null, TimeSpan.Zero, interval);
       }
```

### Load balancing

`Gateway` manages dictionary structure that holds information about available nodes in the system.
It subscribes to heartbeats in etcd using WatchRange and update dictionary state accordingly.

Using that dictionary incoming requests are load-balanced to random available node.

```csharp
 private async Task<HttpStatusCode> HandleRequest(string nodeType)
    {
        var hosts = serviceDiscovery.availableNodes[nodeType].Select(s => s.Address).ToArray();
        var host = hosts[random.Next(0, hosts.Length)];
        await httpClient.GetAsync(host);
        return HttpStatusCode.OK;
    }
```

```csharp
 public class ServiceDiscovery : IDisposable
    {
        private EtcdClient etcdClient { get; } = new EtcdClient();
        public IDictionary<string, IList<DiscoverableService>> availableNodes { get; private set; } = new Dictionary<string, IList<DiscoverableService>>();
        private EtcdWatcher watcher;

        public ServiceDiscovery()
        {
            Run();
        }

        private async Task Run()
        {
            availableNodes = RangeServicesToDictionary(await etcdClient.GetRange("heartbeat"));

            watcher = await etcdClient.WatchRange("heartbeat");
            watcher.Subscribe(events =>
                {
                    foreach (var e in events)
                    {
                        var service = DiscoverableService.CreateFromEtcdKey(e.Key);
                        switch (e.Type)
                        {
                            case EventType.Put:
                                IList<DiscoverableService> values;
                                if (availableNodes.TryGetValue(service.Type, out values))
                                {
                                    values.Add(service);
                                }
                                else
                                {
                                    availableNodes.Add(service.Type, new List<DiscoverableService> { service });
                                }
                                break;
                            case EventType.Delete:
                                var valuesForKey = availableNodes[service.Type];
                                var isDeleted = valuesForKey.Remove(service);
                                if (!isDeleted)
                                {
                                    throw new Exception("This can not happen");
                                }
                                if (!valuesForKey.Any())
                                {
                                    availableNodes.Remove(e.Key);
                                }
                                break;
                        }
                    }
                    Console.WriteLine($"Available nodes {availableNodes.Aggregate(string.Empty, (acc, item) => $"{item.Key} - {item.Value.Aggregate(string.Empty, (a, i) => $"{i.Address} {a}")} {acc}")}");
                });
        }
    }
```

### Try it out

Clone the repo, build, run and observe console to grasp how it works.
Gateway is exposing following api:

```csharp
public ApiModule()
    {

        Get["/", runAsync: true] = async (args, ct) =>
        {
            return await HandleRequest();
        };

        Get["/send/{nodeType}", runAsync: true] = async (args, ct) =>
        {
            return await HandleRequest(args.nodeType);
        };
    }
```

With that gateway is calling node process, which exposes by following code
```csharp
public class ApiModule : NancyModule
{
    public ApiModule()
    {
        Get["/"] = _ =>
        {
            return HandleRequest();
        };

        Get["/{path*}"] = _ =>
        {
            return HandleRequest();
        };
    }

    private HttpStatusCode HandleRequest()
    {
        Console.WriteLine($"Received request {this.Request.Url}");
        return HttpStatusCode.OK;
    }
}
```

### Etcd Grcp Client library.

I extracted grpc part from above project to seperate github repo and created nuget out of that.   
It greatly simplifies communication with etcd in dotnet projects, and since it's .net standard package it can be used in both .net framework and .net core projects.   
Check it out if it fits your needs and contribute :)

[Github repo](https://github.com/UnderNotic/EtcdGrpcClient){: .btn .btn--primary}{:target="_blank"}
[Nuget page](https://www.nuget.org/packages/EtcdGrcpClient){: .btn .btn--primary}{:target="_blank"}
