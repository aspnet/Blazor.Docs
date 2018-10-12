---
title: Blazor hosting models
author: danroth27
description: Understand client-side and server-side Blazor hosting models.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 07/24/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/host-and-deploy/hosting-models
---
# Blazor hosting models

By [Daniel Roth](https://github.com/danroth27)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

Blazor is a client-side web framework designed primarily to run in the browser on a WebAssembly-based .NET runtime. Blazor supports multiple hosting models, including out-of-process hosting models, where the Blazor component logic runs separately from the UI thread. Regardless of the hosting model, the Blazor app and component models remain *the same*. This article discusses the available hosting models for Blazor.

## Client-side hosting model

The principal hosting model for Blazor is running client-side in the browser. In this model, the Blazor app, its dependencies, and the .NET runtime are downloaded to the browser, and the app is executed directly on the browser UI thread. All UI updates and event handling happens within the same process. The app assets can be deployed as static files using whatever web server is preferred (see [Host and deploy](xref:client-side/blazor/host-and-deploy/index)).

![Blazor client-side](https://user-images.githubusercontent.com/1874516/43042852-998bb680-8d3b-11e8-9d39-adf8d3d77360.png)

To create a Blazor app using the client-side hosting model, use the **Blazor** or **Blazor (ASP.NET Core Hosted)** project templates (`blazor` or `blazorhosted` template when using [dotnet new](/dotnet/core/tools/dotnet-new) at a command prompt). The included *blazor.webassembly.js* script handles:

* Downloading the .NET runtime, the app, and its dependencies.
* Initialization of the runtime to run the app.

The benefits of the client-side hosting model are:

* No .NET server-side dependency
* Rich interactive UI
* Fully leverage the client resources and capabilities
* Offload work from the server to the client
* Support offline scenarios

The downsides to the client-side hosting model are:

* Restricted to the capabilities of the browser
* Requires more capable client hardware and software (for example, WebAssembly support)
* Larger download size and app load time
* Less mature .NET runtime and tooling support (for example, limitations in .NET Standard support and debugging)

> [!NOTE]
> Visual Studio includes the **Blazor (ASP.NET Core hosted)** project template for creating a Blazor app that runs on WebAssembly and is hosted on an ASP.NET Core server. The ASP.NET Core app serves the Blazor app to clients but is otherwise a separate process. The client-side Blazor app can interact with the server over the network using Web API calls or SignalR connections.

> [!IMPORTANT]
> If a client-side Blazor app is served by an ASP.NET Core app hosted as an IIS sub-app, it's important to disable the inherited ASP.NET Core Module handler. Remove the handler in the Blazor app's published *web.config* file by adding a `<handlers>` section to the file:
>
> ```xml
> <handlers>
>   <remove name="aspNetCore" />
> </handlers>
> ```
>
> Set the app base path in the Blazor app's *index.html* file to the IIS alias used when configuring the sub-app in IIS. For more information, see [App base path](xref:client-side/blazor/host-and-deploy/index#app-base-path).

## Server-side hosting model

In the server-side hosting model, Blazor is executed on the server from within an ASP.NET Core app. UI updates, event handling, and JavaScript calls are handled over a SignalR connection.

![Blazor server-side](https://user-images.githubusercontent.com/1874516/43042867-eaa8bb76-8d3b-11e8-8f1d-60768f86f710.png)

To create a Blazor app using the server-side hosting model, use the **Blazor (Server-side on ASP.NET Core)** template (`blazorserver` when using [dotnet new](/dotnet/core/tools/dotnet-new) at a command prompt). An ASP.NET Core app hosts the Blazor server-side app and sets up the SignalR endpoint where clients connect. The ASP.NET Core app references the Blazor `Startup` class to both add the server-side Blazor services and to add the Blazor app to the request handling pipeline:

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        ...
        services.AddServerSideBlazor<App.Startup>();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        ...
        app.UseServerSideBlazor<App.Startup>();
    }
}
```

The *blazor.server.js* script establishes the client connection. It's the app's responsibility to persist and restore app state as needed (for example, in the event of a lost network connection).

The benefits of the server-side hosting model are:

* You can still write your entire app with .NET and C# using the Blazor component model.
* Your app still has a rich interactive feel and avoids unnecessary page refreshes.
* Your app download size is significantly smaller, and the initial app load time is much faster.
* Your Blazor component logic can take full advantage of server capabilities, including using any .NET Core compatible APIs.
* Because you're running on .NET Core on the server, existing .NET tooling, such as debugging, works as expected.
* Server-side hosting works with thin clients (for example, browsers that don't support WebAssembly and resource constrained devices).

The downsides of the server-side hosting model are:

* Latency: every user interaction now involves a network hop.
* No offline support: if the client connection goes down, the app stops working.
* Scalability: the server must manage multiple client connections and handle client state.
