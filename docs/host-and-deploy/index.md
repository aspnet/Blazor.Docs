---
title: Host and deploy Blazor
author: guardrex
description: Discover how to host and deploy Blazor apps using ASP.NET Core, Content Delivery Networks (CDN), file servers, and GitHub Pages.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 04/09/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/host-and-deploy/index
---
# Host and deploy Blazor

By [Luke Latham](https://github.com/guardrex) and [Rainer Stropek](https://www.timecockpit.com)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

## Publishing

Blazor apps are published for deployment in Release configuration with the [dotnet publish](https://docs.microsoft.com/dotnet/core/tools/dotnet-publish) command:

```console
dotnet publish -c Release
```

The .NET linker strips unused [Intermediate Language (IL)](https://docs.microsoft.com/dotnet/standard/managed-code) when the app is built in Release configuration.

The deployment is created in the */bin/Release/netstandard2.0/publish* folder. The assets in the *publish* folder are moved to the web server.

## Rewrite URLs for correct routing

Routing requests for page components in a client-side app isn't as simple as routing requests to a server-side, hosted app. Consider a client-side app with two pages:

* **_Index.cshtml_**: Loads at the root of the app and contains a link to the About page (`href="/About"`).
* **_About.cshtml_**: About page.

When the app's default document is requested using the browser's address bar (for example, `https://www.contoso.com/`):

1. The browser makes a request.
1. The default page is returned, which is usually *index.html*.
1. *index.html* bootstraps the app.
1. Blazor's router loads and the Razor Index page (*Index.cshtml*) is displayed.

On the Index page, selecting the link to the About page loads the About page. Selecting the link to the About page works on the client because the Blazor router stops the browser from making a request on the Internet to `www.contoso.com` for `/About` and serves the About page itself. All of the requests for internal pages *within the client-side app* work the same way: Requests don't trigger browser-based requests to server-hosted resources on the Internet. The router handles the requests internally.

If a request is made using the browser's address bar for `www.contoso.com/About`, the request fails. No such resource exists on the app's Internet host, so a *404 Not found* response is returned.

Because browsers make requests to Internet-based hosts for client-side pages, hosts of hosted Blazor apps must rewrite all requests to the *index.html* page. When *index.html* is returned, the app's router takes over and responds with the correct resource.

Not all hosting scenarios permit the enforcement of rewrite rules. Additional information is provided in the hosting configuration examples that follow.

## Host with ASP.NET Core

ASP.NET Core apps that host Blazor apps use Blazor Middleware by calling the `UseBlazor` extension method on [IApplicationBuilder](https://docs.microsoft.com/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder) in `Startup.Configure`:

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

The Blazor middleware performs the following tasks:

* Reads the Blazor configuration file (*.blazor.config*) from the `ClientAssemblyPath`, which is either an option provided to the `UseBlazor` extension method or defaulted to the app's assembly file location.
* Configures [Static File Middleware](https://docs.microsoft.com/aspnet/core/fundamentals/static-files) to serve Blazor's static assets from the *dist* folder. In the Development environment, the files in *wwwroot* are served.
* Configure live reloading when in the Development environment and Debug configuration.
* Configure single-page application routing for files outside of the *_framework* folder, which serves the default page (*index.html*) for any request that hasn't been served by a prior Static File Middleware instance.

For general information on ASP.NET Core app hosting and deployment, see:

* [Host and deploy ASP.NET Core](https://docs.microsoft.com/aspnet/core/host-and-deploy)
* [.NET Core application deployment](https://docs.microsoft.com/dotnet/core/deploying/)
* [Deploying .NET Core apps with command-line interface (CLI) tools](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-cli)
* [Deploying .NET Core apps with Visual Studio](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs)

## Docker/Linux hosting

The first line of the Blazor config file (*.blazor.config*) configures the source MSBuild path to `.`. A consequence of this setting is that the webroot path is set to `.\wwwroot`, which results in published files being split between *wwwroot* and *dist* folders. For an unsupported workaround, see [On linux UseBlazor extension method fails with "The path must be absolute"](https://github.com/aspnet/Blazor/issues/376).

## Static deployment strategies

The following static deployment strategies make the Blazor deployment's static assets available directly for download. Conceptually, other static hosting system configurations are similar.

### IIS

IIS is a capable static file server for Blazor apps. To configure IIS to host Blazor, follow these steps provided.

In the following configuration examples, the app name, and thus the app folder name, is *MyBlazorApp*. The *MyBlazorApp* folder and *web.config* file are in the *publish* folder.

**Public hosting with Windows Server operating systems**

> [!NOTE]
> When working with an Azure VM, open either or both HTTP (80) and HTTPS (443) ports to allow HTTP/HTTPS traffic to the VM. In the Azure portal, use the VM's **SETTINGS** > **Networking** > **Add inbound port rule** to open ports. to For more information, see [How to open ports to a virtual machine with the Azure portal](https://docs.microsoft.com/azure/virtual-machines/windows/nsg-quickstart-portal).

1. Use the **Add Roles and Features** wizard from the **Manage** menu or select the **Add Roles and Features** link in **Server Manager** to enable the **Web Server (IIS)** server role. In the **Server Roles** step, check the box for **Web Server (IIS)**.
1. After the **Features** step, the **Role services** step loads for Web Server (IIS). Accept the default role services provided. The default role services include **Web Server** > **Common HTTP Features** > **Static Content** and **Web Server** > **Performance** > **Static Content Compression**.
1. Proceed through the **Confirmation** step to install the web server role and services. A server/IIS restart isn't required after installing the **Web Server (IIS)** role.
1. Use the Web Platform Installer to install the [URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite):
   1. Locally, navigate to the [URL Rewrite Module downloads page](https://www.iis.net/downloads/microsoft/url-rewrite#additionalDownloads). For English, select **WebPI** to download the WebPI installer. For other languages, select the appropriate architecture for the server (x86/x64) to download the installer.
   1. Copy the installer to the server. Run the installer. Select the **Install** button and accept the license terms. A server restart isn't required after the install completes.
1. Open the **Internet Information Services (IIS) Manager** by selecting its link in the **Tools** menu of **Server Manager**.
1. In the **Connections** sidebar (left side of the window), open the server's node. Right-click the **Sites** folder and select **Add Website** from the menu.
1. Configure the website:
   1. Set the **Physical path** to the *publish* folder of the deployed Blazor app. The *publish* folder contains:
      * The *web.config* file that IIS uses to configure the website, including required redirect rules and file content types.
      * The app's folder, *MyBlazorApp*.
   1. Configure the **Binding**. Note that the **Default Web Site** is configured to use port 80 at the server's IP address. To use port 80 for the Blazor app, change the **Binding** of the **Default Web Site** to a port other than 80, such as 8000, before setting the binding for the Blazor app (or remove the **Default Web Site** from the server). When testing with the server's public IP address, the app is reachable at `http://<server-IP-address>`. Alternatively, the port of the Blazor app can be set to some other port than 80. Don't forget to set an inbound security rule for the same port if the site is hosted by an Azure VM.
   1. Select **OK** to add the website.
1. Navigate to the Blazor app on the Internet using the binding information configured for the website.

**Local hosting on Windows desktop operating systems**

1. Navigate to **Control Panel** > **Programs** > **Programs and Features** > **Turn Windows features on or off** (left side of the window).
1. Open the **Internet Information Services** node. Open the **Web Management Tools** node.
1. Check the box for **IIS Management Console**.
1. Check the box for **World Wide Web Services**.
1. Accept the default features for **World Wide Web Services**. The default features include **Common HTTP Features** > **Static Content** and **Performance Features** > **Static Content Compression**. Select the **OK** button to install the IIS features.
1. Use the Web Platform Installer to install the [URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite):
   1. Navigate to the [URL Rewrite Module downloads page](https://www.iis.net/downloads/microsoft/url-rewrite#additionalDownloads). For English, select **WebPI** to download the WebPI installer. For other languages, select the appropriate architecture for the server (x86/x64) to download the installer.
   1. Run the installer. Select the **Install** button and accept the license terms. A restart isn't required after the install completes.
1. Open the **Internet Information Services (IIS) Manager** by executing **inetmgr.exe** from a run command prompt or by searching for **IIS** and selecting the app's link in the search results.
1. In the **Connections** sidebar (left side of the window), open the server's node. Right-click the **Sites** folder and select **Add Website** from the menu.
1. Configure the website:
   1. Provide a **Site name**.
   1. Set the **Physical path** to the *publish* folder of the deployed Blazor app. The *publish* folder contains:
      * The *web.config* file that IIS uses to configure the website, including required redirect rules and file content types.
      * The app's folder, *MyBlazorApp*.
   1. Configure the **Binding**. Provide a high numbered port (for example, 8000) and leave the **Host name** blank or set it to `localhost`.
   1. Select **OK** to add the website.
1. Select the website in the **Sites** folder.
1. Navigate to the Blazor app: In the IIS Manager's **Action** sidebar (right side of the window), select the link under **Browse Website** to open the app in the default browser. If the port was set to 8000, the Blazor app is available at `http://localhost:8000`.

### Nginx

The following *nginx.conf* file is simplified to show how to configure Nginx to send the *Index.html* file whenever it cannot find a corresponding file on disk.

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

### Nginx in Docker

Add one line to the Dockerfile, as shown in the following example:

```
FROM nginx:alpine
COPY ./bin/Release/netstandard2.0/publish /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/nginx.conf
```

### GitHub Pages

Blazor apps can be hosted on [GitHub Pages](https://pages.github.com/). For an unsupported example approach, see [Rafael Pedicini's](http://www.rafaelpedicini.com/) [Single Page Apps for GitHub Pages](http://spa-github-pages.rafrex.com/) ([rafrex/spa-github-pages on GitHub](https://github.com/rafrex/spa-github-pages#readme)). Another example using the same approach can be seen at [blazor-demo/blazor-demo.github.io
 on GitHub](https://github.com/blazor-demo/blazor-demo.github.io) ([live site](https://blazor-demo.github.io/)).
