---
title: Host and deploy Blazor
author: guardrex
description: Discover how to host and deploy Blazor apps using ASP.NET Core, Content Delivery Networks (CDN), file servers, and GitHub Pages.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 04/16/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/host-and-deploy/index
---
# Host and deploy Blazor

By [Luke Latham](https://github.com/guardrex), [Rainer Stropek](https://www.timecockpit.com), and [Daniel Roth](https://github.com/danroth27)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

## Publish the app

Blazor apps are published for deployment in Release configuration with the [dotnet publish](https://docs.microsoft.com/dotnet/core/tools/dotnet-publish) command. An IDE may handle executing the `dotnet publish` command automatically using its built-in publishing features, so it might not be necessary to manually execute the command from a command prompt depending on the development tools in use.

```console
dotnet publish -c Release
```

`dotnet publish` triggers a [restore](https://docs.microsoft.com/dotnet/core/tools/dotnet-restore) of the project's dependencies and [builds](https://docs.microsoft.com/dotnet/core/tools/dotnet-build) the project before creating the assets for deployment. As part of the build process, unused methods and assemblies are removed to reduce app download size and load times. The deployment is created in the */bin/Release/&lt;target-framework&gt;/publish* folder.

The assets in the *publish* folder are deployed to the web server. Deployment might be a manual or automated process depending on the development tools in use.

## Configure the Linker

Blazor performs Intermediate Language (IL) linking on each build to remove unnecessary IL from the output assemblies. You can control assembly linking on build. For more information, see <xref:client-side/blazor/host-and-deploy/configure-linker>.

## Rewrite URLs for correct routing

Routing requests for page components in a client-side app isn't as simple as routing requests to a server-side, hosted app. Consider a client-side app with two pages:

* **_Main.cshtml_** &ndash; Loads at the root of the app and contains a link to the About page (`href="About"`).
* **_About.cshtml_** &ndash; About page.

When the app's default document is requested using the browser's address bar (for example, `https://www.contoso.com/`):

1. The browser makes a request.
1. The default page is returned, which is usually *index.html*.
1. *index.html* bootstraps the app.
1. Blazor's router loads and the Razor Main page (*Main.cshtml*) is displayed.

On the Main page, selecting the link to the About page loads the About page. Selecting the link to the About page works on the client because the Blazor router stops the browser from making a request on the Internet to `www.contoso.com` for `About` and serves the About page itself. All of the requests for internal pages *within the client-side app* work the same way: Requests don't trigger browser-based requests to server-hosted resources on the Internet. The router handles the requests internally.

If a request is made using the browser's address bar for `www.contoso.com/About`, the request fails. No such resource exists on the app's Internet host, so a *404 Not found* response is returned.

Because browsers make requests to Internet-based hosts for client-side pages, web servers and hosting services must rewrite all requests for resources not physically on the server to the *index.html* page. When *index.html* is returned, the app's client-side router takes over and responds with the correct resource.

## App base path

The app base path is the virtual app root path on the server. For example, an app that resides on the Contoso server in a virtual folder at `/CoolBlazorApp/` is reached at `https://www.contoso.com/CoolBlazorApp` and has a virtual base path of `/CoolBlazorApp/`. By setting the app base path to `CoolBlazorApp/`, the app is made aware of where it virtually resides on the server. The app can use the app base path to construct URLs relative to the app root from a component that isn't in the root directory. This allows components that exist at different levels of the directory structure to build links to other resources at locations throughout the app. The app base path is also used to intercept hyperlink clicks where the `href` target of the link is within the app base path URI space&mdash;the Blazor router handles the internal navigation.

In many hosting scenarios, the server's virtual path to the app is the root of the app. In these cases, the app base path is a forward slash (`<base href="/" />`), which is the default configuration for a Blazor app. In other hosting scenarios, such as GitHub Pages and IIS virtual directories, the app base path must be set to the server's virtual path to the app. To set the Blazor app's base path, add or update the `<base>` tag in *index.html* found within the `<head>` tag elements. Set the `href` attribute value to `<virtual-path>/` (the trailing slash is required), where `<virtual-path>/` is the full virtual app root path on the server for the app. In the preceding example, the virutal path is set to `CoolBlazorApp/`: `<base href="CoolBlazorApp/" />`.

For an app with a non-root virtual path configured (for example, `<base href="CoolBlazorApp/" />`), the app fails to find its resources *when run locally*. To overcome this problem during local development and testing, you can supply a *path base* argument that matches the `href` value of the `<base>` tag at runtime.

To pass the path base argument with the root path (`/`) when running the app locally, execute the following command from the Blazor app's directory:

```console
dotnet run --pathbase=/CoolBlazorApp
```

The app responds locally at `http://localhost:port/CoolBlazorApp`.

For more information, see the [path base host configuration value](#path-base) section.

> [!IMPORTANT]
> If a Blazor app uses the [client-side (in-process) hosting model](xref:client-side/blazor/host-and-deploy/hosting-models#client-side-in-process-hosting-model) (based on the **Blazor** project template) and is hosted as an IIS sub-application in an ASP.NET Core app, it's important to disable the inherited ASP.NET Core Module handler. Remove the handler in the Blazor app's published *web.config* file by adding a `<handlers>` section to the file:
>
> ```xml
> <handlers>
>   <remove name="aspNetCore" />
> </handlers>
> ```
>
> Removing the handler is performed in addition to configuring the app's base path as described in this section. Set the app base path in the Blazor app's *index.html* file to the IIS alias used when configuring the sub-app in IIS.

## Host configuration values

Blazor apps that use the [server-side hosting model](xref:client-side/blazor/host-and-deploy/hosting-models#server-side-hosting-model) can accept [Web Host configuration values](https://docs.microsoft.com/aspnet/core/fundamentals/host/web-host#host-configuration-values).

Blazor apps that use the [client-side (in-process) hosting model](xref:client-side/blazor/host-and-deploy/hosting-models#client-side-in-process-hosting-model) can accept the following host configuration values as command-line arguments at runtime.

### Content Root

The `--contentroot` argument determines where the app begins searching for content files, such as components.

* Pass the argument when running the app locally at a command prompt. From the app's directory, execute:

  ```console
  dotnet run --contentroot=/<content-root>
  ```
* Add an entry to the app's *launchSettings.json* file in the **IIS Express** profile. This setting is picked up when running the app with the Visual Studio Debugger and when running the app from a command prompt with `dotnet run`.

  ```json
  "commandLineArgs": "--contentroot=/<content-root>"
  ```
* In Visual Studio, specify the argument in **Properties** > **Debug** > **Application arguments**. Setting the argument in the Visual Studio property page adds the argument to the *launchSettings.json* file.

  ```console
  --contentroot=/<content-root>
  ```

### Path base

The `--pathbase` argument sets the app base path for an app run locally with a non-root virtual path (the `<base>` tag `href` is set to a path other than `/` for staging and production). For more information, see the [App base path](#app-base-path) section.

> [!IMPORTANT]
> Unlike the path provided to `href` of the `<base>` tag, don't include a trailing slash (`/`) when passing the `--pathbase` argument value. If the app base path is provided in the `<base>` tag as `<base href="/CoolBlazorApp/" />` (includes a trailing slash), pass the command-line argument value as `--pathbase=/CoolBlazorApp` (no trailing slash).

* Pass the argument when running the app locally at a command prompt. From the app's directory, execute:

  ```console
  dotnet run --pathbase=/<virtual-path>
  ```
* Add an entry to the app's *launchSettings.json* file in the **IIS Express** profile. This setting is picked up when running the app with the Visual Studio Debugger and when running the app from a command prompt with `dotnet run`.

  ```json
  "commandLineArgs": "--pathbase=/<virtual-path>"
  ```
* In Visual Studio, specify the argument in **Properties** > **Debug** > **Application arguments**. Setting the argument in the Visual Studio property page adds the argument to the *launchSettings.json* file.

  ```console
  --pathbase=/<virtual-path>
  ```

### URLs

The `--urls` argument indicates the IP addresses or host addresses with ports and protocols to listen on for requests.

* Pass the argument when running the app locally at a command prompt. From the app's directory, execute:

  ```console
  dotnet run --urls=http://127.0.0.1:0
  ```
* Add an entry to the app's *launchSettings.json* file in the **IIS Express** profile. This setting is picked up when running the app with the Visual Studio Debugger and when running the app from a command prompt with `dotnet run`.

  ```json
  "commandLineArgs": "--urls=http://127.0.0.1:0"
  ```
* In Visual Studio, specify the argument in **Properties** > **Debug** > **Application arguments**. Setting the argument in the Visual Studio property page adds the argument to the *launchSettings.json* file.

  ```console
  --urls=http://127.0.0.1:0
  ```

## Deployment models

There are two deployment models for Blazor apps:

* [Hosted deployment with ASP.NET Core](#hosted-deployment-with-aspnet-core) &ndash; Hosted deployment uses an ASP.NET Core app on the server to host the Blazor app.
* [Standalone deployment](#standalone-deployment) &ndash; Standalone deployment places the Blazor app on a static hosting web server or service, where .NET isn't used to serve the Blazor app.

### Hosted deployment with ASP.NET Core

In a hosted deployment, an ASP.NET Core app handles single-page application routing and Blazor app hosting. The published ASP.NET Core app, along with one or more Blazor apps that it hosts, is deployed to the web server or hosting service.

To host a Blazor app, the ASP.NET Core app must:

* Reference the Blazor app project.
* Reference the [Microsoft.AspNetCore.Blazor.Server](https://www.nuget.org/packages/Microsoft.AspNetCore.Blazor.Server/) package in its project file.
* Configure Blazor app hosting with the `UseBlazor` extension method on [IApplicationBuilder](https://docs.microsoft.com/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder) in `Startup.Configure`.

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseBlazor<Client.Program>();
}
```

The `UseBlazor` extension method performs the following tasks:

* Configure [Static File Middleware](https://docs.microsoft.com/aspnet/core/fundamentals/static-files) to serve Blazor's static assets from the *dist* folder. In the Development environment, the files in *wwwroot* folder are served.
* Configure single-page application routing for resource requests that aren't for actual files that exist on disk. The app serves the default document (*wwwroot/index.html*) for any request that hasn't been served by a prior Static File Middleware instance. For example, a request to receive a page from the app that should be handled by the Blazor router on the client is rewritten into a request for the *wwwroot/index.html* page.

When the ASP.NET Core app is published, the Blazor app is included in the published output so that the ASP.NET Core app and the Blazor app can be deployed together. For more information on ASP.NET Core app hosting and deployment, see [Host and deploy ASP.NET Core](https://docs.microsoft.com/aspnet/core/host-and-deploy).

For information on deploying to Azure App Service, see the following topics:

[Publish to Azure with Visual Studio](https://docs.microsoft.com/aspnet/core/tutorials/publish-to-azure-webapp-using-vs)  
Learn how to publish an ASP.NET Core-hosted Blazor app to Azure App Service using Visual Studio.

[Publish to Azure with CLI tools](https://docs.microsoft.com/aspnet/core/tutorials/publish-to-azure-webapp-using-cli)  
Learn how to publish an ASP.NET Core app to Azure App Service using the Git command-line client.

### Standalone deployment

In a standalone deployment, only the Blazor client-side app is deployed to the server or hosting service. An ASP.NET Core server-side app isn't used to host the Blazor app. The Blazor app's static files are requested by the browser directly from the static file web server or service.

When deploying a standalone Blazor app from the published *dist* folder, any web server or hosting service that serves static files can host a Blazor app.

#### IIS

IIS is a capable static file server for Blazor apps. To configure IIS to host Blazor, see [Build a Static Website on IIS](https://docs.microsoft.com/iis/manage/creating-websites/scenario-build-a-static-website-on-iis).

Published assets are created in the *\\bin\\Release\\&lt;target-framework&gt;\\publish* folder. Host the contents of the *publish* folder on the web server or hosting service.

**web.config**

When a Blazor project is published, a *web.config* file is created with the following IIS configuration:

* MIME types are set for the following file extensions:
  - *\*.dll*: `application/octet-stream`
  - *\*.json*: `application/json`
  - *\*.wasm*: `application/wasm`
  - *\*.woff*: `application/font-woff`
  - *\*.woff2*: `application/font-woff`
* HTTP compression is enabled for the following MIME types:
  - `application/octet-stream`
  - `application/wasm`
* URL Rewrite Module rules are established:
  - Serve the sub-directory where the app's static assets reside (*&lt;assembly_name&gt;\\dist\\&lt;path_requested&gt;*).
  - Create SPA fallback routing so that requests for non-file assets are redirected to the app's default document in its static assets folder (*&lt;assembly_name&gt;\\dist\\index.html*).

**Install the URL Rewrite Module**

The [URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite) is required to rewrite URLs. The module isn't installed by default, and it isn't available for install as an Web Server (IIS) role service feature. The module must be downloaded from the IIS website. Use the Web Platform Installer to install the module:

1. Locally, navigate to the [URL Rewrite Module downloads page](https://www.iis.net/downloads/microsoft/url-rewrite#additionalDownloads). For the English version, select **WebPI** to download the WebPI installer. For other languages, select the appropriate architecture for the server (x86/x64) to download the installer.
1. Copy the installer to the server. Run the installer. Select the **Install** button and accept the license terms. A server restart isn't required after the install completes.

**Configure the website**

Set the website's **Physical path** to the Blazor app's folder. The folder contains:

* The *web.config* file that IIS uses to configure the website, including the required redirect rules and file content types.
* The app's static asset folder.

**Troubleshooting**

If a *500 Internal Server Error* is received and IIS Manager throws errors when attempting to access the website's configuration, confirm that the URL Rewrite Module is installed. When the module isn't installed, the *web.config* file can't be parsed by IIS. This prevents the IIS Manager from loading the website's configuration and the website from serving Blazor's static files.

For more information on troubleshooting deployments to IIS, see [Troubleshoot ASP.NET Core on IIS](https://docs.microsoft.com/aspnet/core/host-and-deploy/iis/troubleshoot).

#### Nginx

The following *nginx.conf* file is simplified to show how to configure Nginx to send the *Index.html* file whenever it can't find a corresponding file on disk.

```
events { }
http {
    server {
        listen 80;

        location / {
            root /usr/share/nginx/html;
            try_files $uri $uri/ /Index.html =404;
        }
    }
}
```

For more information on production Nginx web server configuration, see [Creating NGINX Plus and NGINX Configuration Files](https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/).

#### Nginx in Docker

To host Blazor in Docker using Nginx, setup the Dockerfile to use the Alpine-based Nginx image. Update the Dockerfile to copy the *nginx.config* file into the container.

Add one line to the Dockerfile, as shown in the following example:

```
FROM nginx:alpine
COPY ./bin/Release/netstandard2.0/publish /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/nginx.conf
```

#### GitHub Pages

To handle URL rewrites, add a *404.html* file with a script that handles redirecting the request to the *index.html* page. For an example implementation provided by the community, see [Single Page Apps for GitHub Pages](http://spa-github-pages.rafrex.com/) ([rafrex/spa-github-pages on GitHub](https://github.com/rafrex/spa-github-pages#readme)). An example using the community approach can be seen at [blazor-demo/blazor-demo.github.io on GitHub](https://github.com/blazor-demo/blazor-demo.github.io) ([live site](https://blazor-demo.github.io/)).

When using a project site instead of an organization site, add or update the `<base>` tag in *index.html*. Set the `href` attribute value to `<repository-name>/`, where `<repository-name>/` is the GitHub repository name.
