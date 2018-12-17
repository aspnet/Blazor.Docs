---
title: Component libraries in Blazor
author: stimms
description: Discover how Blazor components can be included in a Blazor app from an external component library and how to create a library.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 12/17/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/component-libraries
---
# Component libraries in Blazor

By [Simon Timms](https://github.com/stimms)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

Blazor components can be shared in component libraries across projects. Components can be included from:

* Another project in the solution.
* A NuGet package.
* A referenced .NET library.

Just as Blazor components are regular .NET types, Blazor component libraries are normal .NET assemblies. 

To create a new component library, use the `blazorlib` template with the [dotnet new](https://docs.microsoft.com/dotnet/core/tools/dotnet-new) command. The template is part of the templates installed when [setting up Blazor](/docs/get-started.html#setup).

```console
dotnet new blazorlib -o BlazorComponentLib1
```

To add the library to an existing project, use the [dotnet sln](https://docs.microsoft.com/dotnet/core/tools/dotnet-sln) command:

```console
dotnet sln add .\BlazorComponentLib1
```

Component libraries may contain static files, such as images, JavaScript, and stylesheets. At build time, static files are embedded into the built assembly file (*.dll*), which allows consumption of the components without having to worry about how to include their resources. Any files included in the `content` directory are marked as an embedded resource. 

## Consume a library component

In order to consume components defined in a library in another project, the [@addTagHelper](https://docs.microsoft.com/aspnet/core/mvc/views/tag-helpers/intro#add-helper-label) directive must be used. Individual components may be added by name. For example, the following directive adds `Component1` of `BlazorComponentLib1`:

```cshtml
@addTagHelper BlazorComponentLib1.Component1, BlazorComponentLib1
```

The general format of the directive is:

```cshtml
@addTagHelper <namespaced component name>, <assembly name>
```

However, it's common to include all of the components from an assembly using a wildcard:

```cshtml
@addTagHelper *, BlazorComponentLib1
```

The `@addTagHelper` directive can be included in `_ViewImport.cshtml` to make the components available for an entire project or scoped to a single page or set of pages within a folder. With the `@addTagHelper` directive in place, the components of the component library can be consumed as if they were in the same assembly as the app. 

## Build, pack, and ship to NuGet

Because Blazor component libraries are standard .NET libraries, packaging and shipping them to NuGet is no different from packaging and shipping any library to NuGet. Packaging is performed using the [dotnet pack](https://docs.microsoft.com/dotnet/core/tools/dotnet-pack) command:

```console
dotnet pack
```

Upload the package to NuGet using the [dotnet nuget publish](https://docs.microsoft.com/dotnet/core/tools/dotnet-nuget-push) command:

```console
dotnet nuget publish
```

Any included static resources are included in the NuGet package. Library consumers automatically receive scripts and stylesheets, so consumers aren't required to manually install the resources.
