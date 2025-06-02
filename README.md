# Briar

[![NuGet Stable](https://img.shields.io/nuget/v/BlossomiShymae.Briar.svg?style=flat-square&logo=nuget&logoColor=black&labelColor=69ffbe&color=77077a)](https://www.nuget.org/packages/BlossomiShymae.Briar/) [![NuGet Downloads](https://img.shields.io/nuget/dt/BlossomiShymae.Briar?style=flat-square&logoColor=black&labelColor=69ffbe&color=77077a)](https://www.nuget.org/packages/BlossomiShymae.Briar/)

Briar is a wrapper for the League Client and Game Client APIs which are unofficially provided by Riot Games.

This library is currently compatible with .NET 8 and higher for Windows and MacOS.

## Contributors

[![](https://contrib.rocks/image?repo=BlossomiShymae/Briar)](https://github.com/BlossomiShymae/Briar/graphs/contributors)

## Usage

### Installation

```bash
dotnet install BlossomiShymae.Briar
```

### Sample application

[A demonstration of Briar with more code examples can be found here.](https://github.com/BlossomiShymae/Briar/blob/main/BlossomiShymae.Briar.Demo/Program.cs)

To run the demo, clone the repo and then do:
```bash
dotnet run --project BlossomiShymae.Briar.Demo
```

### Requesting the LCU

This library uses the `System.Net.Http.HttpClient` interface via `LcuHttpClient`. It comes with a built-in handler that takes care of the request path and authorization header.

> [!NOTE]
> The built-in handler will attempt to refresh the port and auth if a `HttpRequestException` is encountered **on request**. `InvalidOperationException` will be thrown if the port of the current LCU process cannot be found or the port cannot be connected to.


#### Getting the LcuHttpClient instance

```csharp
var client = Connector.GetLcuHttpClientInstance();
```

#### General request

```csharp
var response = await client.SendAsync(new(HttpMethod.Get, "/lol-summoner/v1/current-summoner"));

var me = await response.Content.ReadFromJsonAsync<Summoner>();
```

#### GET request

```csharp
var me = await client.GetFromJsonAsync<Summoner>("/lol-summoner/v1/current-summoner");
```

#### POST request with JSON body

```csharp
var response = await client.PostAsJsonAsync("/player-notifications/v1/notifications", playerNotificationResource);

var resource = await response.Content.ReadFromJsonAsync<PlayerNotificationResource>();
```

> [!WARNING]
> `ProcessFinder.IsActive()` does not necessarily mean that the LCU process port is open for requests. Use `ProcessFinder.IsPortOpen()` instead.

#### Utilities

```csharp
var leagueClientProcess = ProcessFinder.GetProcess();
var processInfo = ProcessFinder.GetProcessInfo();
var isActive = ProcessFinder.IsActive();
var isPortOpen = ProcessFinder.IsPortOpen();

var riotAuthentication = new RiotAuthentication(processInfo.RemotingAuthToken);
```

### Requesting the Game Client

#### Getting the GameHttpClient instance

```csharp
var client = Connector.GetGameHttpClientInstance();
```

#### General request

```csharp
var response = await client.SendAsync(new(HttpMethod.Get, "/liveclientdata/activeplayername"));

var me = await response.Content.ReadFromJsonAsync<string>();
```

#### GET request

```csharp
var me = await client.GetFromJsonAsync<string>("/liveclientdata/activeplayername");
```

### LCU WebSocket

This library uses the `Websocket.Client` wrapper.

Create a client:

```csharp
var client = Connector.CreateLcuWebsocketClient();
```

Listen to events, disconnections, or reconnection messages:

```csharp
using System; // Include to avoid compiler errors CS1503, CS1660.
              // You may or may not need this.

client.EventReceived.Subscribe(msg =>
{
    Console.WriteLine(msg?.Data?.Uri);
});
client.DisconnectionHappened.Subscribe(msg => 
{
    if (msg.Exception != null) throw msg.Exception;
});
client.ReconnectionHappened.Subscribe(msg =>
{
    Console.WriteLine(msg.Type);
});
```

Use it:

```csharp
// This starts the client in a background thread. You will need an event loop
// to listen to messages.
await client.Start();

// Subscribe to every event that the League Client sends.
var message = new EventMessage(EventRequestType.Subscribe, EventKinds.OnJsonApiEvent);
client.Send(message);

// We will need an event loop for the background thread to process.
while(true) await Task.Delay(TimeSpan.FromSeconds(1));
```
