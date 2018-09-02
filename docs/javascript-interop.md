---
title: Blazor JavaScript interop
author: danroth27
description: Learn how to invoke JavaScript functions from .NET and .NET methods from JavaScript.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 08/03/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/javascript-interop
---
# Blazor JavaScript interop

By [Javier Calvarro Nelson](https://github.com/javiercn), [Daniel Roth](https://github.com/danroth27), and [Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

A Blazor app can invoke JavaScript functions from .NET and .NET methods from JavaScript code.

## Invoke JavaScript functions from .NET methods

There are times when Blazor .NET code is required to call a JavaScript function. For example, a JavaScript call can expose browser capabilities or functionality from a JavaScript library to the Blazor app.

To call into JavaScript from .NET, use the `IJSRuntime` abstraction, which is accessible from `JSRuntime.Current`. The `InvokeAsync<T>` method on `IJSRuntime` takes an identifier for the JavaScript function you wish to invoke along with any number of JSON serializable arguments. The function identifier is relative to the global scope (`window`). For example if you wish to call `window.someScope.someFunction`, the identifier is `someScope.someFunction`. There's no need to register the function before it's called. The return type `T` must also be JSON serializable.

In the sample app, two JavaScript functions are available to the client-side app that interact with the DOM to receive user input and display a welcome message:

* `showPrompt` &ndash; Produces a prompt to accept user input (the user's name) and returns the name to the caller.
* `displayWelcome` &ndash; Assigns a welcome message from the caller to a DOM object with an `id` of `welcome`.

*wwwroot/exampleJsInterop.js*:

[!code-javascript[](./common/samples/2.x/BlazorSample/wwwroot/exampleJsInterop.js?highlight=2-7)]

Place the `<script>` tag that references the JavaScript file in the *wwwroot/index.html* file:

[!code-html[](./common/samples/2.x/BlazorSample/wwwroot/index.html?highlight=16)]

Don't place a script tag in a component file because the script tag can't be updated dynamically. For more information, see [Add compiler error if there's a &lt;script&gt; element inside a component (aspnet/Blazor &num;552)](https://go.microsoft.com/fwlink/?linkid=872131).

.NET methods interop with the JavaScript functions by calling `InvokeAsync<T>` method on `IJSRuntime`.

The sample app uses a pair of C# methods, `Prompt` and `Display`, to invoke the `showPrompt` and `displayWelcome` JavaScript functions:

*JsInteropClasses/ExampleJsInterop.cs*:

[!code-csharp[](./common/samples/2.x/BlazorSample/JsInteropClasses/ExampleJsInterop.cs?name=snippet1&highlight=6-8,14-16)]

The `IJSRuntime` abstraction is asynchronous to allow for out-of-process scenarios. If the app runs in-process and you want to invoke a JavaScript function synchronously, downcast to `IJSInProcessRuntime` and call `Invoke<T>` instead. We recommend that most JavaScript interop libraries use the async APIs to ensure the libraries are available in all Blazor scenarios, client-side or server-side.

The sample app includes a component to demonstrate JS interop. The component:

* Receives user input via a JS prompt.
* Returns the text to the component for processing.
* Calls a second JS function that interacts with the DOM to display a welcome message.

*Pages/JSInterop.cshtml*:

[!code-cshtml[](./common/samples/2.x/BlazorSample/Pages/JsInterop.cshtml?start=1&end=21&highlight=2-3,9-11,13,16-20)]

1. When `TriggerJsPrompt` is executed by selecting the component's **Trigger JavaScript Prompt** button, the `ExampleJsInterop.Prompt` method in C# code is called.
1. The `Prompt` method executes the JavaScript `showPrompt` function provided in the *wwwroot/exampleJsInterop.js* file.
1. The `showPrompt` function accepts user input (the user's name), which is HTML-encoded and returned to the `Prompt` method and ultimately back to the component. The component stores the user's name in a local variable, `name`.
1. The string stored in `name` is incorporated into a welcome message, which is passed to a second C# method, `ExampleJsInterop.Display`.
1. `Display` calls a JavaScript function, `displayWelcome`, which renders the welcome message into a heading tag.

## Capture references to elements

Some [JavaScript interop](xref:client-side/blazor/javascript-interop) scenarios require references to HTML elements. For example, a UI library may require an element reference for initialization, or you might need to call command-like APIs on an element, such as `focus` or `play`.

You can capture references to HTML elements in a component by adding a `ref` attribute to the HTML element and then defining a field of type `ElementRef` whose name matches the value of the `ref` attribute.

The following example shows capturing a reference to the username input element:

```csharp
<input ref="username" ... />

@functions {
    ElementRef username;
}
```

> [!NOTE]
> Do **not** use captured element references as a way of populating the DOM. Doing so may interfere with Blazor's declarative rendering model.

As far as .NET code is concerned, an `ElementRef` is an opaque handle. The *only* thing you can do with it is pass it through to JavaScript code via JavaScript interop. When you do so, the JavaScript-side code receives an `HTMLElement` instance, which it can use with normal DOM APIs.

For example, the following code defines a .NET extension method that enables setting the focus on an element:

*mylib.js*:

```javascript
window.myLib = {
  focusElement : function (element) {
    element.focus();
  }
}
```

*ElementRefExtensions.cs*:

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

```cshtml
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

> [!IMPORTANT]
> The `username` variable is only populated after the component renders and its output includes the `<input>` element. If you try to pass an unpopulated `ElementRef` to JavaScript code, the JavaScript code receives `null`. To manipulate element references after the component has finished rendering (to set the initial focus on an element) use the `OnAfterRenderAsync` or `OnAfterRender` [component lifecycle methods](xref:client-side/blazor/components/index#lifecycle-methods).

## Invoke .NET methods from JavaScript functions

### Static .NET method call

To invoke a static .NET method from JavaScript, use the `DotNet.invokeMethod` or `DotNet.invokeMethodAsync` functions. Pass in the identifier of the static method you wish to call, the name of the assembly containing the function, and any arguments. Again, the async version is preferred to support out-of-process scenarios. To be invokable from JavaScript, the .NET method must be public, static, and decorated with `[JSInvokable]`. By default, the method identifier is the method name, but you can specify a different identifier using the `JSInvokableAttribute` constructor. Calling open generic methods isn't currently supported.

The sample app includes a C# method to return an array of `int`s. The method is decorated with the `JSInvokable` attribute. 

*Pages/JsInterop.cshtml*:

[!code-cshtml[](./common/samples/2.x/BlazorSample/Pages/JsInterop.cshtml?start=39&end=52&highlight=3-6,9-13)]

JavaScript served to the client invokes the C# .NET method.

*wwwroot/exampleJsInterop.js*:

[!code-javascript[](./common/samples/2.x/BlazorSample/wwwroot/exampleJsInterop.js?highlight=8-12)]

When the **Trigger .NET static method ReturnArrayAsync** button is selected, examine the console output in the browser's web developer tools:

```console
Array(4) [ 1, 2, 3, 4 ]
```

The fourth array value is pushed to the array (`data.push(4);`) returned by `ReturnArrayAsync`.

### Instance method call

In Blazor 0.5.0 or later, you can also call .NET instance methods from JavaScript. To invoke a .NET instance method from JavaScript, first pass the .NET instance to JavaScript by wrapping it in a `DotNetObjectRef` instance. The .NET instance is passed by reference to JavaScript, and you can invoke .NET instance methods on the instance using the `invokeMethod` or `invokeMethodAsync` functions. The .NET instance can also be passed as an argument when invoking other .NET methods from JavaScript.

*ExampleJsInterop.cs*:

```csharp
public class ExampleJsInterop
{
    public static Task SayHello(string name)
    {
        return JSRuntime.Current.InvokeAsync<object>(
            "exampleJsFunctions.sayHello", 
            new DotNetObjectRef(new HelloHelper(name)));
    }
}
```

*exampleJsInterop.js*:

```javascript
window.exampleJsFunctions = {
  sayHello: function (dotnetHelper) {
    return dotnetHelper.invokeMethodAsync('SayHello')
      .then(r => console.log(r));
  }
};
```

*HelloHelper.cs*:

```csharp
public class HelloHelper
{
    public HelloHelper(string name)
    {
        Name = name;
    }

    public string Name { get; set; }

    [JSInvokable]
    public string SayHello() => $"Hello, {Name}!";
}
```

*Output*:

```
Hello, Blazor!
```

## Share interop code in a Blazor class library

JavaScript interop code can be included in a Blazor class library (`dotnet new blazorlib`), which allows you to share the code in a NuGet package.

The Blazor class library handles embedding JavaScript resources in the built assembly. The JavaScript files are placed in the *wwwroot* folder, and the tooling takes care of embedding the resources when the library is built.

The built NuGet package is referenced in the project file of a Blazor app just as any normal NuGet package is referenced. After the app has been restored, app code can call into JavaScript as if it were C#.
