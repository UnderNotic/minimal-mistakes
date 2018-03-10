---
title: "Url reservation in owin"
excerpt: "Stars and pluses"
header:
 teaser:
tags: 
  - hosting
  - owin
  - urls
  - ports
  - .net
  - netsh
  - csharp
  - urlacl
--- 

## Strong wildcard +
Using strong wildcard You can reserve all hostnames on given port.   
Thing to remember is that even if there is another service that is reserving url on the same port without wildcard - 
all http requests will go anyway to service that is using strong wildcard.         

Sometimes example is more than thousand words:
``` csharp
//Service #1
using (WebApp.Start<Startup> ("http://mywebsite:80/")) 
{
    Console.WriteLine("Service #1");
}

//Service #2
using (WebApp.Start<Startup> ("http://+:80/")) 
{
    Console.WriteLine("Service #2");
}
```
In above all http traffic on port 80 will go to `Service #2`.  
`Console.WriteLine("Service #1")` won't be called at all.

## Weak wildcard *
Suppose You have a service to which you want to redirect all trafic going through port 80, but at the same time You want to have another service running also on port 80 that will be accessible only using specified url (`http://mywebsite:80/`). If so weak wildcard is your friend.

Consider this example:
``` csharp
//Service #1
using (WebApp.Start<Startup> ("http://mywebsite:80/")) 
{
    Console.WriteLine("Service #1");
}

//Service #2
using (WebApp.Start<Startup> ("http://*:80/")) 
{
    Console.WriteLine("Service #2");
}
```
Any request other than one to `http://mywebsite:80/` will end up in Service #2.


Important thing to mention is if your not executing above code as user who is admin, you have to add same urls you provide to owin to netsh using `netsh http add urlacl` syntax.

Cheers!