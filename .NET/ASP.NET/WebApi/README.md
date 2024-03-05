# ASP.NET Web API

It's simply a type of console applications which listens to HTTP requests, processes it, then respond with the result as an HTTP response.

## Create a WebApplication

to start listening for the incoming HTTP requests, we need to spin up an http web server that listens on a certain port, and we need to implement the logic that will process requests and generate the response depending on the shape and type of the request.

Asp.net provides us with a facade class called `WebApplication` that provides us with functions and properties that makes it easier for us to configure our application's "settings" to suite our needs.

in our app there's an object of type `IHost`, we will call it the host, it basically contains the components the application needs to function.

one of these components (and probably the most important one) is the web server that will listen for the requests on the network.

### The Web Server (Kestrel)

the web server is responsible for listening for http requests, on receiving one it encapsulates it into an `HttpContext` object which is a type that contains all the info about the incoming request and it's response, then it calls a `RequestDelegate`, which is basically an async function that returns a `Task` passing the `HttpContext` object to it. the `RequestDelegate` calls a series of other `RequestDelegate`s internally, this is known as the "Middleware Pipeline", that take the `HttpContext` object one by one and process the request and might modify the `HttpContext` object.

> Note that all the Middleware pipeline and request processing in not a part of nor the role of the web server, it all happens outside. the web server only passes the `HttpContext` object to the `RequestDelegate` and waits for the `Task` to complete.

the web server keeps a reference to the `HttpContext` it sends to the `RequestDelegate`, once the Task returned by the `RequestDelegate` is complete, the web server creates the Http response from the `HttpContext` (which is by now processed and modified) and sends that response back to the client over the network. This is the main role of the web server, dealing with the network.

The web server implementation in Asp.net is called Kestrel, it's a cross-platform web server that is used by Asp.net to listen for incoming HTTP requests.
Asp.net can also use other web servers like IIS, but Kestrel is the default one.

To use other third party web servers like IIS, a bridge is used to intercept the incoming requests to that the 3rd party web server and pass them to our asp.net application as `HttpContext` and pass it to the `RequestDelegate` then return the result from the processed `HttpContext` after the `RequestDelegate` finishes to the web server. The bridge must be supported by both the 3rd party web server and Asp.net.

## Configuring the WebApplication

The `WebApplication` object that we mentioned earlier represents our "Application" that we want to create and configure to suite our needs.

First We user the `CreateBuilder()` Method inside `WebApplication` class to get a `WebApplicationBuilder` object.

```csharp
var builder = WebApplication.CreateBuilder();
```

The builder allows us to add configuration sources and services

### configuration

Configuration is a set of variables with values that can be used throughout the program to apply some settings. For example if we want to set a database connection string we can put it in a variable called "ConnectionString" in the configuration. and if we needed to change the database we can just change that variable.

the builder allows us to add multiple sources to read the configuration from. one of the most popular sources are json files.

ASP.NET reads configuration by default from some different sources like

- appsettings.json (created by default)
- Environment variables
- command line arguments

To add another source we call the suitable `Add` method in our `builder.Configuration`

For example to add another json file

```csharp
builder.Configuration.AddJson(@"path/to/json/file");
```

Configuration variables can later be accessed using `builder.Configuration.GetSection("S1:V1")` or `builder.Configuration["S1:V1"]`

> where "S1" is a json object that has a variable "V1" inside it.

We can also create a class to be the model of our configuration and binding an instance of it to the configuration file.

```csharp
builder.Configuration.Bind(ConfigModelInstance);
```

> where ConfigModelInstance is an instance of a model class that represents the json object used for configuration.

### Services

A service is basically a class that contains logic and can be instantiated.

Our host that we talked about earlier contains a service container. a service container maintains the services Interfaces, concrete types, and their life time

let's say we have a service `Service` that implements `IService` we can register it in the services container in one of these ways

```csharp
var InstantOfService = new Service(); 
builder.Services.Singleton(InstanceOfService);
builder.Services.AddTransient<IService, Service>();
builder.Services.Scoped<IService, Service>();
```

> Note: this way we can easily change the implementation of a service seamlessly without breaking anything only by providing another implementation to the `IService` interface.

Each function of those register the service with a different lifetime.

Then when a service is requested the container resolves it's dependencies and instantiates an object (if required).

#### Service Lifetimes

- **Singleton**: the service is instantiated once and retrieved each time the service is requested.

- **Scoped**: the service is instantiated once for every incoming request and the same instance is used each time it's requested during the whole journey of the http request processing pipeline.

- **Transient**: a new instance of the service is created for every time it's requested.

After setting up our configuration settings and registering our required services we can call the `Build()` method to get our configured `WebApplication` object.

```csharp
var app = builder.Build();
```

the `WebApplication` object allows us to set the Middleware pipeline of the request.

To register a middleware we can do

```csharp
app.Use(async (context, next) =>
{
    var stopwatch = new Stopwatch();
    stopwatch.Start();
    
    await next.Invoke();
    
    stopwatch.Stop();
    Console.WriteLine($"Request [{context.Request.Path}] took [{stopwatch.Elapsed.Milliseconds}ms]");
});
```

Or

```csharp
app.UseMiddleware<ProfilingMiddleware>();
```

where `ProfilingMiddleware` is a class that has a function `Invoke`/`InvokeAsync` that takes the `HttpContext` as a parameter and the `RequestDelegate` of the next middleware in it's constructor

```csharp
public class ProfilingMiddleware
{
    private readonly RequestDelegate _next;
    public ProfilingMiddleware(RequestDelegate next)
    {
        _next = next;
    }


    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = new Stopwatch();
        stopwatch.Start();
        
        await _next(context);
        
        stopwatch.Stop();
        Console.WriteLine($"Request [{context.Request.Path}] took [{stopwatch.Elapsed.Milliseconds}ms]");
    }
}
```

> Here this middleware starts a stop watch and calls the request delegate of the next middleware in the chain, which will call the rest of the middlewares to the last one, then after it returns the time elapsed to process the request is printed in the console.

> Note that to get accurate results from this middleware it needs to be registered the first in the chain so it starts the timer before all the middlewares start and stops it after they all finish.

You might be wondering how will we get the `next` request delegate and the `HttpContext` and where the class is even instantiated and called!!

After finishing registering our Middleware pipeline we call the `Run` method

```csharp
app.Run();
```

this method prompts the host to start, which starts the services configured for our program. it takes responsibility of instantiating the needed classes and providing the parameters needed in their constructor.

Actually one of the ways (and the most popular one) of getting an instance of a service we registered in the services container, is by adding it as a parameter in the class constructor, and when the host tries to instantiate that class it will provide it with an instance of the service from the container. This is called dependency injection.

## Traditional Configuration of the WebApplication

When creating a new asp.net web api project with the default settings you'll get a project template that has one endpoint that serves pseudo wether forecast data.

you'll find that `Program.cs` has two parts

In the first part the builder is created and the services are registered then the application is built.

```c#
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();
```

There's three built-in service are being registered

- **`AddControllers()`** : Adds built-in services required for handling requests by a controller.

- **`AddEndpointsApiExplorer()`** : Adds services necessary for discovering and storing Api endpoints info especially when using minimal apis.

- **`AddSwaggerGen()`** : Adds services required by swagger to run properly.

In the second part, after we `Build()` the builder, we configure the middleware pipeline then we call `Run()` to start the services so the app starts listening for requests and responding to them.

```csharp
// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers();

app.Run();
```

The registered middlewares here are

- `UseSwagger()`: this middleware intercepts the requests coming to `swagger/v1/swagger.json/yaml` and returns a json/yaml that describes all the registered endpoints.

- `UseSwaggerUI()` : this middleware intercepts the requests coming for `swagger/index.html` or `swagger/` and returns a page from which you can explore, try and tinker with the registered endpoints (and it needs the `UseSwagger()` middleware to be registered in order to work).

- `UseHttpsRedirection()` : this one intercepts the requests if they are coming through `http` and returns a redirection response to the same endpoint but with `https`.

- `UseAuthorization()` : this makes sure the endpoint being requested is authorized to be accessed by this user. using the authorization policies set up in the first part (if any) while adding services (the `AddAuthorization()` service).

The we got `MapControllers()` : this one maps the incoming request to the correct controller action that can handle it. in our case in this template project we have `Controllers/WeatherForecastController.cs` that has the controller class with a route attribute describing the route that should be mapped to it and inside it we have a function (action) that has an attribute describing the http method that it's mapped to it. so when a request comes in this form `GET https://[hostname:port]/weatherforecast` it will be mapped to our action inside WeatherForecastController.

### Controllers?

we mentioned controllers a couple of times in the previous section, but what are they?

controllers are classes that implement the abstract class `ControllerBase` (in our case) or `Controller` (in the MVC case) which is a class that inherits from `ControllerBase` and adds some more functionality. The `ControllerBase`/`Controller` class has some helper methods and properties that makes it easier to access the request properties, handle the request and generate the response.

Controller classes have the attribute `[ApiController]` which tells the framework that this class is a controller and it should be treated as such. and the `[Route]` attribute that tells the framework that this class should be mapped to the route specified in the attribute as mentioned earlier .
> How to write the route pattern is out of the scope of this cheatsheet.

The function inside the controller class are called actions and they might also have a `[Route]`, they are the functions that will be called when a request comes to the route that the controller is mapped to.

## HTTP Request Pipeline

### Middlewares

Middlewares are small "programs" or "functions" that are put in order and executed sequentially, each one of those receives the request as a parameter, it can do whatever it wants with it then it either passes this processed/modified/unchanged(if it wants) request to the next middleware in the sequence/pipeline (by calling it and passing the modified request reference), and since it's a function call our middleware can also do some processing after this function returns, or it can stop/short circuit the pipeline and return immediately.

After receiving an Http request it's represented in the `HttpContext` object, then it goes through a pipeline of middlewares, each middleware can do some processing and pass the request to the next middleware as we said. After passing by every middleware in the pipeline, the request processing is done and the response by then is constructed and sent back to the client.

### Important Middlewares

### Action Filters

### Authorization Filters

### Result Filters
