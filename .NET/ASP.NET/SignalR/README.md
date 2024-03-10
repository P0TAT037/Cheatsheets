# SignalR

SignalR is a library for ASP.NET developers that makes it easy to add real-time web functionality to your applications. What is "real-time web" functionality? It's the ability to have your server-side code push content to the connected clients as it happens, in real-time, this can be done using WebSockets, Server-Sent Events, or Long Polling.

## Intro

SignalR have something called "Hubs" a hub is a class that implements the `Hub` abstract class in which you can define methods that can be called by the client. you can think of a hub as a special kind of controller, you can add `[Authorize]` attribute to it or it's functions or the hub itself to enable authorization. and you can add filters that implement `IHubFilter` interface. (the authorization runs before the filters in case both of them exist)

The `Hub` abstract class contains properties that allow us to interact with the client/caller/connection like:

- **`Context` :** through this you can access the `HttpContext`, `User`, `ConnectionId`, and have methods to `Abort` the connection and other methods.
- **`Clients` :** through this you can access the connected clients to send them a message or the caller. or a certain client with it's ConnectionId.
- **`Groups` :** through this we can add clients/users to a groups and interact with these groups.

the `Hub` also has two virtual methods `OnConnectedAsync()` and `OnDisconnectedAsync()` that run on client connection and on connection termination respectively.

## How to use

- First we define our hub class

```csharp
using Microsoft.AspNetCore.SignalR;

namespace SignalRChat.Hubs
{
    public class ChatHub : Hub
    {
        public async Task SendMessage(string user, string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", user, message);
        }
    }
}
```

- In `Program.cs` we add the SignalR service

```csharp
builder.Services.AddSignalR();
```

- after `app.Build()` we map the hub to a certain path/pattern

```csharp
app.MapHub<ChatHub>("/chatHub");
```

now that's the server side and our hub is ready to be used.

----

On the client side depending on the client we can use it's SignalRClient library to connect to the hub and call it's methods.

For example if we are using a C# client we can use the `Microsoft.AspNetCore.SignalR.Client` library to connect to the hub and call it's methods.

```csharp
using Microsoft.AspNetCore.SignalR.Client;

var connection = new HubConnectionBuilder()
    .WithUrl("http://localhost:5000/chatHub")
    .Build();

connection.On<string, string>("ReceiveMessage", (user, message) =>
{
    Console.WriteLine($"{user}: {message}");
});

await connection.StartAsync();

await connection.InvokeAsync("SendMessage", user, message);
```

In the previous code a `HubConnection` is created and the `On` Method is used to register a delegate to be executed When receiving a message `ReceiveMessage` and we give it the types of the expected incoming parameters with the message as type parameters from the server. and then the `StartAsync` method is called to start the connection, once the connection is started we can start sending messages to the server to invoke the methods of the hub. In this example we use `InvokeAsync` and give it the method name as a string with it's parameters to prompt the server to run its `SendMessage` method with these parameters.

## How it works

SignalR uses several protocols to achieve real time communication, these protocols are:

- **WebSockets :** a full-duplex communication protocol that works over a single TCP connection, it's the most efficient protocol for real-time communication.

- **Server-Sent Events :** a standard that allows servers to push data to the client once an event occurs, it's a one way communication from the server to the client.

- **Long Polling :** a technique that the client sends a request to the server and the server holds the request until it has new data to send to the client, then it sends the response to the client and the client sends a new request to the server.

WebSockets is the one that is used by default if it's supported by the client and the server, if it's not supported SignalR will try to use Server-Sent Events, and if it's not supported it will use Long Polling.

## SignalR Hub Protocol

SignalR has it's own protocol of messaging over these protocols,  it defines the message types, formats and how it should be sent and replied to between the client and the server.

Messages can be encoded either in JSON format or in a binary format called MessagePack, the format is agreed upon in the connection establishment between the client and the server.

Here are the steps of connection establishment and communication between the client and the server:

- After opening a connection (the WebSocket connection) to the server the client must send a `HandshakeRequest` message to the server as its first message. The handshake message is always a JSON message and contains the name of the format (protocol) as well as the version of the protocol that will be used for the duration of the connection. The server will reply with a `HandshakeResponse`, also always JSON, containing an error if the server does not support the protocol. If the server does not support the protocol requested by the client or the first message received from the client is not a `HandshakeRequest` message the server must close the connection. Both the `HandshakeRequest` and `HandshakeResponse` messages must be terminated by the ASCII character 0x1E (record separator).

- After the handshake is successful the client and the server can both send messages to each other. There are three types of messages:

- The client can send `Invocation` messages to the server containing the name of the method to be invoked and the arguments to be passed to the method.

- The server replies with either a `Completion` message or a `StreamItem` message. The `Completion` message is sent in response to an `Invocation` message and it contains the result of the method if it returns one result or doesn't if it doesn't contain any result. The `StreamItem` message is sent in response to an `Invocation` message and contains a single item from a stream of results, for example if the invoked method is returning an `IAsyncEnumerable`.

- Both the server and the client can send `Close` messages to each other to close the connection.

You can check the full documentation of the SignalR Hub Protocol and all the message types [here](https://github.com/dotnet/aspnetcore/blob/main/src/SignalR/docs/specs/HubProtocol.md?WT.mc_id=dotnet-35129-website#signalr-hub-protocol).

----

### *That's it, now you know.*
