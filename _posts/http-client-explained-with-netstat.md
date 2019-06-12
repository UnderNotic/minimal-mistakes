---
title: "HttpClient explained with netstat by example"
excerpt: ""
header:
  teaser: "assets/images/http-client-explained-with-netstat/netstat.jpg"
tags:
  - dotnet
  - httpclient
  - networking
  - http
  - TCP
---

## Learn http client once again this time with netstat

This is not a guide to:

> How to use HttpClient in your app?

But rather how to use netstat and how TCP connections are behaving in different code scenarios.

Below examples will showcase dotnet `HttpClient` running next to nestat being refreshed every 1sec.  
`Keep-alive` is set to true on default so connections will be reused. Output from netstat is filtered to show only relevant data.

```bash
sudo watch -n 1 "netstat --TCP -pnW | grep 104.27"
```

## Netstat explained

Every TCP connection is a pair of local and foreign address.

Address is combination of IP/port, where:

- IP is used to locate machine
- Port is used to determine application on that machine.

If machine is talking to the server, port for local address will be picked from pool of available ports. Port for foreign address will be of course port on what remote server is listening (in this case 443).

![image-center](/assets/images/http-client-explained-with-netstat/tcp-conn.png){: .align-center }{:style="width: 75%"}

### Netstat columns:

- `Local address` IP/port of local machine
- `Foreign address` IP/port of remote machine
- Every connection is in some state, here are the most important:
  - `ESTABLISHED` connection is opened after 3-way handshake and ready to transmit the data
  - `TIME_WAIT` client side (`HttpClient`) closed the connection, but there can be still something flowing from server, because of that connection is still open and will be closed after some time
  - `CLOSE_WAIT` same as above, but from server side
  - `FIN_WAIT2` remote server has sent acknowledgement of receiving connection termination request (from httpclient)
  - `LISTENING` local server is waiting for incoming remote connections

<figure class="align-center">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/http-client-explained-with-netstat/ns.png" alt="">
  <figcaption>Output of `netstat --TCP -pnW`</figcaption>
</figure>

## Example 1

In code:

- `HttpClient` is instantiated per single request
- `HttpClient` is not disposed

In netstat:

- Each request gets its own TCP connection
- TCP connections are not closed after completion of requests
- TCP connections are closed after program is terminated

If program is continuously running:

- After some time of idleing, foreign address sends connections termination requests and local connections go to `CLOSE_WAIT` state
- System is waiting for dotnet program to close the connections if not after some time connections will be closed by operating system.

We created overhead by not-reusing existing TCP connection to (104.31.66.95:443).
Make this loop iterate over 65,535 time and you will reach maximum number of ports available, making this machine completely unresponsive.
{: .notice--danger}

<video controls autoplay loop muted width="100%">
  <source src="/assets/images/http-client-explained-with-netstat/example_1.mp4" type="video/mp4">
</video>

## Example 2

In code:

- `HttpClient` is instantiated per single request
- `HttpClient` is disposed after each request
  In netstat:
  - Each call gets its own TCP connection (see port changing on client side (192.168.1.229))
  - Connection immediately goes to `TIME_WAIT` state after disposing `HttpClient`

We still have overhead by not reusing existing connection, on the other hand port deplation is much less likely to happen. Although make this loop parallel `Parallel.ForEach` and see machine becoming unresponsive.
{: .notice--warning}

<video controls autoplay loop muted width="100%">
  <source src="/assets/images/http-client-explained-with-netstat/example_2.mp4" type="video/mp4">
</video>

## Example 3

In code:

- `HttpClient` is instantiated only once
- `HttpClient` is disposed once it executed all requests
  In netstat:
  - Each http call goes through single TCP connection
  - All connections immediately go to `TIME_WAIT` state after disposing `HttpClient`

TCP connections are finally reused but remember this is only example.  
In real-life scenarios I recommend using `HttpClient Factories`, which takes care of `HttpClients` life-cycle, [more over here.](https://dotnetcoretutorials.com/2018/05/03/httpclient-factories-in-net-core-2-1/)

<video controls autoplay loop muted width="100%">
  <source src="/assets/images/http-client-explained-with-netstat/example_3.mp4" type="video/mp4">
</video>

Keep in my mind everything above was conducted on Linux Mint, on Windows situation in netstat might be slightly different.
{: .notice}
