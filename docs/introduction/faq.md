---
title: Frequently asked questions (FAQ) about Blazor
author: guardrex
description: Find the answers to frequently asked questions about Blazor.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 04/16/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/introduction/faq
---
# Frequently asked questions (FAQ) about Blazor

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

## What is Blazor?

Blazor is a single-page web app framework built on .NET that runs in the browser with WebAssembly.

## I'm new to .NET. What's .NET?

[.NET](https://www.microsoft.com/net) is a free, cross-platform, open source developer platform for building many different types of apps (desktop, mobile, games, web). .NET includes a managed runtime, a standard set of [libraries](https://docs.microsoft.com/dotnet/api), and support for multiple modern programming languages: [C#](https://docs.microsoft.com/dotnet/csharp/), [F#](https://docs.microsoft.com/dotnet/fsharp/), and [VB](https://docs.microsoft.com/dotnet/visual-basic/). You can [get started with .NET in 10 min](https://www.microsoft.com/net/learn/get-started/windows).

## Why would I use .NET for web development?

Using .NET in the browser offers many advantages that can help make web development easier and more productive:

* **Stable and consistent**: .NET offers standard APIs, tools, and build infrastructure across all .NET platforms that are stable, feature rich, and easy to use.
* **Modern innovative languages**: .NET languages like [C#](https://docs.microsoft.com/dotnet/csharp/) and [F#](https://docs.microsoft.com/dotnet/fsharp/) make programming a joy and keep getting better with innovative new language features.
* **Industry leading tools**: The [Visual Studio](https://www.visualstudio.com/) product family provides a great .NET development experience on Windows, Linux, and macOS.
* **Fast and scalable**: .NET has a long history of performance, reliability, and security on the server. Using .NET as a full-stack solution makes it easier to scale your apps.

## How can you run .NET in a web browser?

Running .NET in the browser is made possible by a relatively new standardized web technology called [WebAssembly](http://webassembly.org/). WebAssembly is a "portable, size- and load-time-efficient format suitable for compilation to the web." Code compiled to WebAssembly can run in any browser at native speeds. To run .NET binaries in a web browser, we use a .NET runtime (specifically [Mono](http://www.mono-project.com/news/2017/08/09/hello-webassembly/)) that has been compiled to WebAssembly.

## Does Blazor compile my entire .NET-based app to WebAssembly?

No, a Blazor app consists of normal compiled .NET assemblies that are downloaded and run in a web browser using a WebAssembly-based .NET runtime. Only the .NET runtime itself is compiled to WebAssembly. That said, support for full [static ahead of time (AoT) compilation](http://www.mono-project.com/news/2018/01/16/mono-static-webassembly-compilation/) of the app to WebAssembly may be something we add further down the road.

## Wouldn't the app download size be huge if it also includes a .NET runtime?

Not necessarily. .NET runtimes come in all shapes in sizes. Early Blazor prototypes used a compact .NET runtime (including assembly execution, garbage collection, threading) that compiled to a mere 60KB of WebAssembly. Blazor now runs on Mono, which is currently significantly larger. However, opportunities for size optimization abound, including merging and trimming the runtime and application binaries. Other potential download size mitigations include caching and using a CDN.

## What features will Blazor support?

Blazor will support all of the features of a modern single-page app framework:

* A component model for building composable UI
* Routing
* Layouts
* Forms and validation
* Dependency injection
* JavaScript interop
* Live reloading in the browser during development
* Server-side rendering
* Full .NET debugging both in browsers and in the IDE
* Rich IntelliSense and tooling
* Ability to run on older (non-WebAssembly) browsers via asm.js
* Publishing and app size trimming

## Can I use Blazor without running .NET on the server?

Yes, a Blazor app can be deployed as a set of static files without the need for any .NET support on the server.

## Can I use Blazor with ASP.NET Core on the server?

Yes! Blazor optionally integrates with [ASP.NET Core](https://docs.microsoft.com/aspnet/core) to provide a seamless and consistent full-stack web development solution.

## Is Blazor a .NET port of an existing JavaScript framework?

Blazor is a *new framework* inspired by existing modern single-page app frameworks, such as React, Angular, and Vue.

## How can I try out Blazor?

To build your first Blazor web app check out our [getting started guide](xref:client-side/blazor/get-started).

## Why is Blazor an "experimental" project?

Blazor is an experimental project because there are still many questions to answer about its viability and appeal. The purposes of this initial experimental phase are to:

* Work through technical issues.
* Gauge interest and to listen to feedback.

We're optimistic about Blazor's future; but at this time, Blazor isn't a committed product and should be considered pre-alpha.

## Is this Silverlight all over again?

No, Blazor is a .NET web framework based on HTML and CSS that runs in the browser using open web standards. It doesn't require a plugin, and it works on mobile devices and older browsers.

## Does Blazor use XAML?

No, Blazor is a web framework based on HTML, CSS, and other standard web technologies.

## Is WebAssembly supported in all browsers?

Yes, WebAssembly has achieved cross-browser consensus and [all modern browsers now support WebAssembly](https://caniuse.com/#search=webassembly).

## Does Blazor work on mobile browsers?

Yes, [modern mobile browsers also support WebAssembly](https://caniuse.com/#search=webassembly).

## What about older browsers that don't support WebAssembly? For example, does Blazor work in IE?

For older browsers that don't support WebAssembly, Blazor falls back to using an asm.js-based .NET runtime. Using asm.js is slower and has a larger download size but is still quite functional.

## Can I use .NET Standard libraries with Blazor?

Yes, the .NET runtime used for Blazor supports .NET Standard 2.0. APIs that aren't supported in the browser throw *Not Supported* exceptions.

## Don't you need features like garbage collection and threading added to WebAssembly to make this work?

No, WebAssembly in its current state is sufficient. The .NET runtime handles its own garbage collection and threading concerns.

## Can I use existing JavaScript libraries with Blazor?

Yes, Blazor apps can call into JavaScript through JavaScript interop APIs.

## Can I access the DOM from a Blazor app?

You can access the DOM through JavaScript interop from .NET code. However, Blazor is a component-based framework that minimizes the need to access the DOM directly.

## Why Mono? Why not .NET Core or CoreRT?

[Mono](http://www.mono-project.com/) is a Microsoft-sponsored open source implementation of the .NET Framework. Mono is used by [Xamarin](https://www.xamarin.com/) to build native client apps for Android, iOS, and macOS. Xamarin is also used by [Unity](https://unity3d.com/) for game development. Microsoft's Xamarin team [announced their plans](http://www.mono-project.com/news/2017/08/09/hello-webassembly/) to add support for WebAssembly to Mono, and they've been making steady progress ([Mono and WebAssembly - Updates on Static Compilation 1/16/2018](http://www.mono-project.com/news/2018/01/16/mono-static-webassembly-compilation/)). Because Blazor is a client-side web UI framework targeted at WebAssembly, Mono is a natural fit.

By comparison, [.NET Core](https://www.microsoft.com/net/learn/get-started/windows) is primarily used for server apps and for cross-platform console apps. .NET Core can be used for creating an ASP.NET Core backend for a Blazor app but not for building the client app itself. [CoreRT](https://github.com/dotnet/corert) is a .NET Core runtime optimized for AoT compilation; and while it has WebAssembly aspirations, the project is still a work-in-progress and not a shipping product.

## Where did the name "Blazor" come from?

Blazor makes heavy use of [Razor](https://docs.microsoft.com/aspnet/core/mvc/views/razor?view=aspnetcore-2.1), a markup syntax for HTML and C#. **Browser + Razor = Blazor!** When pronounced, it's also the name of a swanky jacket worn by hipsters that have excellent taste in fashion, style, and programming languages.
