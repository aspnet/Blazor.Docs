---
title: Dependency injection in Blazor
author: rstropek
description: Learn how Blazor apps can use built-in services by having them injected into components.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 04/16/2018
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

* Share a single instance of a service class across many components (known as a *singleton* service).
* Decouple components from particular concrete service classes and only reference abstractions. For example, an interface `IDataAccess` is implemented by a concrete class `DataAccess`. When a component uses DI to receive an `IDataAccess` implementation, the component isn't coupled to the concrete type. The implementation can be swapped, perhaps to a mock implementation in unit tests.

Blazor's DI system is responsible for supplying instances of services to components. DI also resolves dependencies recursively so that services themselves can depend on further services. DI is configured during startup of the app. An example is shown later in this topic.

## Add services to DI

After creating a new app, examine the `Main` method in *Program.cs*:

```csharp
static void Main(string[] args)
{
    var serviceProvider = new BrowserServiceProvider(services =>
    {
        // Add custom services here
    });

    new BrowserRenderer(serviceProvider).AddComponent<App>("app");
}
```

`BrowserServiceProvider` receives an action where app services are added to DI. `services` references the underlying [IServiceCollection](https://docs.microsoft.com/dotnet/api/microsoft.extensions.dependencyinjection.iservicecollection), which is a list of service descriptor objects ([ServiceDescriptor](https://docs.microsoft.com/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor)). Services are added by providing service descriptors to the service collection. The following code sample demonstrates the concept:

```csharp
@using Microsoft.Extensions.DependencyInjection

static void Main(string[] args)
{
    var serviceProvider = new BrowserServiceProvider(services =>
    {
        services.AddSingleton<IDataAccess, DataAccess>();
    });

    new BrowserRenderer(serviceProvider).AddComponent<App>("app");
}
```

Services can be configured with the following lifetimes:

| Method      | Description |
| ----------- | ----------- |
| [Singleton](https://docs.microsoft.com/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor.singleton#Microsoft_Extensions_DependencyInjection_ServiceDescriptor_Singleton__1_System_Func_System_IServiceProvider___0__) | DI creates a *single instance* of the service. All components requiring this service receive a reference to this instance. |
| [Transient](https://docs.microsoft.com/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor.transient) | Whenever a component requires this service, it receives a *new instance* of the service. |
| [Scoped](https://docs.microsoft.com/dotnet/api/microsoft.extensions.dependencyinjection.servicedescriptor.scoped) | Blazor doesn't currently have the concept of DI scopes. `Scoped` behaves like `Singleton`. Therefore, prefer `Singleton` and avoid `Scoped`. |

## Default services

Blazor provides default services that are automatically added to the service collection of an app. The following table shows a list of the default services currently provided by Blazor's [BrowserServiceProvider](/api/Microsoft.AspNetCore.Blazor.Browser.Services.BrowserServiceProvider.html).

| Method       | Description |
| ------------ | ----------- |
| [IUriHelper](/api/Microsoft.AspNetCore.Blazor.Services.IUriHelper.html) | Helpers for working with URIs and navigation state (singleton). |
| [HttpClient](https://docs.microsoft.com/dotnet/api/system.net.http.httpclient) | Provides methods for sending HTTP requests and receiving HTTP responses from a resource identified by a URI (singleton). Note that this instance of `HttpClient` uses the browser for handling the HTTP traffic in the background. [HttpClient.BaseAddress](https://docs.microsoft.com/dotnet/api/system.net.http.httpclient.baseaddress) is automatically set to the base URI prefix of the app. |

Note that it is possible to use a custom services provider instead of the default `BrowserServiceProvider` that's added by the default template. A custom service provider doesn't automatically provide the default services listed in the table. Those services must be added to the new service provider explicitly.

## Request a service in a component

Once services are added to the service collection, they can be injected into the components' Razor templates using the `@inject` Razor directive. `@inject` has two parameters:

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
        // The property DataRepository received an implemenation
        // of IDataAccess through dependency injection. Use 
        // DataRepository to obtain data from the server.
        Customers = await DataRepository.GetAllCustomersAsync();
    }
}
```

Internally, the generated property (`DataRepository`) is decorated with the [InjectAttribute](/api/Microsoft.AspNetCore.Blazor.Components.InjectAttribute.html) attribute. Typically, this attribute isn't used directly. If a base class is required for components and injected properties are also required for the base class, `InjectAttribute` can be manually added:

```csharp
public class ComponentBase : BlazorComponent
{
    // Blazor's dependency injection works even if using the
    // InjectAttribute in a component's base class.
    [Inject]
    protected IDataAccess DataRepository { get; set; }
    ...
}
```

In components derived from the base class, the `@inject` directive isn't required. The `InjectAttribute` of the base class is sufficient:

```csharp
@page "/demo"
@inherits ComponentBase

<h1>...</h1>
...
```

## Dependency injection in services

Complex services might require additional services. In the prior example, `DataAccess` might require Blazor's default service `HttpClient`. `@inject` or the `InjectAttribute` can't be used in services. *Constructor injection* must be used instead. Required services are added by adding parameters to the service's constructor. When dependency injection creates the service, it recognizes the services it requires in the constructor and provides them accordingly.

The following code sample demonstrates the concept:

```csharp
public class DataAccess : IDataAccess
{
    // The constructor receives an HttpClient via dependency
    // injection. HttpClient is a default service offered by Blazor.
    public DataAccess(HttpClient client)
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

## Additional resources

* [Dependency injection in ASP.NET Core](https://docs.microsoft.com/aspnet/core/fundamentals/dependency-injection)
