---
# required metadata

title: NuGet.Server to Host NuGet Feeds | Microsoft Docs
author: kraigb
ms.author: kraigb
manager: ghogen
ms.date: 8/25/2017
ms.topic: article
ms.prod: nuget
#ms.service:
ms.technology: null
ms.assetid: 45138f80-9717-42c2-8b34-9a1bc1fb3eab

# optional metadata

description: How to create and host a NuGet package feed on any server running IIS using NuGet.Server, making packages available through HTTP and OData.
keywords: NuGet feed, NuGet gallery, remote package feed, NuGet.Server
#ROBOTS:
#audience:
#ms.devlang:
ms.reviewer:
- karann
- unnir
#ms.suite:
#ms.tgt_pltfrm:
#ms.custom:

---

# NuGet.Server

NuGet.Server is a package provided by the .NET Foundation that creates an ASP.NET application that can host a package feed on any server that runs IIS. Simply said, NuGet.Server makes a folder on the server available through HTTP(S) (specifically OData). It's easy to set up and is best for simple scenarios.

1. Create an empty ASP.NET Web application in Visual Studio and add the NuGet.Server package to it.
1. Configure the `Packages` folder in the application and add packages.
1. Deploy the application to a suitable server.

The following sections walk through this process in detail, using C#.

## Create and deploy an ASP.NET Web application with NuGet.Server

1. In Visual Studio, select **File > New > Project**, set the target framework for .NET Framework 4.6 (see below), search for "ASP.NET", and select the **ASP.NET Web Application (.NET Framework)** template for C#.

    ![Setting .NET Framework target to 4.6](media/Hosting_01-NuGet.Server-Set4.6.png)

1. Give the application a suitable name *other* than NuGet.Server, select OK, and in the next dialog select the **Empty** template and select OK.

1. Right-click the project, select **Manage NuGet Packages**, and in the Package Manager UI search and install the latest version of the NuGet.Server package if you're targeting .NET Framework 4.6. (You can also install it from the Package Manager Console with `Install-Package NuGet.Server`.)

    ![Installing the NuGet.Server package](media/Hosting_02-NuGet.Server-Package.png)

    > [!Important]
    > If your web application targets .NET Framework 4.5.2, you must install NuGet Server **2.10.3** instead.

1. Installing NuGet.Server converts the empty Web application into a package source. It creates a `Packages` folder in the application and overwrites `web.config` to include additional settings (see the comments in that file for details).

1. To make packages available in the feed when you publish the application to a server, add their `.nupkg` files to the `Packages` folder in Visual Studio, then set their **Build Action** to **Content** and **Copy to Output Directory** to **Copy always**:

    ![Copying packages to the Packages folder in the project](media/Hosting_03-NuGet.Server-Package-Folder.png)

1. Run the site locally in Visual Studio (without debugging, that is Ctrl+F5). The home page provides the package feed URLs:

    ![Default home page for an application with NuGet.Server](media/Hosting_04-NuGet.Server-FeedHomePage.png)

1. Click on **here** in the area outlined above to see the OData feed of packages.

1. The first time you run the application, NuGet.Server restructures the `Packages` folder to contain a folder for each package. This matches the [local storage layout](http://blog.nuget.org/20151118/nuget-3.3.html#folder-based-repository-commands) introduced with NuGet 3.3 to improve performance. When adding more packages, continue to follow this structure.

1. Once you've tested your local deployment, deploy the application to any other internal or external site as needed.
1. Once deployed to `http://<domain>`, the URL that you use for the package source will be `http://<domain>/nuget`.


## Configuring the Packages folder

With `NuGet.Server` 1.5 and later, you can more specifically configure the package folder using the `appSetting/packagesPath` value in `web.config`:

```xml
<appSettings>
    <!-- Set the value here to specify your custom packages folder. -->
    <add key="packagesPath" value="C:\MyPackages" />
</appSettings>
```

`packagesPath` can be an absolute or virtual path.

When `packagesPath` is omitted or left blank, the packages folder is the default `~/Packages`.

## Adding packages to the feed externally

Once a NuGet.Server site is running, you can add or delete packages using nuget.exe provided that you set an API key value in `web.config`.

After installing the NuGet.Server package, `web.config` contains an empty `appSetting/apiKey` value:

```xml
<appSettings>
    <add key="apiKey" value="" />
</appSettings>
```

When `apiKey` is omitted or blank, pushing packages to the feed is disabled.

To enable this capability, set the `apiKey` to a value (ideally a strong password) and add a key called `appSettings/requireApiKey` with the value of `true`:

```xml
<appSettings>
        <!-- Sets whether an API Key is required to push/delete packages -->
    <add key="requireApiKey" value="true" />

    <!-- Set a shared password (for all users) to push/delete packages -->
    <add key="apiKey" value="" />
</appSettings>
```

If your server is already secured or you do not otherwise require an API key (for example, when using a private server on a local team network), you can set `requireApiKey` to `false`. All users with access to the server can then push or delete packages.