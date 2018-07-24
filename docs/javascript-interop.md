---
title: Blazor JavaScript interop
author: danroth27
description: Learn how to invoke JavaScript functions from .NET and .NET methods from JavaScript.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 06/10/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/javascript-interop
---
# Blazor JavaScript interop

By [Javier Calvarro Nelson](https://github.com/javiercn) and [Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

A Blazor app can invoke JavaScript functions from .NET and .NET methods from JavaScript code.

## Invoke JavaScript functions from .NET methods

There are times when Blazor .NET code is required to call a JavaScript function. For example, a JavaScript call can expose browser capabilities or functionality from a JavaScript library to the Blazor app.

To call into JavaScript from .NET use the new `IJSRuntime` abstraction, which is accessible from `JSRuntime.Current`. The `InvokeAsync<T>` method on `IJSRuntime` takes an identifier for the JavaScript function you wish to invoke along with any number of JSON serializable arguments. The function identifier is relative to the global scope (`window`). For example, if you wish to call `window.someScope.someFunction` then the identifier would be `someScope.someFunction`. There is no longer any need to register the function before it can be called. The return type `T` must also be JSON serializable.

*exampleJsInterop.js*
```js
window.exampleJsFunctions = {
  showPrompt: function (message) {
    return prompt(message, 'Type anything here');
  }
};
```

*ExampleJsInterop.cs*
```csharp
public class ExampleJsInterop
{
    public static Task<string> Prompt(string message)
    {
        // Implemented in exampleJsInterop.js
        return JSRuntime.Current.InvokeAsync<string>(
            "exampleJsFunctions.showPrompt",
            message);
    }
}
```
The `IJSRuntime` abstraction is async to allow for out-of-process scenarios. However, if you are running in-process and  want to invoke a JavaScript function synchronously you can downcast to `IJSInProcessRuntime` and call `Invoke<T>` instead. We recommend that most JavaScript interop libraries should use the async APIs to ensure the libraries can be used in all Blazor scenarios, client-side or server-side.

## Invoke .NET methods from JavaScript functions

To invoke a static .NET method from JavaScript use the `DotNet.invokeMethod` or `DotNet.invokeMethodAsync` functions passing in the identifier of the static method you wish to call, the name of the assembly containing the function, and any arguments. Again, the async version is preferred to support out-of-process scenarios. To be invokable from JavaScript the .NET method must be public, static, and attributed with `[JSInvokable]`. By default, the method identifier is the method name, but you can specify a different identifier using the `JSInvokableAttribute` constructor. Calling open generic methods is not currently supported.

*JavaScriptInteroperable.cs*
```csharp
public class JavaScriptInvokable
{
    [JSInvokable]
    public static Task<int[]> ReturnArrayAsync()
    {
        return Task.FromResult(new int[] { 1, 2, 3 });
    }
}
```

*dotnetInterop.js*
```js
DotNet.invokeMethodAsync(assemblyName, 'ReturnArrayAsync').then(data => ...)
```

New in Blazor 0.5.0, you can also call .NET instance methods from JavaScript. To invoke a .NET instance method from JavaScript you first pass the .NET instance to JavaScript by wrapping it in a `DotNetObjectRef` instance. The .NET instance will then be passed by reference to JavaScript and you can invoke .NET instance methods on the instance using the `invokeMethod` or `invokeMethodAsync` functions. The .NET instance can also be passed as an argument when invoking other .NET methods from JavaScript.

*ExampleJsInterop.cs*
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

*exampleJsInterop.js*
```js
window.exampleJsFunctions = {
  sayHello: function (dotnetHelper) {
    return dotnetHelper.invokeMethodAsync('SayHello')
      .then(r => console.log(r));
  }
};
```

*HelloHelper.cs*
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

*Output*
```
Hello, Blazor!
```

## Share interop code in a Blazor class library

JavaScript interop code can be included in a Blazor class library (`dotnet new blazorlib`), which allows you to share the code in a NuGet package.

The Blazor class library handles embedding JavaScript resources in the built assembly. The JavaScript files are placed in the *wwwroot* folder, and the tooling takes care of embedding the resources when the library is built.

The built NuGet package is referenced in the project file of a Blazor app just as any normal NuGet package is referenced. After the app has been restored, app code can call into JavaScript as if it were C#.
