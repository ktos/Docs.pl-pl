---
title: Zwiększ możliwości aplikacji z zewnętrznego zestawu w ASP.NET Core z IHostingStartup
author: guardrex
description: Wykryj jak zwiększyć aplikacji platformy ASP.NET Core z zewnętrznego zestawu przy użyciu implementacji IHostingStartup.
manager: wpickett
monikerRange: '>= aspnetcore-2.0'
ms.author: riande
ms.custom: mvc
ms.date: 12/07/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: fundamentals/configuration/platform-specific-configuration
ms.openlocfilehash: 618cb4349dcff696db37012af3aee844b82974f2
ms.sourcegitcommit: a0b6319c36f41cdce76ea334372f6e14fc66507e
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/02/2018
ms.locfileid: "34729054"
---
# <a name="enhance-an-app-from-an-external-assembly-in-aspnet-core-with-ihostingstartup"></a><span data-ttu-id="45f49-103">Zwiększ możliwości aplikacji z zewnętrznego zestawu w ASP.NET Core z IHostingStartup</span><span class="sxs-lookup"><span data-stu-id="45f49-103">Enhance an app from an external assembly in ASP.NET Core with IHostingStartup</span></span>

<span data-ttu-id="45f49-104">Przez [Luke Latham](https://github.com/guardrex)</span><span class="sxs-lookup"><span data-stu-id="45f49-104">By [Luke Latham](https://github.com/guardrex)</span></span>

<span data-ttu-id="45f49-105">[IHostingStartup](/dotnet/api/microsoft.aspnetcore.hosting.ihostingstartup) implementacji umożliwia dodawania rozszerzenia do aplikacji przy uruchamianiu z zewnętrznego zestawu poza aplikacji `Startup` klasy.</span><span class="sxs-lookup"><span data-stu-id="45f49-105">An [IHostingStartup](/dotnet/api/microsoft.aspnetcore.hosting.ihostingstartup) implementation allows adding enhancements to an app at startup from an external assembly outside of the app's `Startup` class.</span></span> <span data-ttu-id="45f49-106">Na przykład można użyć biblioteki zewnętrznych narzędzi `IHostingStartup` wdrożenia jest zapewnienie dodatkowej konfiguracji dostawcy usług do aplikacji.</span><span class="sxs-lookup"><span data-stu-id="45f49-106">For example, an external tooling library can use an `IHostingStartup` implementation to provide additional configuration providers or services to an app.</span></span> <span data-ttu-id="45f49-107">`IHostingStartup` *jest dostępna w programie ASP.NET Core 2.0 lub nowszej.*</span><span class="sxs-lookup"><span data-stu-id="45f49-107">`IHostingStartup` *is available in ASP.NET Core 2.0 and later.*</span></span>

<span data-ttu-id="45f49-108">[Wyświetlić lub pobrać przykładowy kod](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/configuration/platform-specific-configuration/sample/) ([sposobu pobierania](xref:tutorials/index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="45f49-108">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/configuration/platform-specific-configuration/sample/) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

## <a name="discover-loaded-hosting-startup-assemblies"></a><span data-ttu-id="45f49-109">Odnajdywanie załadowanych zestawów uruchomienia hostingu</span><span class="sxs-lookup"><span data-stu-id="45f49-109">Discover loaded hosting startup assemblies</span></span>

<span data-ttu-id="45f49-110">Aby dowiedzieć się, że hosting uruchamiania zestawów załadowanych przez aplikację lub bibliotek, Włącz rejestrowanie i sprawdź dzienniki zdarzeń aplikacji.</span><span class="sxs-lookup"><span data-stu-id="45f49-110">To discover hosting startup assemblies loaded by the app or by libraries, enable logging and check the application logs.</span></span> <span data-ttu-id="45f49-111">Błędów występujących podczas ładowania zestawów są rejestrowane.</span><span class="sxs-lookup"><span data-stu-id="45f49-111">Errors that occur when loading assemblies are logged.</span></span> <span data-ttu-id="45f49-112">Załadowanych zestawów uruchomienia hostingu są zalogowani na poziomie debugowania i są rejestrowane wszystkie błędy.</span><span class="sxs-lookup"><span data-stu-id="45f49-112">Loaded hosting startup assemblies are logged at the Debug level, and all errors are logged.</span></span>

<span data-ttu-id="45f49-113">Odczyty aplikacji przykładowej [HostingStartupAssembliesKey](/dotnet/api/microsoft.aspnetcore.hosting.webhostdefaults.hostingstartupassemblieskey) do `string` tablicy i wyświetlane w aplikacji indeksu strony:</span><span class="sxs-lookup"><span data-stu-id="45f49-113">The sample app reads the [HostingStartupAssembliesKey](/dotnet/api/microsoft.aspnetcore.hosting.webhostdefaults.hostingstartupassemblieskey) into a `string` array and displays the result in the app's Index page:</span></span>

[!code-csharp[](platform-specific-configuration/sample/HostingStartupSample/Pages/Index.cshtml.cs?name=snippet1&highlight=14-16)]

## <a name="disable-automatic-loading-of-hosting-startup-assemblies"></a><span data-ttu-id="45f49-114">Wyłącz automatyczne ładowanie hosting zestawy uruchamiania</span><span class="sxs-lookup"><span data-stu-id="45f49-114">Disable automatic loading of hosting startup assemblies</span></span>

<span data-ttu-id="45f49-115">Istnieją dwa sposoby wyłączyć automatyczne ładowanie hosting zestawy uruchamiania:</span><span class="sxs-lookup"><span data-stu-id="45f49-115">There are two ways to disable the automatic loading of hosting startup assemblies:</span></span>

* <span data-ttu-id="45f49-116">Ustaw [uniemożliwiają uruchamianie Hosting](xref:fundamentals/host/web-host#prevent-hosting-startup) hosta ustawienia konfiguracji.</span><span class="sxs-lookup"><span data-stu-id="45f49-116">Set the [Prevent Hosting Startup](xref:fundamentals/host/web-host#prevent-hosting-startup) host configuration setting.</span></span>
* <span data-ttu-id="45f49-117">Ustaw `ASPNETCORE_PREVENTHOSTINGSTARTUP` zmiennej środowiskowej.</span><span class="sxs-lookup"><span data-stu-id="45f49-117">Set the `ASPNETCORE_PREVENTHOSTINGSTARTUP` environment variable.</span></span>

<span data-ttu-id="45f49-118">Jeśli ustawienie hosta lub zmienna środowiskowa jest równa `true` lub `1`, hostingu uruchamiania zestawy nie są automatycznie załadowany.</span><span class="sxs-lookup"><span data-stu-id="45f49-118">When either the host setting or the environment variable is set to `true` or `1`, hosting startup assemblies aren't automatically loaded.</span></span> <span data-ttu-id="45f49-119">Jeśli są ustawione, ustawienie hosta steruje zachowaniem.</span><span class="sxs-lookup"><span data-stu-id="45f49-119">If both are set, the host setting controls the behavior.</span></span>

<span data-ttu-id="45f49-120">Wyłączenie obsługi zestawów uruchamiania przy użyciu zmiennej ustawienie lub środowisko hosta wyłącza je globalnie i wyłączyć pewne cechy aplikacji.</span><span class="sxs-lookup"><span data-stu-id="45f49-120">Disabling hosting startup assemblies using the host setting or environment variable disables them globally and may disable several characteristics of an app.</span></span> <span data-ttu-id="45f49-121">Nie można obecnie selektywnego wyłączania hostingu zestawu uruchamiania dodane przez bibliotekę, chyba że biblioteki udostępnia własną opcję konfiguracji.</span><span class="sxs-lookup"><span data-stu-id="45f49-121">It isn't currently possible to selectively disable a hosting startup assembly added by a library unless the library offers its own configuration option.</span></span> <span data-ttu-id="45f49-122">Przyszłych wydaniach oferuje możliwość selektywnego wyłączania hostingu zestawy uruchamiania (zobacz [GitHub wystawiać aspnet/hostingu #1243](https://github.com/aspnet/Hosting/pull/1243)).</span><span class="sxs-lookup"><span data-stu-id="45f49-122">A future release will offer the ability to selectively disable hosting startup assemblies (see [GitHub issue aspnet/Hosting #1243](https://github.com/aspnet/Hosting/pull/1243)).</span></span>

## <a name="implement-ihostingstartup"></a><span data-ttu-id="45f49-123">Implementowanie IHostingStartup</span><span class="sxs-lookup"><span data-stu-id="45f49-123">Implement IHostingStartup</span></span>

### <a name="create-the-assembly"></a><span data-ttu-id="45f49-124">Tworzenie zestawu</span><span class="sxs-lookup"><span data-stu-id="45f49-124">Create the assembly</span></span>

<span data-ttu-id="45f49-125">`IHostingStartup` Rozszerzenie jest wdrożona jako zestawu oparte na aplikację konsoli bez punktu wejścia.</span><span class="sxs-lookup"><span data-stu-id="45f49-125">An `IHostingStartup` enhancement is deployed as an assembly based on a console app without an entry point.</span></span> <span data-ttu-id="45f49-126">Odwołania do zestawów [Microsoft.AspNetCore.Hosting.Abstractions](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.Abstractions/) pakietu:</span><span class="sxs-lookup"><span data-stu-id="45f49-126">The assembly references the [Microsoft.AspNetCore.Hosting.Abstractions](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.Abstractions/) package:</span></span>

[!code-xml[](platform-specific-configuration/snapshot_sample/StartupEnhancement.csproj)]

<span data-ttu-id="45f49-127">A [HostingStartup](/dotnet/api/microsoft.aspnetcore.hosting.hostingstartupattribute) atrybut określa klasę jako implementacja `IHostingStartup` ładowania oraz wykonywania podczas kompilowania [IWebHost](/dotnet/api/microsoft.aspnetcore.hosting.iwebhost).</span><span class="sxs-lookup"><span data-stu-id="45f49-127">A [HostingStartup](/dotnet/api/microsoft.aspnetcore.hosting.hostingstartupattribute) attribute identifies a class as an implementation of `IHostingStartup` for loading and execution when building the [IWebHost](/dotnet/api/microsoft.aspnetcore.hosting.iwebhost).</span></span> <span data-ttu-id="45f49-128">W poniższym przykładzie, przestrzeń nazw jest `StartupEnhancement`, a klasa `StartupEnhancementHostingStartup`:</span><span class="sxs-lookup"><span data-stu-id="45f49-128">In the following example, the namespace is `StartupEnhancement`, and the class is `StartupEnhancementHostingStartup`:</span></span>

[!code-csharp[](platform-specific-configuration/snapshot_sample/StartupEnhancement.cs?name=snippet1)]

<span data-ttu-id="45f49-129">Implementuje klasę `IHostingStartup`.</span><span class="sxs-lookup"><span data-stu-id="45f49-129">A class implements `IHostingStartup`.</span></span> <span data-ttu-id="45f49-130">Klasa [Konfiguruj](/dotnet/api/microsoft.aspnetcore.hosting.ihostingstartup.configure) używa metody [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) można dodać rozszerzenia do aplikacji:</span><span class="sxs-lookup"><span data-stu-id="45f49-130">The class's [Configure](/dotnet/api/microsoft.aspnetcore.hosting.ihostingstartup.configure) method uses an [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) to add enhancements to an app:</span></span>

[!code-csharp[](platform-specific-configuration/snapshot_sample/StartupEnhancement.cs?name=snippet2&highlight=3,5)]

<span data-ttu-id="45f49-131">Podczas kompilowania `IHostingStartup` projektu pliku zależności (*\*. deps.json*) ustawia `runtime` lokalizacji zestawu *bin* folderu:</span><span class="sxs-lookup"><span data-stu-id="45f49-131">When building an `IHostingStartup` project, the dependencies file (*\*.deps.json*) sets the `runtime` location of the assembly to the *bin* folder:</span></span>

[!code-json[](platform-specific-configuration/snapshot_sample/StartupEnhancement1.deps.json?range=2-13&highlight=8)]

<span data-ttu-id="45f49-132">Wyświetlane jest tylko część pliku.</span><span class="sxs-lookup"><span data-stu-id="45f49-132">Only part of the file is shown.</span></span> <span data-ttu-id="45f49-133">Nazwa zestawu, w tym przykładzie jest `StartupEnhancement`.</span><span class="sxs-lookup"><span data-stu-id="45f49-133">The assembly name in the example is `StartupEnhancement`.</span></span>

### <a name="update-the-dependencies-file"></a><span data-ttu-id="45f49-134">Zaktualizuj plik zależności</span><span class="sxs-lookup"><span data-stu-id="45f49-134">Update the dependencies file</span></span>

<span data-ttu-id="45f49-135">Lokalizacja środowiska uruchomieniowego jest określona w  *\*. deps.json* pliku.</span><span class="sxs-lookup"><span data-stu-id="45f49-135">The runtime location is specified in the *\*.deps.json* file.</span></span> <span data-ttu-id="45f49-136">Aktywny ulepszeń `runtime` element musi określać lokalizację ulepszenie zestawu środowiska wykonawczego.</span><span class="sxs-lookup"><span data-stu-id="45f49-136">To active the enhancement, the `runtime` element must specify the location of the enhancement's runtime assembly.</span></span> <span data-ttu-id="45f49-137">Prefiks `runtime` lokalizacji z `lib/<TARGET_FRAMEWORK_MONIKER>/`:</span><span class="sxs-lookup"><span data-stu-id="45f49-137">Prefix the `runtime` location with `lib/<TARGET_FRAMEWORK_MONIKER>/`:</span></span>

[!code-json[](platform-specific-configuration/snapshot_sample/StartupEnhancement2.deps.json?range=2-13&highlight=8)]

<span data-ttu-id="45f49-138">W przykładowej aplikacji, modyfikacja  *\*. deps.json* plików jest wykonywane przez [PowerShell](/powershell/scripting/powershell-scripting) skryptu.</span><span class="sxs-lookup"><span data-stu-id="45f49-138">In the sample app, modification of the *\*.deps.json* file is performed by a [PowerShell](/powershell/scripting/powershell-scripting) script.</span></span> <span data-ttu-id="45f49-139">Skrypt programu PowerShell jest automatycznie wyzwalane przez element docelowy kompilacji w pliku projektu.</span><span class="sxs-lookup"><span data-stu-id="45f49-139">The PowerShell script is automatically triggered by a build target in the project file.</span></span>

### <a name="enhancement-activation"></a><span data-ttu-id="45f49-140">Rozszerzenie aktywacji</span><span class="sxs-lookup"><span data-stu-id="45f49-140">Enhancement activation</span></span>

<span data-ttu-id="45f49-141">**Umieść plik zestawu**</span><span class="sxs-lookup"><span data-stu-id="45f49-141">**Place the assembly file**</span></span>

<span data-ttu-id="45f49-142">`IHostingStartup` Implementacji w pliku zestawu musi być *bin*-wdrożone w aplikacji lub umieszczone w [magazynu środowiska uruchomieniowego](/dotnet/core/deploying/runtime-store):</span><span class="sxs-lookup"><span data-stu-id="45f49-142">The `IHostingStartup` implementation's assembly file must be *bin*-deployed in the app or placed in the [runtime store](/dotnet/core/deploying/runtime-store):</span></span>

<span data-ttu-id="45f49-143">Do użytku dla poszczególnych użytkowników należy umieścić zestaw w magazynie środowiska uruchomieniowego profilu użytkownika:</span><span class="sxs-lookup"><span data-stu-id="45f49-143">For per-user use, place the assembly in the user profile's runtime store at:</span></span>

```
<DRIVE>\Users\<USER>\.dotnet\store\x64\<TARGET_FRAMEWORK_MONIKER>\<ENHANCEMENT_ASSEMBLY_NAME>\<ENHANCEMENT_VERSION>\lib\<TARGET_FRAMEWORK_MONIKER>\
```

<span data-ttu-id="45f49-144">Do użytku globalnego należy umieścić zestaw w magazynie obsługi instalacji platformy .NET Core:</span><span class="sxs-lookup"><span data-stu-id="45f49-144">For global use, place the assembly in the .NET Core installation's runtime store:</span></span>

```
<DRIVE>\Program Files\dotnet\store\x64\<TARGET_FRAMEWORK_MONIKER>\<ENHANCEMENT_ASSEMBLY_NAME>\<ENHANCEMENT_VERSION>\lib\<TARGET_FRAMEWORK_MONIKER>\
```

<span data-ttu-id="45f49-145">Podczas wdrażania zestawu w magazynie środowiska uruchomieniowego, można wdrożyć także pliku symboli, ale nie jest wymagane w celu poprawy do pracy.</span><span class="sxs-lookup"><span data-stu-id="45f49-145">When deploying the assembly to the runtime store, the symbols file may be deployed as well but isn't required for the enhancement to work.</span></span>

<span data-ttu-id="45f49-146">**Umieść plik zależności**</span><span class="sxs-lookup"><span data-stu-id="45f49-146">**Place the dependencies file**</span></span>

<span data-ttu-id="45f49-147">Implementacja  *\*. deps.json* plik musi być w dostępnej lokalizacji.</span><span class="sxs-lookup"><span data-stu-id="45f49-147">The implementation's *\*.deps.json* file must be in an accessible location.</span></span>

<span data-ttu-id="45f49-148">Do użytku dla poszczególnych użytkowników, należy umieścić plik w `additonalDeps` folderu profilu użytkownika `.dotnet` ustawienia:</span><span class="sxs-lookup"><span data-stu-id="45f49-148">For per-user use, place the file in the `additonalDeps` folder of the user profile's `.dotnet` settings:</span></span> 

```
<DRIVE>\Users\<USER>\.dotnet\x64\additionalDeps\<ENHANCEMENT_ASSEMBLY_NAME>\shared\Microsoft.NETCore.App\2.1.0\
```

<span data-ttu-id="45f49-149">Do użytku globalnego, należy umieścić plik w `additonalDeps` folderu instalacji platformy .NET Core:</span><span class="sxs-lookup"><span data-stu-id="45f49-149">For global use, place the file in the `additonalDeps` folder of the .NET Core installation:</span></span>

```
<DRIVE>\Program Files\dotnet\additionalDeps\<ENHANCEMENT_ASSEMBLY_NAME>\shared\Microsoft.NETCore.App\2.1.0\
```

<span data-ttu-id="45f49-150">Sprawdź wersję, `2.1.0`, odzwierciedla wersji udostępnionego środowisko uruchomieniowe, które korzysta z aplikacji docelowej.</span><span class="sxs-lookup"><span data-stu-id="45f49-150">Note the version, `2.1.0`, reflects the version of the shared runtime that the target app uses.</span></span> <span data-ttu-id="45f49-151">Udostępniony środowiska uruchomieniowego jest wyświetlany w obszarze  *\*. runtimeconfig.json* pliku.</span><span class="sxs-lookup"><span data-stu-id="45f49-151">The shared runtime is shown in the *\*.runtimeconfig.json* file.</span></span> <span data-ttu-id="45f49-152">W przykładowej aplikacji, udostępnionych środowiska uruchomieniowego jest określona w *HostingStartupSample.runtimeconfig.json* pliku.</span><span class="sxs-lookup"><span data-stu-id="45f49-152">In the sample app, the shared runtime is specified in the *HostingStartupSample.runtimeconfig.json* file.</span></span>

<span data-ttu-id="45f49-153">**Ustaw zmienne środowiskowe**</span><span class="sxs-lookup"><span data-stu-id="45f49-153">**Set environment variables**</span></span>

<span data-ttu-id="45f49-154">Ustaw następujące zmienne środowiskowe w kontekście aplikacji, który używa poprawy.</span><span class="sxs-lookup"><span data-stu-id="45f49-154">Set the following environment variables in the context of the app that uses the enhancement.</span></span>

<span data-ttu-id="45f49-155">ASPNETCORE\_HOSTINGSTARTUPASSEMBLIES</span><span class="sxs-lookup"><span data-stu-id="45f49-155">ASPNETCORE\_HOSTINGSTARTUPASSEMBLIES</span></span>

<span data-ttu-id="45f49-156">Tylko zestawy uruchomienia hostingu są skanowane pod kątem `HostingStartupAttribute`.</span><span class="sxs-lookup"><span data-stu-id="45f49-156">Only hosting startup assemblies are scanned for the `HostingStartupAttribute`.</span></span> <span data-ttu-id="45f49-157">Nazwa zestawu wdrożenia znajduje się w tej zmiennej środowiskowej.</span><span class="sxs-lookup"><span data-stu-id="45f49-157">The assembly name of the implementation is provided in this environment variable.</span></span> <span data-ttu-id="45f49-158">Przykładowa aplikacja ustawia tę wartość na `StartupDiagnostics`.</span><span class="sxs-lookup"><span data-stu-id="45f49-158">The sample app sets this value to `StartupDiagnostics`.</span></span>

<span data-ttu-id="45f49-159">Wartość można również ustawić za pomocą [Hosting zestawy uruchamiania](xref:fundamentals/host/web-host#hosting-startup-assemblies) hosta ustawienia konfiguracji.</span><span class="sxs-lookup"><span data-stu-id="45f49-159">The value can also be set using the [Hosting Startup Assemblies](xref:fundamentals/host/web-host#hosting-startup-assemblies) host configuration setting.</span></span>

<span data-ttu-id="45f49-160">DOTNET\_DODATKOWYCH\_DEPS</span><span class="sxs-lookup"><span data-stu-id="45f49-160">DOTNET\_ADDITIONAL\_DEPS</span></span>

<span data-ttu-id="45f49-161">Lokalizacja wdrożenia  *\*. deps.json* pliku.</span><span class="sxs-lookup"><span data-stu-id="45f49-161">The location of the implementation's *\*.deps.json* file.</span></span>

<span data-ttu-id="45f49-162">Jeśli plik znajduje się w profilu użytkownika *.dotnet* folderu do użycia na użytkownika:</span><span class="sxs-lookup"><span data-stu-id="45f49-162">If the file is placed in the user profile's *.dotnet* folder for per-user use:</span></span>

```
<DRIVE>\Users\<USER>\.dotnet\x64\additionalDeps\
```

<span data-ttu-id="45f49-163">Jeśli plik znajduje się w instalacji platformy .NET Core do użytku globalnego, należy wprowadzić pełną ścieżkę do pliku:</span><span class="sxs-lookup"><span data-stu-id="45f49-163">If the file is placed in the .NET Core installation for global use, provide the full path to the file:</span></span>

```
<DRIVE>\Program Files\dotnet\additionalDeps\<ENHANCEMENT_ASSEMBLY_NAME>\shared\Microsoft.NETCore.App\2.1.0\<ENHANCEMENT_ASSEMBLY_NAME>.deps.json
```

<span data-ttu-id="45f49-164">Przykładowa aplikacja ustawia tę wartość na:</span><span class="sxs-lookup"><span data-stu-id="45f49-164">The sample app sets this value to:</span></span>

```
%UserProfile%\.dotnet\x64\additionalDeps\StartupDiagnostics\
```

<span data-ttu-id="45f49-165">Przykłady sposobu ustawiania zmiennych środowiskowych dla różnych systemów operacyjnych, zobacz [używać wiele środowisk](xref:fundamentals/environments).</span><span class="sxs-lookup"><span data-stu-id="45f49-165">For examples of how to set environment variables for various operating systems, see [Use multiple environments](xref:fundamentals/environments).</span></span>

## <a name="sample-app"></a><span data-ttu-id="45f49-166">Przykładowa aplikacja</span><span class="sxs-lookup"><span data-stu-id="45f49-166">Sample app</span></span>

<span data-ttu-id="45f49-167">[Przykładowa aplikacja](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/configuration/platform-specific-configuration/sample/) ([sposobu pobierania](xref:tutorials/index#how-to-download-a-sample)) używa `IHostingStartup` na utworzenie narzędzia diagnostyczne.</span><span class="sxs-lookup"><span data-stu-id="45f49-167">The [sample app](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/configuration/platform-specific-configuration/sample/) ([how to download](xref:tutorials/index#how-to-download-a-sample)) uses `IHostingStartup` to create a diagnostics tool.</span></span> <span data-ttu-id="45f49-168">Dodaje dwa middlewares do aplikacji podczas uruchamiania, aby uzyskać informacje diagnostyczne:</span><span class="sxs-lookup"><span data-stu-id="45f49-168">The tool adds two middlewares to the app at startup that provide diagnostic information:</span></span>

* <span data-ttu-id="45f49-169">Zarejestrowane usługi</span><span class="sxs-lookup"><span data-stu-id="45f49-169">Registered services</span></span>
* <span data-ttu-id="45f49-170">Adres: schemat, hosta, podstawa ścieżki, ścieżka, ciąg zapytania</span><span class="sxs-lookup"><span data-stu-id="45f49-170">Address: scheme, host, path base, path, query string</span></span>
* <span data-ttu-id="45f49-171">Połączenie: zdalny adres IP, port zdalny lokalny adres IP, portu lokalnego, certyfikatu klienta</span><span class="sxs-lookup"><span data-stu-id="45f49-171">Connection: remote IP, remote port, local IP, local port, client certificate</span></span>
* <span data-ttu-id="45f49-172">Nagłówki żądania</span><span class="sxs-lookup"><span data-stu-id="45f49-172">Request headers</span></span>
* <span data-ttu-id="45f49-173">Zmienne środowiskowe</span><span class="sxs-lookup"><span data-stu-id="45f49-173">Environment variables</span></span>

<span data-ttu-id="45f49-174">Aby uruchomić przykład:</span><span class="sxs-lookup"><span data-stu-id="45f49-174">To run the sample:</span></span>

1. <span data-ttu-id="45f49-175">Uruchamianie diagnostyki projekt korzysta z [środowiska PowerShell](/powershell/scripting/powershell-scripting) można zmodyfikować jego *StartupDiagnostics.deps.json* pliku.</span><span class="sxs-lookup"><span data-stu-id="45f49-175">The Startup Diagnostic project uses [PowerShell](/powershell/scripting/powershell-scripting) to modify its *StartupDiagnostics.deps.json* file.</span></span> <span data-ttu-id="45f49-176">Środowisko PowerShell jest instalowany domyślnie na systemu operacyjnego Windows, począwszy od systemu Windows 7 z dodatkiem SP1 i Windows Server 2008 R2 z dodatkiem SP1.</span><span class="sxs-lookup"><span data-stu-id="45f49-176">PowerShell is installed by default on Windows OS starting with Windows 7 SP1 and Windows Server 2008 R2 SP1.</span></span> <span data-ttu-id="45f49-177">Aby uzyskać programu PowerShell na innych platformach, zobacz [Instalowanie programu Windows PowerShell](/powershell/scripting/setup/installing-windows-powershell).</span><span class="sxs-lookup"><span data-stu-id="45f49-177">To obtain PowerShell on other platforms, see [Installing Windows PowerShell](/powershell/scripting/setup/installing-windows-powershell).</span></span>
2. <span data-ttu-id="45f49-178">Skompiluj projekt startowy diagnostycznych.</span><span class="sxs-lookup"><span data-stu-id="45f49-178">Build the Startup Diagnostic project.</span></span> <span data-ttu-id="45f49-179">Element docelowy kompilacji w pliku projektu:</span><span class="sxs-lookup"><span data-stu-id="45f49-179">A build target in the project file:</span></span>
   * <span data-ttu-id="45f49-180">Przenosi zestaw i pliki do profilu użytkownika środowiska uruchomieniowego magazynu symboli.</span><span class="sxs-lookup"><span data-stu-id="45f49-180">Moves the assembly and symbols files to the user profile's runtime store.</span></span>
   * <span data-ttu-id="45f49-181">Uruchamia skrypt programu PowerShell w celu zmodyfikowania *StartupDiagnostics.deps.json* pliku.</span><span class="sxs-lookup"><span data-stu-id="45f49-181">Triggers the PowerShell script to modify the *StartupDiagnostics.deps.json* file.</span></span>
   * <span data-ttu-id="45f49-182">Przenosi *StartupDiagnostics.deps.json* pliku profilu użytkownika `additionalDeps` folderu.</span><span class="sxs-lookup"><span data-stu-id="45f49-182">Moves the *StartupDiagnostics.deps.json* file to the user profile's `additionalDeps` folder.</span></span>
3. <span data-ttu-id="45f49-183">Ustaw zmienne środowiskowe:</span><span class="sxs-lookup"><span data-stu-id="45f49-183">Set the environment variables:</span></span>
    * <span data-ttu-id="45f49-184">`ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`: `StartupDiagnostics`</span><span class="sxs-lookup"><span data-stu-id="45f49-184">`ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`: `StartupDiagnostics`</span></span>
    * <span data-ttu-id="45f49-185">`DOTNET_ADDITIONAL_DEPS`: `%UserProfile%\.dotnet\x64\additionalDeps\StartupDiagnostics\`</span><span class="sxs-lookup"><span data-stu-id="45f49-185">`DOTNET_ADDITIONAL_DEPS`: `%UserProfile%\.dotnet\x64\additionalDeps\StartupDiagnostics\`</span></span>
4. <span data-ttu-id="45f49-186">Uruchamianie przykładowej aplikacji.</span><span class="sxs-lookup"><span data-stu-id="45f49-186">Run the sample app.</span></span>
5. <span data-ttu-id="45f49-187">Żądanie `/services` punktu końcowego, aby zobaczyć aplikacji w zarejestrowany usług.</span><span class="sxs-lookup"><span data-stu-id="45f49-187">Request the `/services` endpoint to see the app's registered services.</span></span> <span data-ttu-id="45f49-188">Żądanie `/diag` punktu końcowego, aby wyświetlić informacje diagnostyczne.</span><span class="sxs-lookup"><span data-stu-id="45f49-188">Request the `/diag` endpoint to see the diagnostic information.</span></span>