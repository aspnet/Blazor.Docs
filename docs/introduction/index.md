---
title: Introduction to Blazor
author: guardrex
description: Learn how Blazor runs in the browser to execute C#/Razor code with WebAssembly and the Mono runtime in this introduction.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 04/09/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/introduction/index
---
# Introduction to Blazor

By [Steve Sanderson](http://blog.stevensanderson.com), [Daniel Roth](https://github.com/danroth27), and [Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

Blazor is an experimental .NET web framework using C#/Razor and HTML that runs in the browser with WebAssembly. Blazor provides all of the benefits of a client-side web UI framework using .NET on both the server and the client.

## Why use .NET for client-side apps?

Web development has improved in many ways over the years, but building modern web apps still poses challenges. Using .NET in the browser offers many advantages that can help make web development easier and more productive: 

* **Stability and consistency**: .NET provides standardized programming frameworks across platforms that are stable, feature-rich, and easy to use.
* **Modern innovative languages**: .NET languages are constantly improving with innovative new language features.
* **Industry-leading tools**: The Visual Studio product family provides a fantastic .NET development experience across platforms on Windows, Linux, and macOS.
* **Speed and scalability**: .NET has a strong history of performance, reliability, and security for app development. Using .NET as a full-stack solution makes it easier to build fast, reliable, and secure apps.
* **Full-stack development that leverages existing skills**: C#/Razor developers use their existing C#/Razor skills to write client-side code and share server and client-side logic among apps.
* **Wide browser support**: Blazor runs on .NET using open web standards in the browser with no plugins and no code transpilation. It works in all modern web browsers, including mobile browsers.

## Running .NET in the browser

Running .NET code inside web browsers is made possible by a relatively new technology, [WebAssembly](http://webassembly.org) (abbreviated *wasm*). WebAssembly is an open web standard and is supported in web browsers without plugins. WebAssembly is a compact bytecode format optimized for fast download and maximum execution speed.

WebAssembly code can access the full functionality of the browser via JavaScript interop. At the same time, WebAssembly code runs in the same trusted sandbox as JavaScript to prevent malicious actions on the client machine.

When a Blazor app is built and run in a browser:

1. C# code files and Razor files are compiled into .NET assemblies.
1. The assemblies and the .NET runtime are downloaded to the browser.
1. Blazor uses JavaScript to bootstrap the .NET runtime and configures the runtime to load required assembly references. Document object model (DOM) manipulation and browser API calls are handled by the Blazor runtime via JavaScript interoperability.

## Browsers that don't support WebAssembly

The .NET runtime is supplied as a WebAssembly binary and an [asm.js](https://wikipedia.org/wiki/Asm.js)-based implementation. *asm.js* is a subset of JavaScript and can be executed by JavaScript runtimes in browsers going back several years. When Blazor loads in the browser, it checks for WebAssembly support. If WebAssembly isn't supported, the *asm.js* runtime is loaded. *asm.js* isn't always used because it's larger and slower than the WebAssembly runtime.

## Blazor components

Blazor apps are built with *components*. A component is a piece of UI, such as a page, dialog, or data entry form. Components can be nested, reused, and shared between projects.

In Blazor, a component is a .NET class. The class can either be written directly, as a C# class (*\*.cs*), or more commonly in the form of a Razor markup page (*\*.cshtml*).

[Razor](https://docs.microsoft.com/aspnet/core/mvc/views/razor) is a syntax for combining HTML markup with C# code. Razor is designed for developer productivity, allowing the developer to switch between markup and C# in the same file with [IntelliSense](https://docs.microsoft.com/visualstudio/ide/using-intellisense) support. The following markup is an example of a basic custom dialog component in a Razor file (*DialogComponent.cshtml*):

```cshtml
<div>
    <h2>@Title</h2>
    @RenderContent(Body)
    <button onclick=@OnOK>OK</button>
</div>

@functions {
    public string Title { get; set; }
    public Content Body { get; set; }
    public Action OnOK { get; set; }
}
```

When this component is used elsewhere in the app, IntelliSense speeds development with syntax completion and parameter info.

Components can be:

* Nested.
* Generated procedurally in code.
* Shared via class libraries.
* Unit tested without requiring a browser DOM.

## Infrastructure

Blazor offers the core facilities that most apps require, including:

* Layouts
* Routing
* Dependency injection
* Unit testing

All of these features are optional. When one of these features isn't used in an app, the implementation is stripped out of the app when published by the IL linker.

A few low-level elements are included in the framework. For example, routing and layouts aren't built-in. Routing and layouts are implemented in *user space*, code that an app developer can write without using internal APIs. These features can be replaced with different systems to suit the app's requirements. The current layouts prototype is implemented in about 30 lines of C# code, so a developer can understand and replace it if desired.

## Code sharing and .NET Standard

The [.NET Standard](https://docs.microsoft.com/dotnet/standard/net-standard) is a formal specification of .NET APIs that are intended to be available on all .NET implementations. Blazor supports .NET Standard 2.0 or higher. APIs that can't be supported due to browser limitations (for example, accessing the file system, opening a socket, threading, and others) throw [PlatformNotSupportedException](https://docs.microsoft.com/dotnet/api/system.platformnotsupportedexception). .NET Standard class libraries can be shared across server code and in browser-based apps.

## JavaScript/TypeScript interop

For apps that require third-party JavaScript libraries and browser APIs, WebAssembly is designed to interoperate with JavaScript. Blazor is capable of using any library or API that JavaScript is able to use. JavaScript code can call into C# (for example, to handle an event).

## Optimization

For client-side apps, payload size is critical. Blazor optimizes payload size to reduce download times. For example, unused IL code is removed when a Blazor app is built, and response compression is utilized.

## Deployment

Developers have the option of using Blazor for only client-side development or for full-stack .NET development. Full-stack development offers many advantages&mdash;client- and server-side development uses the same tooling, build infrastructure, and language. Code can be shared between client and server apps.

For ASP.NET Core, [middleware](https://docs.microsoft.com/aspnet/core/fundamentals/middleware) offers an easy path to serve a Blazor UI seamlessly from an ASP.NET Core app. Equally important are developers who don't yet use ASP.NET Core. To make Blazor a viable consideration for developers using Node.js, Rails, PHP, or even for serverless web apps, ASP.NET Core isn't required on the server. When a Blazor app is built, a *dist* directory is produced containing nothing but static files. The contents of the *dist* folder can be hosted on the Azure CDN, GitHub Pages, Node.js servers, and many other servers and services.
