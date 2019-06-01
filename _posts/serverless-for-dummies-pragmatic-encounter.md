---
title: "Serverless for dummies - pragmatic encounter on azure - creating/solving real world problem"
excerpt: "Serverless to dummies - pragmatic encounter on azure"
header:
 teaser:
tags: 
  - serverless
  - c#
  - dotnet
  - .net
  - .net core
  - azure
--- 

## Here I am without the server

Pros/cons
stateless - put state into external system/db/whatever
you focus on solving problems/writing code

 scale as you go, pay as you go, serverless in on rise there is a lot of traction to push new abililties, features into serverless
limited message size, limited active connection time(only request reply), limited number of concurrent connections
//https://docs.microsoft.com/en-us/azure/azure-functions/manage-connections

harder/trickier to do statefull things like connection pooling (i.e. connections pooling to rmds databases)

## State of serverless??
Many options
azure, aws, frameworks hosted on kubernetes, serverless framework (that you can tune to your needs), open source etc.

## But What can I build?
Typical question is what I can build it with it and asnwer to that is typical `it depends` or everything.
Most of the time serverless is used as a additional layer on top of typical server-full apps :). But you can use them to solve little problems, and compose functions together to create soemthing `bigger`
Solving little problems - threat functions as single purpose endpoint that solve one problem

https://spotifytranslatorapp.azurewebsites.net/api/map?playlist=37i9dQZF1DXcBWIGoYBM5M

## Let's create something useful
## Not another hello world example

## What we need?
There are many serverless provided, here we will use azure  functions 2.0 with .net core.


We will need to connect to api of spotify and youtube. Since only limited usage of these api's is free, we need to create youtube app and spotify app so our usage can be tracked. The in code we need to pass our  application keys, which are secrets that authenticate our app.
We will use application settings for that. But if more poweer is needed have a look at azure key-vault fit into this requirement very well, its ment to store secrets :).

Important part => How to obtain application keys for your yt and spotify apps.


## Scafolding
```bash
  mkdir SpotifyToYoutubeTranslator && cd $_
  func init --worker-runtime dotnet
  func new --language C# --name SpotifyToYoutubeTranslator
```

<!-- gif here -->

Let's remove what is inside `Run` method in `ServerlessForDummies.cs` file.

## Reading application keys

## HttpClient in static field
Check on docs about static field in azure functions -> how long they live

## Getting playlist from spotify
First we need to know what playlist we want to take from spotify

## Caching spotify token

## getting youtube links

## Returning

## Tokens

//https://stackoverflow.com/questions/39387423/sharing-a-object-across-several-azure-function-instances