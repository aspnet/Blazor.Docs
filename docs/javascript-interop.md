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

To call a JavaScript function, register the JavaScript function and call it in the .NET method:

1. Register a JavaScript function by calling `Blazor.registerFunction('functionName', functionImplementation)` in JavaScript. The following example registers two functions that echo a message. The `echo` function is synchronous, while the `echoAsync` version is asynchronous:

    ```javascript
    Blazor.registerFunction(
      'echo',
      function(message){
        return message;
      });

    Blazor.registerFunction(
      'echoAsync',
      function(message){
        return Promise.Resolve(message);
      });
    ```

1. To invoke a JavaScript function from C#, use the `T Invoke<T>(functionName, args)` (synchronous) or `Task<T> InvokeAsync<T>(functionName, args)` (asynchronous) method from the [RegisteredFunction](/api/Microsoft.AspNetCore.Blazor.Browser.Interop.RegisteredFunction.html) class. The following example calls the two preceding JavaScript functions from C# code by indicating their function names and passing `message` arguments:

    ```csharp
    var helloWorld = RegisteredFunction.Invoke<string>("echo", "Hello world!");
    var helloWorldAsync = 
        await RegisteredFunction
            .InvokeAsync<string>("echoAsync", "Hello world async!");
    ```

   Often, the JavaScript interop call is composed as a static .NET method so that it can be easily called throughout the Blazor app. The [Blazor class library template](#share-interop-code-in-a-blazor-class-library), described later in this topic, follows this pattern.

## Invoke .NET methods from JavaScript functions

JavaScript code in the browser might be required to call .NET methods. For example, a .NET method call can be made when a callback in JavaScript is triggered.

A .NET method call can be achieved by using the JavaScript functions `Blazor.invokeDotNetMethod` and `Blazor.invokeDotNetMethodAsync`. These two functions allow JavaScript to call synchronous and asynchronous .NET methods, respectively. The .NET method must have the following characteristics:

* Static
* Non-generic
* Not overloaded
* Concrete type method parameters
* Parameters deserializable using JSON

To call a .NET method, create the method in C# and invoke the method from JavaScript:

1. Create the .NET method in C#. The following example creates a `TimeoutCallback` method that writes a message to the console when the method is invoked:

    ```csharp
    namespace Alerts
    {
        public class Timeout
        {
            public static void TimeoutCallback()
            {
                Console.WriteLine('Timeout triggered!');
            }
        }
    }
    ```

1. Invoke the .NET method from JavaScript with a call to `Blazor.invokeDotNetMethod` (or `Blazor.invokeDotNetMethodAsync`). Specify the .NET assembly, namespace with class, and method name. The following example calls the `TimeoutCallback` .NET method in the preceding step:

    ```javascript
    Blazor.invokeDotNetMethod({
      type: {
        assembly: 'MyTimeoutAssembly',
        name: 'Alerts.Timeout'
      },
      method: {
        name: 'TimeoutCallback'
      }
    });
    ```

   The schema for the object parameter is:

   ```json
   {
     "type": {
       "assembly": "AssemblyName",
       "name": "NamespaceWithClass"
     },
     "method": {
       "name": "MethodName"
     }
   }
   ```

## Share interop code in a Blazor class library

JavaScript interop code can be included in a Blazor class library (`dotnet new blazorlib`), which allows you to share the code in a NuGet package.

The Blazor class library handles embedding JavaScript resources in the built assembly. The JavaScript files are placed in the *wwwroot* folder, and the tooling takes care of embedding the resources when the library is built.

The built NuGet package is referenced in the project file of a Blazor app just as any normal NuGet package is referenced. After the app has been restored, app code can call into JavaScript as if it were C#.
