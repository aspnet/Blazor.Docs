---
title: Blazor hosting models
author: danroth27
description: Understand client-side and server-side Blazor hosting models
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 07/24/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/hosting-models
---
# Blazor hosting models

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

Blazor is a client-side web framework designed primarily to run in the browser on a WebAssembly based .NET runtime. Blazor supports both multiple hosting models, including out-of-process hosting models, where the Blazor component logic runs separately from the UI thread. But, regardless of the hosting model, the Blazor app and component model remains *the same*. This article discusses the available hosting models for Blazor.

## Client-side (in-process) hosting model

The principal hosting model for Blazor is running client-side in the browser. In this model, the Blazor app, it's dependencies, and the .NET runtime are all downloaded to the browser and the app is executed directly on the browser UI thread (we may also support running Blazor in a Web Worker at some future point). All UI updates and event handling happens within the same process. The app assets can be hosted as static files using whatever web server is preferred (see [Publishing](xref:client-side/blazor/publishing/index)).

![Blazor client-side](https://user-images.githubusercontent.com/1874516/43042852-998bb680-8d3b-11e8-9d39-adf8d3d77360.png)

To create a Blazor app using the client-side hosting model, use the "Blazor" or "Blazor (ASP.NET Core Hosted)" project templates ("blazor" or "blazorhosted" when using `dotnet new` on the command-line). The included *blazor.webassembly.js* script handles downloading the .NET runtime, the app, and its dependencies and then initializes the runtime to run the app. 

In the "Blazor" stand-alone template a simple web host is provided during development and the production web server is left to the user to provide. In the "Blazor (ASP.NET Core Hosted)" template an ASP.NET Core app is used to host the Blazor client-side app enabling full-stack .NET web development.

The benefits of the client-side hosting model are:
- No .NET server-side dependency
- Rich interactive UI
- Fully leverage the client resources and capabilities
- Offload work from the server to the client
- Support offline scenarios

The downsides to the client-side hosting model are:
- Restricted to the capabilities of the browser
- Requires more capable client hardware and software (ex WebAssembly support)
- Larger download size and app load time
- Less mature .NET runtime and tooling support (ex. limitations in .NET Standard support, debugging, etc.)

## Server-side hosting model

In the server-side hosting model Blazor is executed on the server using .NET Core. UI updates, event handling, and JavaScript calls are all handled over a SignalR connection.

![Blazor server-side](https://user-images.githubusercontent.com/1874516/43042867-eaa8bb76-8d3b-11e8-8f1d-60768f86f710.png)

To create a Blazor app using the server-side hosting model, use the "Blazor (Server-side on ASP.NET Core)" template ("blazorserver" when using `dotnet new` from the command-line). An ASP.NET Core app hosts the Blazor server-side app and sets up the SignalR endpoint for clients to connect to. The Blazor app project uses the *blazor.server.js* script to establish the client connection. It is the app's responsibility to persist and restore app state as needed (ex in the event of a lost network connection).

The benefits of the server-side hosting model are:
- You can still write your entire app with .NET and C# using the Blazor component model.
- Your app still has a rich interactive feel and avoids unnecessary page refreshes.
- Your app download size is significantly smaller and the initial app load time is much faster.
- Your Blazor component logic can take full advantage of server capabilities including using any .NET Core compatible APIs.
- Because you're running on .NET Core on the server existing .NET tooling, like debugging, just works.
- Works with thin clients (ex browsers that don't support WebAssembly, resource constrained devices, etc.).

The downsides of the server-side hosting model are:
- Latency: every user interaction now involves a network hop.
- No offline support: if the client connection goes down the app stops working.
- Scalability: the server must manage multiple client connections and handle client state.