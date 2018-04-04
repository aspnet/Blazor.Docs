---
title: Blazor layouts
author: rstropek
description: Learn how to create reusable layout components for Blazor apps.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 04/09/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/layouts
---
# Blazor layouts

By [Rainer Stropek](https://www.timecockpit.com)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

Blazor apps typically contain more than one page. Layout elements, such as menus, copyright messages, and logos, must be present on all pages. Copying the code of these layout elements onto all of the pages of an app isn't an efficient solution. Such duplication is hard to maintain and probably leads to inconsistent content over time. *Blazor layouts* solve this problem.

## What are Blazor layouts?

Technically, a layout is just another Blazor component. A layout is defined in a Razor template or in C# code and can contain data binding, dependency injection, and other ordinary features of components. Two additional aspects are special and turn a Blazor component into a Blazor layout:

* The layout component must implement `Microsoft.AspNetCore.Blazor.Layouts.ILayoutComponent`. This interface adds a property `Body` to the component, which contains the content to be rendered inside the layout.
* The layout component contains the Razor keyword `@Body`. During rendering, it's replaced by the content of the layout.

The following code sample shows the Razor template of a layout component. Note the use of `ILayoutComponent` and `@Body`:

```csharp
@implements ILayoutComponent

<header>
    <h1>ERP Master 3000</h1>
    <p>Current user: @UserName</p>
</header>

<nav>
    <a href="/master-data">Master Data Management</a>
    <a href="/invoicing">Invoicing</a>
    <a href="/accounting">Accounting</a>
</nav>

@Body

<footer>
    &copy; by Acme Corp
</footer>

@functions {
    public string UserName { get; set; }
    public RenderFragment Body { get; set; }
}
```

## Use a layout in a Blazor component

Use the Razor keyword `@layout` to apply a layout to a Blazor component. The Blazor compiler turns this keyword into the `Microsoft.AspNetCore.Blazor.Layouts.LayoutAttribute` attribute, which is applied to the Blazor component class.

The following code sample demonstrates the concept. The content of this component is inserted into the *MasterLayout* at the position of `@Body`:

```csharp
@layout MasterLayout

@page "/master-data"

<h2>Master Data Management</h2>
...
```

## Centralized layout selection

Every folder of a Blazor app can optionally contain a template file named *_ViewImports.cshtml*. The Blazor compiler includes the content of the view imports file in all of the Razor templates in the same folder and recursively in all of its subfolders. Therefore, a *_ViewImports.cshtml* file containing `@layout MainLayout` ensures that all Blazor components in a folder use the *MainLayout* layout. There's no need to repeatedly add `@layout` to all *\*.cshtml* files.

Note that the default template for Blazor apps uses the *_ViewImports.cshtml* mechanism for layout selection. A newly created Blazor app contains the relevant *_ViewImports.cshtml* file in the *Pages* folder.

## Nested layouts

Blazor apps can consist of nested layouts. That means that a Blazor component can reference a layout which in turn references another layout. For example, nesting layouts can be used to reflect a multi-level menu structure.

The following code samples show how to use nested layouts. The *CustomersComponent.cshtml* file is the Blazor component to display. Note that the component references the layout `MasterDataLayout`.

*CustomersComponent.cshtml*:

```csharp
@layout MasterDataLayout

@page "/master-data/customers"

<h1>Customer Maintenance</h1>
...
```

The *MasterDataLayout.cshtml* file provides the `MasterDataLayout`. The layout references another layout `MainLayout` in which it's going to be embedded.

*MasterDataLayout.cshtml*:

```csharp
@layout MainLayout
@implements ILayoutComponent

<nav>
    <!-- Menu structure of master data module -->
    ...
</nav>

@Body

@functions {
    public RenderFragment Body { get; set; }
}
```

Finally, `MainLayout` contains the top-level layout elements, such as the header, footer, and main menu.

*MainLayout.cshtml*:

```csharp
@implements ILayoutComponent

<header>...</header>
<nav>...</nav>

@Body

@functions {
    public RenderFragment Body { get; set; }
}
```
