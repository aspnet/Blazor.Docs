---
title: Get started with Blazor
author: danroth27
description: Learn how to get started with the Blazor framework.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 04/11/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/get-started
---
# Get started with Blazor

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

To get set up with Blazor:

1. Install the [.NET Core 2.1 Preview 2 SDK](https://www.microsoft.com/net/download/dotnet-core/sdk-2.1.300-preview2).
1. Install the latest *preview* of [Visual Studio 2017 (15.7)](https://www.visualstudio.com/vs/preview) with the *ASP.NET and web development* workload.
   *Note:* You can install Visual Studio previews side-by-side with an existing Visual Studio installation without impacting your existing development environment.
1. Install the [ASP.NET Core Blazor Language Services extension](https://go.microsoft.com/fwlink/?linkid=870389) from the Visual Studio Marketplace.

To create your first project from Visual Studio:

1. Select **File** > **New Project** > **Web** > **ASP.NET Core Web Application**.
1. Make sure **.NET Core** and **ASP.NET Core 2.0** are selected at the top.
1. Choose the Blazor template.

   ![New Blazor app dialog](https://msdnshared.blob.core.windows.net/media/2018/03/new-blazor-app-dialog.png)
   
1. Press **Ctrl-F5** to run the app *without the debugger*. Running with the debugger (F5) isn't supported at this time.

If you're not using Visual Studio, you can install the Blazor templates from the command-line:

```console
dotnet new -i Microsoft.AspNetCore.Blazor.Templates
dotnet new blazor -o MyBlazorApp
cd MyBlazorApp
dotnet run
```

Congrats! You just ran your first Blazor app!

![Blazor app home page](https://msdnshared.blob.core.windows.net/media/2018/03/blazor-home.png)
