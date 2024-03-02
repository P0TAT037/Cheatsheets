# ASP.NET Web API

It's simply a type of console applications which listens to HTTP requests, processes it, then respond with the result as an HTTP response.

## Create a WebApplication

to start listening for the incoming HTTP requests, we need to spin up an http web server that listens on a certain port, and we need to implement the logic that will process requests and generate the response depending on the shape and type of the request.

Asp.net provides us with a facade class called WebApplication that provides us with function and properties that makes it easier for us to set our configure our application's "settings" to suite our needs.

in our app there's an object of type `IHost`, we will call it the host, that contains basically the components the application needs to function.

one of these components (and probably the most important) is the web server that will listen for the requests on the network.

### The Web Server (Kestrel)

the web server is responsible for listening for http requests, on receiving one it encapsulates it into an `HttpContext` object which is a type that contains all the info about the incoming request and it's response, then it calls a `RequestDelegate`, which is basically an async function that returns a `Task` passing the `HttpContext` object to it. the `RequestDelegate` calls a series of other `RequestDelegate`s known as the "Middleware Pipeline" that take the `HttpContext` object one by one and process the request and modifies the object if it needs to. by the end of the pipeline the `HttpContext` object is in a state that can represent the response of the request.

> Note that all the Middleware pipeline and request processing in not a part of nor the role of the web server, it all happens outside. the web server only passes the `HttpContext` object to the `RequestDelegate` and waits for the `Task` to complete.

the web server keeps a reference to the `HttpContext` it sends to the `RequestDelegate`, once the Task returned by the `RequestDelegate` is completed, the web server creates the Http response from the `HttpContext` that is by now processed and modified and sends that response back to the client over the network. This is the main role of the web server, dealing with the network.

The web server implementation in Asp.net is called Kestrel, it's a cross-platform web server that is used by Asp.net to listen for incoming HTTP requests.
Asp.net can also use other web servers like IIS, but Kestrel is the default one.

To use other third party web servers like IIS, a bridge is used to intercept the incoming requests to that the 3rd party web server and pass them to our asp.net application as `HttpContext` and pass it to the `RequestDelegate` then return the result from the processed `HttpContext` after the `RequestDelegate` finishes to the web server. The bridge must be supported by both the 3rd party web server and Asp.net.

## HTTP Request Pipeline

### Middlewares

Middlewares are small "programs" or "functions" that are put in order and executed sequentially, each one of those receives the request as a parameter, it can do whatever it wants with it then it either passes this processed/modified/unchanged(if it wants) request to the next middleware in the sequence/pipeline (by calling it and passing the modified request reference), and since it's a function call our middleware can also do some processing after this function returns, or it can stop/short circuit the pipeline and return immediately.

After receiving an Http request it's represented in the `HttpContext` object, then it goes through a pipeline of middlewares, each middleware can do some processing and pass the request to the next middleware as we said. After passing by every middleware in the pipeline, the request processing is done and the response by then is constructed and sent back to the client.

### Creating a middleware

### Important Middlewares

### Action Filters

### Authorization Filters

### Result Filters
