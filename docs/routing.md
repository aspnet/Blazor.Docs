---
title: Blazor routing
author: guardrex
description: Learn how to route requests in a client-side Blazor app and about the NavLink component.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 05/01/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/routing
---
# Blazor routing

By [Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

Learn how to route requests in a client-side Blazor app and about the NavLink component.

[View or download sample code](https://github.com/aspnet/Blazor.Docs/tree/master/docs/common/samples/) ([how to download](xref:client-side/blazor/index#view-and-download-samples)). See the [Get started](xref:client-side/blazor/get-started) topic for prerequisites.

## Route templates

The **&lt;Router&gt;** component enables routing, and a route template is provided to each accessible component. The **&lt;Router&gt;** component appears in the *App.cshtml* file:

```cshtml
<Router AppAssembly=typeof(Program).Assembly />
```

When a *\*.cshtml* file with an `@page` directive is compiled, the generated class is given a [RouteAttribute](https://docs.microsoft.com/dotnet/api/microsoft.aspnetcore.mvc.routeattribute) specifying the route template. At runtime, the router looks for component classes with a `RouteAttribute` and renders whichever component has a route template that matches the requested URL.

Multiple route templates can be applied to a component. In the [sample app](https://github.com/aspnet/Blazor.Docs/tree/master/docs/common/samples/), the following component responds to requests for `/BlazorRoute` and `/DifferentBlazorRoute`.

*Pages/BlazorRoute.cshtml*:

[!code-cshtml[](common/samples/2.x/BlazorSample/Pages/BlazorRoute.cshtml?start=1&end=4)]

## Route parameters

The Blazor client-side router uses route parameters to populate the corresponding component parameters with the same name (case insensitive).

*Pages/RouteParameter.cshtml*:

[!code-cshtml[](common/samples/2.x/BlazorSample/Pages/RouteParameter.cshtml?start=1&end=8)]

Optional parameters aren't supported yet, so two `@page` directives are applied in the example above. The first permits navigation to the component without a parameter. The second `@page` directive takes the `{text}` route parameter and assigns the value to the `Text` property.

## Route constraints

A route constraint enforces type matching on a route segment to a Blazor component.

In the following example, the route to the Users component only matches if:

* An `Id` route segment is present on the request URL.
* The `Id` segment is an integer (`int`).

```cshtml
@page "/Users/{Id:int}"

<h1>The user Id is @Id!</h1>

@functions {
    [Parameter]
    private int Id { get; set; }
}
```

For more information and a list of available route constraints, see [ASP.NET Core Routing: Route constraint reference](https://docs.microsoft.com/aspnet/core/fundamentals/routing#route-constraint-reference).

## NavLink component

Use a NavLink component in place of HTML **\<a>** elements when creating navigation links. A NavLink component behaves like an **\<a>** element, except it toggles an `active` CSS class based on whether its `href` matches the current URL. The `active` class helps a user understand which page is the active page among the navigation links displayed.

The NavMenu component in the [sample app](https://github.com/aspnet/Blazor.Docs/tree/master/docs/common/samples/) creates a [Bootstrap](https://getbootstrap.com/docs/) nav bar that demonstrates how to use NavLink components. The following markup shows the first two NavLinks in the *Shared/NavMenu.cshtml* file.

[!code-cshtml[](common/samples/2.x/BlazorSample/Shared/NavMenu.cshtml?start=13&end=24&highlight=4-6,9-11)]

There are two [NavLinkMatch](/api/Microsoft.AspNetCore.Blazor.Routing.NavLinkMatch.html) options:

* `NavLinkMatch.All` &ndash; Specifies that the NavLink should be active when it matches the entire current URL.
* `NavLinkMatch.Prefix` &ndash; Specifies that the NavLink should be active when it matches any prefix of the current URL.

In the preceding example, the Home NavLink (`href=""`) matches all URLs and always receives the `active` CSS class. The second NavLink only receives the `active` class when the user visits the BlazorRoute component (`href="BlazorRoute"`).
