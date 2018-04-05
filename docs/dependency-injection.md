---
title: Dependency injection in Blazor
author: rstropek
description: Learn how Blazor apps can use built-in services by having them injected into components.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 04/09/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/dependency-injection
---
# Dependency injection in Blazor

By [Rainer Stropek](https://www.timecockpit.com)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

Blazor has [dependency injection (DI)](https://docs.microsoft.com/aspnet/core/fundamentals/dependency-injection) built-in. Apps can use built-in services by having them injected into components. Apps can also define custom services and make them available via DI.

## What is dependency injection?

DI is a technique for accessing services configured in a central location. This can be useful to:

* Share a single instance of a service class across many components (known as a *singleton* service)
* Decouple components from particular concrete service classes and only reference abstractions. For example, there might be an interface `IDataAccess` implemented by a concrete class `DataAccess`. If a component uses DI to receive an `IDataAccess` implementation, it isn't coupled to any concrete type. That means the implementation could easily be swapped, perhaps to a mock implementation in unit tests.

Blazor's DI system is responsible for supplying instances of services to components. It also resolves dependencies recursively, so that services themselves can depend on further services, and so on. DI is configured during startup of the app. An example is shown later in this topic.

## Use of existing .NET mechanisms

DI in Blazor is based on .NET's [System.IServiceProvider](https://docs.microsoft.com/dotnet/api/system.iserviceprovider) interface. It defines a generic mechanism for retrieving a service object in .NET apps.

Blazor's implementation of `System.IServiceProvider` obtains its services from an underlying [Microsoft.Extensions.DependencyInjection.IServiceCollection](https://docs.microsoft.com/dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection).

## Add services to DI

After creating a new app, examine the `Main` method in *Program.cs*:

```csharp
static void Main(string[] args)
{
    var serviceProvider = new BrowserServiceProvider(configure =>
    {
        // Add any custom services here
    });

    new BrowserRenderer(serviceProvider).AddComponent<App>("app");
}
```

`BrowserServiceProvider` receives an action with which you can add your app services to DI. `configure` references the underlying `IServiceCollection`, which is a list of service descriptor objects ([Microsoft.Extensions.DependencyInjection.ServiceDescriptor](https://docs.microsoft.com/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor)). Services are added by providing service descriptors to the service collection. The following is a code sample demonstrating the concept:

```csharp
static void Main(string[] args)
{
    var serviceProvider = new BrowserServiceProvider(configure =>
    {
        configure.Add(ServiceDescriptor.Singleton<IDataAccess, DataAccess>());
    });

    new BrowserRenderer(serviceProvider).AddComponent<App>("app");
}
```

`ServiceDescriptor` offers various overloads of three different functions that can be used to add services to Blazor's DI:

| Method      | Description |
| ----------- | ----------- |
| `Singleton` | DI creates a *single instance* of your service. All components requiring this service receive a reference to this instance. |
| `Transient` | Whenever a component requires this service, it receives a *new instance* of the service. |
| `Scoped`    | Blazor doesn't currently have the concept of DI scopes. `Scoped` behaves like `Singleton`. Therefore, prefer `Singleton` and avoid `Scoped`. |

## Default services

Blazor provides default services that are automatically added to the service collection of an app. The following table shows a list of the default services currently provided by Blazor's `BrowserServiceProvider`.

| Method       | Description |
| ------------ | ----------- |
| `IUriHelper` | Helpers for working with URIs and navigation state (singleton). |
| `HttpClient` | Provides methods for sending HTTP requests and receiving HTTP responses from a resource identified by a URI (singleton). Note that this instance of [System.Net.Http.HttpClient](https://docs.microsoft.com/dotnet/api/system.net.http.httpclient) uses the browser for handling the HTTP traffic in the background. Its [BaseAddress](https://docs.microsoft.com/dotnet/api/system.net.http.httpclient.baseaddress) is automatically set to the base URI prefix of the app. |

Note that it is possible to use a custom services provider instead of the default `BrowserServiceProvider` which is added by the default template. A custom service provider would not automatically provide the default services mentioned above. They would have to be added to the new service provider explicitly.

## Request a service in a component

Once services are added to the service collection, they can be injected into the components' Razor templates using the `@inject` Razor keyword. `@inject` has two parameters:

* Type name: The type of the service to inject.
* Property name: The name of the property receiving the injected app service. Note that the property doesn't require manual creation. The compiler creates the property.

Multiple `@inject` statements can be used to inject different services.

The following example shows how to use `@inject`. The service implementing `Services.IDataAccess` is injected into the component's property `DataRepository`. Note how the code is only using the `IDataAccess` abstraction:

```csharp
@page "/customer-list"
@using Services
@inject IDataAccess DataRepository

<ul>
    @if (Customers != null)
    {
        @foreach (var customer in Customers)
        {
            <li>@customer.FirstName @customer.LastName</li>
        }
    }
</ul>

@functions {
    private IReadOnlyList<Customer> Customers;

    protected override async Task OnInitAsync()
    {
        // The property DataRepository has received an implemenation
        // of IDataAccess through dependency injection. We can use
        // it to get data from our server.
        Customers = await DataRepository.GetAllCustomersAsync();
    }
}
```

Internally, the generated property (`DataRepository`) is decorated with the `Microsoft.AspNetCore.Blazor.Components.InjectAttribute` attribute. Typically, this attribute isn't used directly. If a base class is required for components and injected properties are also required for the base class, `InjectAttribute` can be manually added:

```csharp
public class ComponentBase : BlazorComponent
{
    // Note that Blazor's dependency injection works even if using the
    // InjectAttribute in a component's base class.
    [Inject]
    protected IDataAccess DataRepository { get; set; }
    ...
}
```

In components derived from the base class, the `@inject` directive isn't required. The `InjectAttribute` of the base class is satisfactory:

```csharp
@page "/demo"
@inherits ComponentBases

<h1>...</h1>
...
```

## Dependency injection in services

Complex services might require additional services. In the prior example, `DataAccess` might require Blazor's default service `HttpClient`. `@inject` or the `InjectAttribute` can't be used in services. *Constructor injection* must be used instead. Required services are added by adding parameters to the service's constructor. When dependency injection creates the service, it recognizes the services it requires in the constructor and provides them accordingly.

The following code sample demonstrates the concept:

```csharp
public class DataAccess : IDataAccess
{
    // Note that the constructor receives an HttpClient via dependency
    // injection. HttpClient is a default service offered by Blazor.
    public Repository(HttpClient client)
    {
        ...
    }
    ...
}
```

Note the following prerequisites for constructor injection:

* There must be one constructor whose arguments can all be fulfilled by dependency injection. Note that additional parameters not covered by DI are allowed if default values are specified for them.
* The applicable constructor must be *public*.
* There must only be one applicable constructor. In case of an ambiguity, DI throws an exception.

## Service lifetime

Note that Blazor doesn't automatically dispose injected services that implement `IDisposable`. Components can implement `IDisposable`. Services are disposed when the user navigates away from the component.

The following code sample demonstrates how to implement `IDisposable` in a component:

```csharp
...
@using System;
@implements IDisposable
...

@functions {
    ...
    public void Dispose()
    {
        // Add code for disposing here
        ...
    }
}
```
