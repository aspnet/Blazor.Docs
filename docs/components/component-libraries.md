---
title: Component libraries in Blazor
author: stimms
description: Discover how Blazor components can be included in a Blazor app from an external component library and how to create a library.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 12/14/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/components/component-libraries
---
# Component libraries in Blazor

By [Simon Timms](https://github.com/stimms)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]
## Component libraries

Components may be included from external libraries which can be another project in the solution, a NuGet package or a referenced .NET library. Just as Blazor components are regular .NET types, Blazor component libraries are normal .NET assemblies. 

To create a new component library use the `blazorlib` template which is part of the templates installed when setting up Blazor.

```
dotnet new blazorlib -o BlazorComponentLib1
```

To add the library to an existing project use the `sln` command on `dotnet`

```
dotnet sln add .\BlazorComponentLib1
```

Component libraries may contain static files such as images, JavaScript and CSS. At build time these will be embedded in the built DLL allowing consumption of the components without having to worry about how to include their resources. Any files included in the `content` directory will be marked as an embedded resource. 

### Consuming a library component

In order to consume components defined in a library in another project the [@addTagHelper](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro?view=aspnetcore-2.1#add-helper-label) directive must be used. Individual components may be added by including

```
@addTagHelper BlazorComponentLib1.Component1,BlazorComponentLib1
```

or more generally

```
@addTagHelper <namespaced component name>,<assembly name>
```

However, it is more common to include all the components from an assembly using a wildcard

```
@addTagHelper *,BlazorComponentLib1
```

The `@addTagHelper` directive can be included in `_ViewImport.cshtml` to make them available for an entire project or scoped to a single page. With the `@addTagHelper` directive in place the components can now be consumed as if they were in the same assembly. 

### Building, packing and shipping to NuGet

Because Blazor component libraries are standard .NET libraries packaging and shipping them to NuGet is no different from packaging and shipping any library to NuGet. Packaging is done using the [pack](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-pack?tabs=netcore2x) command

```
dotnet pack
```

The package can then be uploaded to NuGet using the [publish](https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-nuget-push?tabs=netcore21) command

```
dotnet nuget publish
```

Any included static resources will be included in the NuGet package so consumers are not required to manually install them. Library consumers will automatically include JavaScript and CSS - no additional steps are necessary.