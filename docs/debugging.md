---
title: Blazor debugging
author: danroth27
description: Learn how to debug Blazor apps.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 07/25/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/debugging
---
# Blazor debugging

[Daniel Roth](https://github.com/danroth27)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

Blazor has some *very early* support for debugging client-side Blazor apps running on WebAssembly in Chrome. While this initial debugging support is very limited and unpolished it does show the basic debugging infrastructure coming together.

To debug your client-side Blazor app in Chrome:
- Build a Blazor app in `Debug` configuration (the default for non-published apps)
- Run the Blazor app in Chrome
- With the keyboard focus on the app (not in the dev tools, which you should probably close as it's less confusing that way), press the following Blazor specific hotkey:
  - Shift+Alt+D on Windows/Linux
  - Shift+Cmd+D on macOS

You need to run Chrome with remote debugging enabled to debug your Blazor app. If you don't, you will get an error page with instructions for running Chrome with the debugging port open so that the Blazor debugging proxy can connect to it. You will need to *close all Chrome instances* and then restart Chrome as instructed.

![Blazor debugging error page](https://user-images.githubusercontent.com/1874516/43123091-01ec0796-8ed8-11e8-844c-23b4e6e9d069.png)

Once you have Chrome running with remote debugging enabled, hitting the debugging hotkey will open a new debugger tab. After a brief moment the *Sources* tab will show a list of the .NET assemblies in the app. You can expand each assembly and find the *.cs*/*.cshtml* source files you want to debug. You can then set breakpoints, switch back to your app's tab, and cause the breakpoints to be hit. You can then single-step (F10) or resume (F8).

![Blazor debugging](https://user-images.githubusercontent.com/1874516/43123060-efb0b3b0-8ed7-11e8-9ea5-97aa34247a0b.png)

How does this work? Blazor provides a debugging proxy that implements the [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/) and augments the protocol with .NET specific information. When you hit the debugging hotkey, Blazor points the Chrome DevTools at the proxy, which in turn connects to the browser window you are trying to debug (hence the need for enabling remote debugging).

You might be wondering why we don't just use browser source maps. Source maps allow the browser to map compiled files back to their original source files. However, Blazor does not map C# directly to JS/WASM (at least not yet). Instead, Blazor does IL interpretation within the browser, so source maps are not relevant.

NOTE: The debugger capabilities are **very limited.** You can currently only:
- Single-step through the current method (F10) or resume (F8)
- In the *Locals* display, observe the values of any local variables of type `int`/`string`/`bool`
- See the call stack, including call chains that go from JavaScript into .NET and vice-versa

That's it! You *cannot* step into child methods (i.e., F11), observe the values of any locals that aren't an `int`/`string`/`bool`, observe the values of any class properties or fields, hover over variables to see their values, evaluate expressions in the console, step across async calls, or do basically anything else. Work on the long tail of debugger features remains a significant ongoing task.
