---
title: "Serverless for dummies - pragmatic encounter on azure"
excerpt: "Serverless to dummies - pragmatic encounter on azure"
header:
 teaser: "assets/images/serverless-for-dummies/teaser.jpg"
tags:
  - serverless
  - c#
  - dotnet
  - .net
  - .net core
  - azure
---

## Here I am without the server

Serverless in on the rise. Gaining a lot of traction to push new abililties, features and there is more and more hosting/deployment options.

It is not only bound to public cloud providers but it's also getting more popular on on-premise environments where you can tune it to your needs.
This is backed up by frameworks like OpenFAAS, Kubeless, which are build on top of kubernetes and open source world.  

> Main idea of serverless is to not worry about infrastructure technicalities (no server) and instead focus on solving real problems/writing code.

Serverless code generally is short-living and needs to be staless, meaning state should be put into/handled by external system, database, etc.
Thus it is harder/trickier to do small statefull things like connection pooling (i.e. connections pooling to rmds databases).

<figure class="align-center" style="width: 75%">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/serverless-for-dummies/pros.jpeg" alt="">
  <figcaption>Best place for servers is in the trash.</figcaption>
</figure>
<hr/>

When thinking about it in context of cloud platforms (azure, aws), payment model is very interesting:

- pay as you go
- scale as you go

On other hand there are some limitations:

- limited message size
- limited active connection time(only request reply)
- limited number of concurrent connections

## But What can I build ?

> Typical question is what I can build it with it and answer to that is typical `it depends` or `everything`.

Most of the time serverless is used as a additional layer on top of typical server-full apps.
It fullfils role to solve little problems, threat serverless function as single purpose endpoint that solves one problem.
Then compose those little problem-solvers (functions) together to create something `bigger`.

## Not another hello world example

Let's create something that might be useful.
> Function that will accept as a parameter `id of spotify playlist` and return out of it list of songs together with youtube link.

Here is the final result:

<figure class="align-center" style="width: 80%">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/serverless-for-dummies/app.jpeg" alt="">
  <figcaption>Final look of the app in the browser.</figcaption>
</figure>

You can experience that by executing:

[https://spotifytranslatorapp.azurewebsites.net/api/map?playlist=37i9dQZF1DXcBWIGoYBM5M](https://spotifytranslatorfunctionapp.azurewebsites.net/api/map?playlist=2GK6j1Wh1NfmIPcFVXC97t){:target="_blank"}

Where:

- `playlist=` is a id of spotify playlist

Please note i will focus on local development, thus i won't go throgh `azure portal` and creating/managing/deploying stuff there.  
Never the less this of course can be deployed to cloud without a problem.
{: .notice--warning}

## What we need ?

We will use azure, below are prerequisites:

- Setup `.net core` (version 3.1 >= required).
- Install `Azure Functions Core Tools` from [here.](https://docs.microsoft.com/pl-pl/azure/azure-functions/functions-run-local){:target="_blank"}
- We will need to connect to api of spotify and youtube. Since only limited usage of these api's is free, we need to create `youtube app` and `spotify app` so our usage can be tracked.

## Scafolding

```bash
  mkdir SpotifyToYoutubeTranslator && cd $_
  func init --worker-runtime dotnet
  func new --language C# --name SpotifyToYoutubeTranslator
```

![full](/assets/images/serverless-for-dummies/serverless-setup.gif){: .full}

## Code walkthrough

Entire code is available [here.](https://github.com/UnderNotic/spotify-to-youtube-translator){:target="_blank"}
It's splitted into 3 major sections:

![full](/assets/images/serverless-for-dummies/diagram.jpeg){: .full}

## Before we begin

Youtube/spotify application keys need to be stored somewhere safe. Those are secrets that authenticate our app, that can not be compromised.
We will use simple approach, which is `local.settings.json`.

Cloud version is using `azure application settings`, as secrets from `local.settings.json` are of course not commited.
{: .notice--warning}

Then in code these will be read as follow.
```csharp
private static readonly string SPOTIFY_APP_KEY =
    System.Environment.GetEnvironmentVariable("SPOTIFY_APP_KEY");
private static readonly string YOUTUBE_APP_KEY =
    System.Environment.GetEnvironmentVariable("YOUTUBE_APP_KEY");
```

If something more advanced is needed, have a look at `azure key-vault`.
It is ment to store secrets but with more features on top of it :)
{: .notice--info}

## <b style="color: #ffce2f">Section #1</b>

First we need to know what playlist we want to take from spotify.
This is easy as getting `playlist id` from incoming reuqest query parameters.

```csharp
var playlist = req.Query["playlist"];
 ```

If there is no cached spotify token or it expired, a new one is obtained and cached.

```csharp
private static async Task<SpotifyItems> GetSpotifyItems(HttpRequest req, string playlist)
{
    if (_spotifyToken == null || _spotifyToken.ExpiryDate < DateTime.UtcNow)
    {
        var fullToken = await GetSpotifyAccessToken();
        _spotifyToken = new SpotifyToken(fullToken["access_token"].ToString(), Int32.Parse(fullToken["expires_in"].ToString()));
    }
    var token = _spotifyToken.Token;
```

To obtain fresh token, a `POST` request with spotify app key has to be send to spotify api.

```csharp
private static async Task<dynamic> GetSpotifyAccessToken()
{
    var tokenHeaders = new FormUrlEncodedContent(new[]{
        new KeyValuePair<string, string>("grant_type", "client_credentials")
    });

    var tokenRequest = new HttpRequestMessage(HttpMethod.Post, "https://accounts.spotify.com/api/token");
    tokenRequest.Content = tokenHeaders;
    tokenRequest.Headers.Add("Authorization", SPOTIFY_APP_KEY);

    var tokenRequestResponse = await _httpClient.SendAsync(tokenRequest);

    return await tokenRequestResponse.Content.ReadAsAsync<dynamic>();
}
```

## <b style="color: #0036e9">Section #2</b>

Secondly, We need actual playlist of songs, given above `playlist id`.
For that, `GET` request is executed. In the header We pass spotify app token from previous section.

Query parameters contain items, that are going to be returned back by spotify api.
In this case we would have:

- name of a track
- artist/s
- name of an album

```csharp
    private static async Task<SpotifyItems> GetSpotifyItems(HttpRequest req, string playlist)
    {
        if (_spotifyToken == null || _spotifyToken.ExpiryDate < DateTime.UtcNow)
        {
            var fullToken = await GetSpotifyAccessToken();
            _spotifyToken = new SpotifyToken(fullToken["access_token"].ToString(), Int32.Parse(fullToken["expires_in"].ToString()));
        }
        var token = _spotifyToken.Token;

        var playlistQuery = HttpUtility.ParseQueryString(string.Empty);
        playlistQuery["fields"] = "items(track(name,artists, album(name))), next";

        string playlistQueryString = playlistQuery.ToString();
        var playlistRequest = new HttpRequestMessage(HttpMethod.Get, $"https://api.spotify.com/v1/playlists/{playlist}/tracks?{playlistQueryString}");
        playlistRequest.Headers.Add("Authorization", $"Bearer {token}");

        var playlistRequestResponse = await _httpClient.SendAsync(playlistRequest);
        playlistRequestResponse.EnsureSuccessStatusCode();

        var spotifyItems = await playlistRequestResponse.Content.ReadAsAsync<SpotifyItems>();
        return spotifyItems;
    }
```

## <b style="color: #ff1b1b">Section #3</b>

Finally, for each song from the playlist, we need to have link to youtube video and return it as a final response.
To do so we call Youtube with `GET` request.
For this request no token is needed, we just set the `app key` in query parameters.

Also in query params we provide:

- `q`, search phrase `{artist name - track name}`
- `type` of a resource, hardcoded to `video`
- `part`, we want to get as a result, hardcoded to `id`
- `maxResults`, we are interested only in the first video matching

```csharp
var resultItems = spotifyItems.Items.Select(async track =>
{
    var youtubeItems = await GetYoutubeItems(track);

    return new ResultItem(
        track.Track.Name,
        track.Track.Artists[0].Name,
        track.Track.Album.Name,
        youtubeItems.Items[0].Id.VideoId
    );
});

return (ActionResult)new OkObjectResult(await Task.WhenAll(resultItems));
```

```csharp
private static async Task<YoutubeItems> GetYoutubeItems(SpotifyItem track)
{
    var youtubeRequest =
        new HttpRequestMessage(HttpMethod.Get, $"https://www.googleapis.com/youtube/v3/search?q={track.Track.Artists[0].Name}-{track.Track.Name}&type=video&part=id&maxResults=1&key={YOUTUBE_APP_KEY}");

    var youtubeRequestResponse = await _httpClient.SendAsync(youtubeRequest);
    var youtubeItems = await youtubeRequestResponse.Content.ReadAsAsync<YoutubeItems>();
    return youtubeItems;
}
```

Final result is composed, by taking `ids` returned by api and putting them as `v` query param into youtube video link.

```csharp
public ResultItem(string name, string artist, string album, string ytId)
{
    Name = name;
    Artist = artist;
    Album = album;
    Url = $"https://www.youtube.com/watch?v={ytId}";
}
```

## Important note - HttpClient is in a static field

We don't want to create `HttpClient` for each request. Rather it should be long-lived resource, this will lower memory footpring and it will benefit from reusing tcp connections.

Important thing to remebmer is that, there won't be one and only instance of `HttpClient`, instead each instance of a function runtime will have single `HttpClient` instance.

For more info about `HttpClient` check [this blog post.](https://deaddesk.top/http-client-explained-with-netstat/){:target="_blank"}
{: .notice--info}

## Running locally

You can run it locally by cloning [repo](https://github.com/UnderNotic/spotify-to-youtube-translator){:target="_blank"} and creating config file `local.settings.json`.

```json
  "Values": {
        "AzureWebJobsStorage": "",
        "FUNCTIONS_WORKER_RUNTIME": "dotnet",
        "SPOTIFY_APP_KEY": "Basic YourSpotifyAppKey",
        "YOUTUBE_APP_KEY": "YourYoutubeAppKey"
    }
```

Then just run `func start` and call [http://localhost:7071/api/map?playlist=YourSpotifyPlaylistId](http://localhost:7071/api/map?playlist=YourSpotifyPlaylistId){:target="_blank"} 

Have fun :)
