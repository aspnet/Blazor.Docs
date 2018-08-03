---
title: Blazor components
author: guardrex
description: Learn how to create and use Blazor components, the fundamental building blocks of Blazor apps provided by compiled Razor or C# files.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 04/16/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/components/index
---
# Blazor components

By [Luke Latham](https://github.com/guardrex) and [Daniel Roth](https://github.com/danroth27)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

[View or download sample code](https://github.com/aspnet/Blazor.Docs/tree/master/docs/common/samples/) ([how to download](xref:client-side/blazor/index#view-and-download-samples)). See the [Get started](xref:client-side/blazor/get-started) topic for prerequisites.

Blazor apps are built using *components*. A component is a self-contained chunk of user interface (UI), such as a page, dialog, or form. A component includes both the HTML markup to render along with the processing logic needed to inject data or respond to UI events. Components are flexible and lightweight, and they can be nested, reused, and shared between projects.

## Component classes

Blazor components are typically implemented in *\*.cshtml* files using a combination of C# and HTML markup. The UI for a component is defined using HTML. Dynamic rendering logic (for example, loops, conditionals, expressions) is added using an embedded C# syntax called [Razor](https://docs.microsoft.com/aspnet/core/mvc/views/razor). When a Blazor app is compiled, the HTML markup and C# rendering logic are converted into a component class. The name of the generated class matches the name of the file.

Members of the component class are defined in a `@functions` block (more than one `@functions` block is permissible). In the `@functions` block, component state (properties, fields) is specified along with methods for event handling or for defining other component logic.

Component members can then be used as part of the component's rendering logic using C# expressions that start with `@`. For example, a C# field is rendered by prefixing `@` to the field name. The following example evaluates and renders:

* `_headingFontStyle` to the CSS property value for `font-style`.
* `_headingText` to the content of the `<h1>` element.

```cshtml
<h1 style="font-style:@_headingFontStyle">@_headingText</h1>

@functions {
    private string _headingFontStyle = "italic";
    private string _headingText = "Put on your new Blazor!";
}
```

After the component is initially rendered, the component regenerates its render tree in response to events. Blazor then compares the new render tree against the previous one and applies any modifications to the browser's Document Object Model (DOM).

## Using components

Components can include other components by declaring them using HTML element syntax. The markup for using a component looks like an HTML tag where the name of the tag is the component type.

The following markup renders a `HeadingComponent` (*HeadingComponent.cshtml*) instance:

[!code-cshtml[](../common/samples/2.x/BlazorSample/Pages/Index.cshtml?start=11&end=11)]

## Component parameters

Components can have *component parameters*, which are defined using *non-public* properties on the component class decorated with `[Parameter]`. Use attributes to specify arguments for a component in markup.

In the following example, the `ParentComponent` sets the value of the `Title` property of the `ChildComponent`:

*ParentComponent.cshtml*:

[!code-cshtml[](../common/samples/2.x/BlazorSample/Pages/ParentComponent.cshtml?start=1&end=7&highlight=5)]

*ChildComponent.cshtml*:

[!code-cshtml[](../common/samples/2.x/BlazorSample/Pages/ChildComponent.cshtml?highlight=7-8)]

## Child content

Components can set the content in another component. The assigning component provides the content between the tags that specify the receiving component. For example, a `ParentComponent` can provide content that is to be rendered by a `ChildComponent` by placing the content inside **\<ChildComponent>** tags.

*ParentComponent.cshtml*:

[!code-cshtml[](../common/samples/2.x/BlazorSample/Pages/ParentComponent.cshtml?start=1&end=7&highlight=6)]

The child component has a `ChildContent` property that represents a [RenderFragment](/api/Microsoft.AspNetCore.Blazor.RenderFragment.html). The value of `ChildContent` is positioned in the child component's markup where the content should be rendered. In the following example, the value of `ChildContent` is received from the parent component and rendered inside the Bootstrap panel's `panel-body`.

*ChildComponent.cshtml*:

[!code-cshtml[](../common/samples/2.x/BlazorSample/Pages/ChildComponent.cshtml?highlight=3,10-11)]

> [!NOTE]
> The property receiving the `RenderFragment` content must be named `ChildContent` by convention.

## Data binding

Data binding to both components and DOM elements is accomplished with the `bind` attribute. The following example binds the `ItalicsCheck` property to the check box's checked state:

```cshtml
<input type="checkbox" class="form-check-input" id="italicsCheck" bind="@_italicsCheck" />
```

When the check box is selected and cleared, the property's value is updated to `true` and `false`, respectively.

The check box is updated in the UI only when the component is rendered, not in response to changing the property's value. Since components render themselves after event handler code executes, property updates are usually reflected in the UI immediately.

Using `bind` with a `CurrentValue` property (`<input bind="@CurrentValue" />`) is essentially equivalent to the following:

```cshtml
<input value="@CurrentValue" 
    onchange="@((UIValueEventArgs __e) => CurrentValue = __e.Value)" />
```

When the component is rendered, the `value` of the input element comes from the `CurrentValue` property. When the user types in the text box, the `onchange` is fired and the `CurrentValue` property is set to the changed value. In reality, the code generation is a little more complex because `bind` deals with a few cases where type conversions are performed. In principle, `bind` associates the current value of an expression with a `value` attribute and handles changes using the registered handler.

**Format strings**

Data binding works with [DateTime](https://docs.microsoft.com/dotnet/api/system.datetime) format strings (but not other format expressions at this time, such as currency or number formats):

```cshtml
<input bind="@StartDate" format-value="yyyy-MM-dd" />

@functions {
    [Parameter]
    private DateTime StartDate { get; set; } = new DateTime(2020, 1, 1);
}
```

The `format-value` attribute specifies the date format to apply to the `value` of the `input` element. The format is also used to parse the value when an `onchange` event occurs.

**Component parameters**

Binding also recognizes component parameters, where `bind-{property}` can bind a property value across components.

The following component uses `ChildComponent` and binds the `ParentYear` parameter from the parent to the `Year` parameter on the child component:

Parent component:

```cshtml
@page "/ParentComponent"

<h1>Parent Component</h1>

<p>ParentYear: @ParentYear</p>

<ChildComponent bind-Year="@ParentYear" />

<button class="btn btn-primary" onclick="@ChangeTheYear">Change Year to 1986</button>

@functions {
    [Parameter]
    private int ParentYear { get; set; } = 1978;

    void ChangeTheYear()
    {
        ParentYear = 1986;
    }
}
```

Child component:

```cshtml
<h2>Child Component</h2>

<p>Year: @Year</p>

@functions {
    [Parameter]
    private int Year { get; set; }

    [Parameter]
    private Action<int> YearChanged { get; set; }
}
```

The `Year` parameter is bindable because it has a companion `YearChanged` event that matches the type of the `Year` parameter.

Loading the `ParentComponent` produces the following markup:

```html
<h1>Parent Component</h1>

<p>ParentYear: 1978</p>

<h2>Child Component</h2>

<p>Year: 1978</p>
```

If the value of the `ParentYear` property is changed by selecting the button in the `ParentComponent`, the `Year` property of the `ChildComponent` is updated. The new value of `Year` is rendered in the UI when the `ParentComponent` is rerendered:

```html
<h1>Parent Component</h1>

<p>ParentYear: 1986</p>

<h2>Child Component</h2>

<p>Year: 1986</p>
```

## Event handling

Blazor provides event handling features. For an HTML element attribute named `on<event>` (for example, `onclick`, `onsubmit`) with a delegate-typed value, Blazor treats the attribute's value as an event handler. The attribute's name always starts with `on`.

The following code calls the `UpdateHeading` method when the button is selected in the UI:

```cshtml
<button class="btn btn-primary" onclick="@UpdateHeading">
    Update heading
</button>

@functions {
    void UpdateHeading(UIMouseEventArgs e)
    {
        ...
    }
}
```

The following code calls the `CheckboxChanged` method when the check box is changed in the UI:

```cshtml
<input type="checkbox" class="form-check-input" onchange="@CheckboxChanged" />

@functions {
    void CheckboxChanged()
    {
        ...
    }
}
```

For some events, event-specific event argument types are permitted. If access to one of these event types isn't necessary, it isn't required in the method call.

The list of supported event arguments is:

* UIEventArgs
* UIChangeEventArgs
* UIKeyboardEventArgs
* UIMouseEventArgs

Lambda expressions can also be used:

```cshtml
<button onclick="@(e => Console.WriteLine("Hello, world!"))">Say hello</button>
```

## Capturing references to elements

Some [JavaScript interop](xref:client-side/blazor/javascript-interop) scenarios require references to HTML elements. For example, a UI library may require an element reference for initialization, or you might need to call command-like APIs on an element like `focus` or `play`.

You can capture references to HTML elements in a component by adding a `ref` attribute to the HTML element and then defining a field of type `ElementRef` whose name matches the value of the `ref` attribute.

The following example shows capturing a reference to the username input element:

```
<input ref="username" ... />

@functions {
    ElementRef username;
}
```

> NOTE: Captured element references should **not** be used as a way of populating the DOM. Doing so may interfere with Blazor's declarative rendering model.

As far as .NET code is concerned, an `ElementRef` is an opaque handle. The *only* thing you can do with it is pass it through to JavaScript code via JavaScript interop. When you do so, the JavaScript-side code receives an `HTMLElement` instance which it can use with normal DOM APIs.

For example, you can define a .NET extension method that enables setting the focus on an element like this:

*mylib.js*
```js
window.myLib = {
  focusElement : function (element) {
    element.focus();
  }
}
```

*ElementRefExtensions.cs*
```csharp
using Microsoft.AspNetCore.Blazor;
using Microsoft.JSInterop;
using System.Threading.Tasks;

namespace MyLib
{
    public static class MyLibElementRefExtensions
    {
        public static Task Focus(this ElementRef elementRef)
        {
            return JSRuntime.Current.InvokeAsync<object>("myLib.focusElement", elementRef);
        }
    }
}
```

Now you can focus inputs in any of your components:

```html
@using MyLib

<input ref="username" />
<button onclick="@SetFocus">Set focus</button>

@functions {
    ElementRef username;

    void SetFocus()
    {
        username.Focus();
    }
}
```

*Important*: The `username` variable will only be populated after the component has rendered and its output includes the `<input>` element. If you try to pass an unpopulated `ElementRef` to JavaScript code, then the JavaScript code will receive `null`. To manipulate element references after the component has finished rendering (i.e. to set the initial focus on an element) use the `OnAfterRenderAsync` or `OnAfterRender` lifecycle methods.

## Capturing references to components

Component references provide a way get a reference to a component instance so that you can issue commands to that instance, like `Show` or `Reset`. To capture a component reference add  a `ref` attribute to the child component and then define a field with the same name and the same type as the child component.

```html
<MyLoginDialog ref="loginDialog" ... />

@functions {
    MyLoginDialog loginDialog;

    void OnSomething()
    {
        loginDialog.Show();
    }
}
```

When the component is rendered the `loginDialog` field is populated with the `MyLoginDialog` child component instance. You can then invoke .NET methods on the component instance.

*Important*: The `loginDialog` variable will only be populated after the component has rendered and its output includes the `MyLoginDialog` element, because until then there is nothing to reference. To manipulate components references after the component has finished rendering use the `OnAfterRenderAsync` or `OnAfterRender` lifecycle methods.

> NOTE: Component references should **not** be used to mutate the state of child components. Instead, always use normal declarative parameters to pass data to child components. This will cause child components to re-render at the correct times automatically.

## Lifecycle methods

`OnInitAsync` and `OnInit` execute code after the component has been initialized. To perform an asynchronous operation, use `OnInitAsync` and use the `await` keyword:

```csharp
protected override async Task OnInitAsync()
{
    await ...
}
```

For a synchronous operation, use `OnInit`:

```csharp
protected override void OnInit()
{
    ...
}
```

`OnParametersSetAsync` and `OnParametersSet` are called when a component has received parameters from its parent and the values are assigned to properties. These methods are executed after `OnInit` during component initialization.

```csharp
protected override async Task OnParametersSetAsync()
{
    await ...
}
```

```csharp
protected override void OnParametersSet()
{
    ...
}
```

`OnAfterRenderAsync` and `OnAfterRender` are called each time after a component has finished rendering. Element and component references are populated at this point. Use this stage to perform additional initialization steps using the rendered content, such as activating third-party JavaScript libraries that operate on the rendered DOM elements.

```csharp
protected override async Task OnAfterRenderAsync()
{
    await ...
}
```

```csharp
protected override void OnAfterRender()
{
    ...
}
```

`SetParameters` can be overridden to execute code before parameters are set:

```csharp
public override void SetParameters(ParameterCollection parameters)
{
    ...

    base.SetParameters(parameters);
}
```

If `base.SetParameters` isn't invoked, the custom code can interpret the incoming parameters value in any way required. For example, the incoming parameters aren't required to be assigned to the properties on the class.

`ShouldRender` can be overridden to suppress refreshing of the UI. If the implementation returns `true`, the UI is refreshed. Even if `ShouldRender` is overridden, the component is always initially rendered.

```csharp
protected override bool ShouldRender()
{
    var renderUI = true;

    return renderUI;
}
```

## Component disposal with IDisposable

If a component implements [IDisposable](https://docs.microsoft.com/dotnet/api/system.idisposable), the [Dispose method](https://docs.microsoft.com/dotnet/standard/garbage-collection/implementing-dispose) is called when the component is removed from the UI. The following component uses `@implements IDisposable` and the `Dispose` method:

```csharp
@using System
@implements IDisposable

...

@functions {
    public void Dispose()
    {
        ...
    }
}
```

## Routing

Routing in Blazor is achieved by providing a route template to each accessible component in the app.

When a *\*.cshtml* file with an `@page` directive is compiled, the generated class is given a [RouteAttribute](https://docs.microsoft.com/dotnet/api/microsoft.aspnetcore.mvc.routeattribute) specifying the route template. At runtime, the router looks for component classes with a `RouteAttribute` and renders whichever component has a route template that matches the requested URL.

Multiple route templates can be applied to a component. The following component responds to requests for `/BlazorRoute` and `/DifferentBlazorRoute`:

[!code-cshtml[](../common/samples/2.x/BlazorSample/Pages/BlazorRoute.cshtml?start=1&end=4)]

## Route parameters

Blazor components can receive route parameters from the route template provided in the `@page` directive. The Blazor client-side router uses route parameters to populate the corresponding component parameters.

*RouteParameter.cshtml*:

[!code-cshtml[](../common/samples/2.x/BlazorSample/Pages/RouteParameter.cshtml?start=1&end=9)]

Optional parameters aren't supported, so two `@page` directives are applied in the example above. The first permits navigation to the component without a parameter. The second `@page` directive takes the `{text}` route parameter and assigns the value to the `Text` property.

## Base class inheritance for a "code-behind" experience

Blazor component files (*\*.cshtml*) mix HTML markup and C# processing code in the same file. The `@inherits` directive can be used to provide Blazor with a "code-behind" experience that separates component markup from processing code.

The [sample app](https://github.com/aspnet/Blazor.Docs/tree/master/docs/common/samples/) shows how a component can inherit a base class, `BlazorRocksBase`, to provide the component's properties and methods.

*BlazorRocks.cshtml*:

[!code-cshtml[](../common/samples/2.x/BlazorSample/Pages/BlazorRocks.cshtml?start=1&end=8)]

*BlazorRocksBase.cs*:

[!code-csharp[](../common/samples/2.x/BlazorSample/Pages/BlazorRocksBase.cs)]

The base class should derive from [BlazorComponent](/api/Microsoft.AspNetCore.Blazor.Components.BlazorComponent.html).

## Razor support

**Razor directives**

Razor directives active with Blazor apps are shown in the following table.

| Directive | Description |
| --------- | ----------- |
| [@functions](https://docs.microsoft.com/aspnet/core/mvc/views/razor#functions) | Adds a C# code block to a component. |
| `@implements` | Implements an interface for the generated component class. |
| [@inherits](https://docs.microsoft.com/aspnet/core/mvc/views/razor#inherits) | Provides full control of the class that the component inherits. |
| [@inject](https://docs.microsoft.com/aspnet/core/mvc/views/razor#inject) | Enables service injection from the [service container](https://docs.microsoft.com/aspnet/core/fundamentals/dependency-injection). For more information, see [Dependency injection into views](https://docs.microsoft.com/aspnet/core/mvc/views/dependency-injection). |
| `@layout` | Specifies a layout component. Layout components are used to avoid code duplication and inconsistency. |
| [@page](https://docs.microsoft.com/aspnet/core/mvc/razor-pages#razor-pages) | Specifies that the component should handle requests directly. The `@page` directive can be specified with a route and optional parameters. Unlike Razor Pages, the `@page` directive doesn't need to be the first directive at the top of the file. For more information, see [Routing](xref:client-side/blazor/routing). |
| [@using](https://docs.microsoft.com/aspnet/core/mvc/views/razor#using) | Adds the C# `using` directive to the generated component class. |
| [@addTagHelper](https://docs.microsoft.com/aspnet/core/mvc/views/razor#tag-helpers) | Use `@addTagHelper` to use a component in a different assembly than the app's assembly. |

**Conditional attributes**

Blazor conditionally renders attributes based on the .NET value. If the value is `false` or `null`, Blazor won't render the attribute. If the value is `true`, the attribute is rendered minimized.

In the following example, `IsCompleted` determines if `checked` is rendered in the control's markup.

```cshtml
<input type="checkbox" checked="@IsCompleted" />

@functions {
    [Parameter]
    private bool IsCompleted { get; set; }
}
```

If `IsCompleted` is `true`, the check box is rendered as:

```html
<input type="checkbox" checked />
```

If `IsCompleted` is `false`, the check box is rendered as:

```html
<input type="checkbox" />
```

**Additional information on Razor**

For more information on Razor, see the [Razor syntax reference](https://docs.microsoft.com/aspnet/core/mvc/views/razor). Note that not all of the features of Razor are available in Blazor at this time.

## Raw HTML

Blazor normally renders strings using DOM text nodes, which means that any markup they may contain is ignored and treated as literal text. To render raw HTML, wrap the HTML content in a `MarkupString` value, which is then parsed as HTML or SVG and inserted into the DOM.

> [!WARNING]
> Rendering raw HTML constructed from any untrusted source is a **security risk** and should be avoided!

The following example shows using the `MarkupString` type to add a block of static HTML content to the rendered output of a component:

```html
@((MarkupString)myMarkup)

@functions {
    string myMarkup = "<p class='markup'>This is a <em>markup string</em>.</p>";
}
```
