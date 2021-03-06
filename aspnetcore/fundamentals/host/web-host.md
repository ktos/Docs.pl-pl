---
title: Host sieci Web platformy ASP.NET Core
author: guardrex
description: Więcej informacji na temat hosta sieci web w programie ASP.NET Core, który jest odpowiedzialny za zarządzanie uruchamiania i czasu życia aplikacji.
ms.author: riande
ms.custom: mvc
ms.date: 09/01/2018
uid: fundamentals/host/web-host
ms.openlocfilehash: 7440ab26534840b190a346614f645860fc2b7d78
ms.sourcegitcommit: 7211ae2dd702f67d36365831c490d6178c9a46c8
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 09/07/2018
ms.locfileid: "44089902"
---
# <a name="aspnet-core-web-host"></a>Host sieci Web platformy ASP.NET Core

Przez [Luke Latham](https://github.com/guardrex)

Konfigurowanie aplikacji platformy ASP.NET Core i uruchamiania *hosta*. Host jest odpowiedzialny za zarządzanie uruchamiania i czasu życia aplikacji. Jako minimum host konfiguruje serwer i potoku przetwarzania żądań. W tym temacie omówiono hosta sieci Web platformy ASP.NET Core ([IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder)), co jest przydatne do hostowania aplikacji sieci web. Pokrycia hosta ogólnego .NET ([IHostBuilder](/dotnet/api/microsoft.extensions.hosting.ihostbuilder)), zobacz <xref:fundamentals/host/generic-host>.

## <a name="set-up-a-host"></a>Konfigurowanie hosta

::: moniker range=">= aspnetcore-2.0"

Tworzenie hosta przy użyciu wystąpienia [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder). To jest zwykle wykonywany w punkcie wejścia aplikacji, `Main` metody. W szablonach projektu `Main` znajduje się w *Program.cs*. Typowa *Program.cs* wywołania [CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) do rozpoczęcia konfigurowania hosta:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```

`CreateDefaultBuilder` wykonuje następujące zadania:

* Konfiguruje [Kestrel](xref:fundamentals/servers/kestrel) jako serwer sieci web i konfiguruje serwer przy użyciu dostawcy usług konfiguracji hosta aplikacji. Kestrel opcjach domyślny, zobacz temat <xref:fundamentals/servers/kestrel#kestrel-options>.
* Ustawia zawartość katalogu głównego ścieżka zwrócona przez element [Directory.GetCurrentDirectory](/dotnet/api/system.io.directory.getcurrentdirectory).
* Ładunki [konfiguracji hosta](#host-configuration-values) od:
  * Zmienne środowiskowe prefiksem `ASPNETCORE_` (na przykład `ASPNETCORE_ENVIRONMENT`).
  * Argumenty wiersza polecenia.
* Konfiguracja aplikacji obciążeń z:
  * *appsettings.json*.
  * *appSettings. {Środowiska} .json*.
  * [Wpisami tajnymi użytkowników](xref:security/app-secrets) uruchamiania aplikacji `Development` środowisko przy użyciu zestawu wpisu.
  * Zmienne środowiskowe.
  * Argumenty wiersza polecenia.
* Konfiguruje [rejestrowania](xref:fundamentals/logging/index) dla danych wyjściowych konsoli i debugowania. Rejestrowanie powoduje umieszczenie [filtrowanie dziennika](xref:fundamentals/logging/index#log-filtering) reguły określone w sekcji Konfiguracja rejestrowania *appsettings.json* lub *appsettings. { Środowisko} .json* pliku.
* Podczas uruchamiania za usług IIS, umożliwia [integracji usług IIS](xref:host-and-deploy/iis/index). Umożliwia skonfigurowanie ścieżki podstawowej i portów serwer nasłuchuje na w przypadku korzystania z [modułu ASP.NET Core](xref:fundamentals/servers/aspnet-core-module). Moduł tworzy zwrotny serwer proxy między usługami IIS a Kestrel. Konfiguruje również aplikacji na [przechwytywania błędów uruchamiania](#capture-startup-errors). Opcjach domyślny usług IIS, zobacz temat <xref:host-and-deploy/iis/index#iis-options>.
* Zestawy [ServiceProviderOptions.ValidateScopes](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes) do `true` w przypadku aplikacji środowiska programowania. Aby uzyskać więcej informacji, zobacz [zakresu walidacji](#scope-validation).

Konfiguracja zdefiniowane przez `CreateDefaultBuilder` zastąpienia i wzmacnia [ConfigureAppConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configureappconfiguration), [ConfigureLogging](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configurelogging)i inne metody, jak i metody rozszerzenia [ IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder). Oto kilka przykładów:

* [ConfigureAppConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configureappconfiguration) służy do określania dodatkowych `IConfiguration` dla aplikacji. Następujące `ConfigureAppConfiguration` wywołanie dodaje delegata w celu uwzględnienia konfiguracji aplikacji w *appsettings.xml* pliku. `ConfigureAppConfiguration` może zostać wywołana wiele razy. Należy pamiętać, że ta konfiguracja nie ma zastosowania do hosta (na przykład adresy URL serwera lub środowiska). Zobacz [hosta wartości konfiguracji](#host-configuration-values) sekcji.

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((hostingContext, config) =>
        {
            config.AddXmlFile("appsettings.xml", optional: true, reloadOnChange: true);
        })
        ...
    ```

* Następujące `ConfigureLogging` wywołanie dodaje delegata, aby skonfigurować poziom rejestrowania minimalny ([SetMinimumLevel](/dotnet/api/microsoft.extensions.logging.loggingbuilderextensions.setminimumlevel)) do [LogLevel.Warning](/dotnet/api/microsoft.extensions.logging.loglevel). Ustawienie to zastępuje ustawienia w *appsettings. Development.JSON* (`LogLevel.Debug`) i *appsettings. Production.JSON* (`LogLevel.Error`) skonfigurowane przez `CreateDefaultBuilder`. `ConfigureLogging` może zostać wywołana wiele razy.

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .ConfigureLogging(logging => 
        {
            logging.SetMinimumLevel(LogLevel.Warning);
        })
        ...
    ```

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

* Następujące wywołanie do `ConfigureKestrel` zastępuje to domyślne [Limits.MaxRequestBodySize](/dotnet/api/microsoft.aspnetcore.server.kestrel.core.kestrelserverlimits.maxrequestbodysize) 30,000,000 bajtów nawiązane, gdy Kestrel została skonfigurowana przez `CreateDefaultBuilder`:

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .ConfigureKestrel((context, options) =>
        {
            options.Limits.MaxRequestBodySize = 20000000;
        });
        ...
    ```

::: moniker-end

::: moniker range="= aspnetcore-2.0 || aspnetcore-2.1"

* Następujące wywołanie do [UseKestrel](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions.usekestrel) zastępuje to domyślne [Limits.MaxRequestBodySize](/dotnet/api/microsoft.aspnetcore.server.kestrel.core.kestrelserverlimits.maxrequestbodysize) 30,000,000 bajtów nawiązane, gdy Kestrel została skonfigurowana przez `CreateDefaultBuilder`:

    ```csharp
    WebHost.CreateDefaultBuilder(args)
        .UseKestrel(options =>
        {
            options.Limits.MaxRequestBodySize = 20000000;
        });
        ...
    ```

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

*Zawartości głównego* Określa, gdzie hosta wyszukuje pliki zawartości, takich jak pliki widoku MVC. Po uruchomieniu aplikacji z folderu głównego projektu, folder główny projektu jest używany jako katalogu głównego zawartości. Jest to opcja domyślna używana w [programu Visual Studio](https://www.visualstudio.com/) i [dotnet nowe szablony](/dotnet/core/tools/dotnet-new).

Aby uzyskać więcej informacji na temat konfiguracji aplikacji, zobacz <xref:fundamentals/configuration/index>.

> [!NOTE]
> Jako alternatywa dla użycia statycznego `CreateDefaultBuilder` metody tworzenia hosta z [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) metoda przy użyciu obsługiwanych za pomocą programu ASP.NET Core 2.x. Aby uzyskać więcej informacji zobacz kartę 1.x platformy ASP.NET Core.

::: moniker-end

::: moniker range="< aspnetcore-2.0"

Tworzenie hosta przy użyciu wystąpienia [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder). Tworzenie hosta jest zwykle wykonywany w punkcie wejścia aplikacji, `Main` metody. W szablonach projektu `Main` znajduje się w *Program.cs*:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        BuildWebHost(args).Run();
    }

    public static IWebHost BuildWebHost(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>()
            .Build();
}
```

`WebHostBuilder` wymaga [serwera, który implementuje IServer](xref:fundamentals/servers/index). Wbudowane serwery są [Kestrel](xref:fundamentals/servers/kestrel) i [HTTP.sys](xref:fundamentals/servers/httpsys) (przed wydaniem programu ASP.NET Core 2.0, sterownik HTTP.sys została wywołana [WebListener](xref:fundamentals/servers/weblistener)). W tym przykładzie [— metoda rozszerzenia UseKestrel](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions.usekestrel?view=aspnetcore-1.1) Określa serwer Kestrel.

*Zawartości głównego* Określa, gdzie hosta wyszukuje pliki zawartości, takich jak pliki widoku MVC. Domyślny katalog główny zawartości zostaje uzyskana dla `UseContentRoot` przez [Directory.GetCurrentDirectory](/dotnet/api/system.io.directory.getcurrentdirectory?view=netcore-1.1). Po uruchomieniu aplikacji z folderu głównego projektu, folder główny projektu jest używany jako katalogu głównego zawartości. Jest to opcja domyślna używana w [programu Visual Studio](https://www.visualstudio.com/) i [dotnet nowe szablony](/dotnet/core/tools/dotnet-new).

Aby używać usług IIS jako zwrotny serwer proxy, należy wywołać [UseIISIntegration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderiisextensions) w ramach tworzenia hosta. `UseIISIntegration` nie Konfiguruj *serwera*, takiej jak [UseKestrel](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions.usekestrel?view=aspnetcore-1.1) jest. `UseIISIntegration` Umożliwia skonfigurowanie ścieżki podstawowej i portów serwer nasłuchuje na w przypadku korzystania z [modułu ASP.NET Core](xref:fundamentals/servers/aspnet-core-module) utworzyć zwrotny serwer proxy między Kestrel i usług IIS. Usługi IIS za pomocą platformy ASP.NET Core `UseKestrel` i `UseIISIntegration` musi być określona. `UseIISIntegration` aktywuje tylko podczas uruchamiania usług IIS lub IIS Express. Aby uzyskać więcej informacji, zobacz <xref:fundamentals/servers/aspnet-core-module> i <xref:host-and-deploy/aspnet-core-module>.

Minimalny implementację, która umożliwia skonfigurowanie hosta (i aplikacji ASP.NET Core) obejmuje określenie serwerze i konfiguracji Potok żądań aplikacji:

```csharp
var host = new WebHostBuilder()
    .UseKestrel()
    .Configure(app =>
    {
        app.Run(context => context.Response.WriteAsync("Hello World!"));
    })
    .Build();

host.Run();
```

::: moniker-end

Podczas konfigurowania hostów [Konfiguruj](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configure?view=aspnetcore-1.1) i [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.configureservices?view=aspnetcore-1.1) można przedstawić metody. Jeśli `Startup` klasy jest określony, należy ją zdefiniować `Configure` metody. Aby uzyskać więcej informacji, zobacz <xref:fundamentals/startup>. Wiele wywołań `ConfigureServices` dołączenia do siebie nawzajem. Wiele wywołań `Configure` lub `UseStartup` na `WebHostBuilder` zastąpienia poprzednich ustawień.

## <a name="host-configuration-values"></a>Wartości konfiguracji hosta

[WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) opiera się na następujących podejść można ustawić wartości konfiguracji hostów:

* Konfiguracja Konstruktora hosta, który zawiera zmienne środowiskowe w formacie `ASPNETCORE_{configurationKey}`. Na przykład `ASPNETCORE_ENVIRONMENT`.
* Rozszerzenia, takie jak [UseContentRoot](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.usecontentroot) i [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration) (zobacz [zastępczą konfigurację](#override-configuration) sekcji).
* [UseSetting](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder.usesetting) i skojarzony klucz. Podczas ustawiania wartości z `UseSetting`, wartość jest ustawiana jako ciąg, niezależnie od typu.

Host używa jednego z tych opcji ustawia wartość ostatniego. Aby uzyskać więcej informacji, zobacz [zastępczą konfigurację](#override-configuration) w następnej sekcji.

### <a name="application-key-name"></a>Klucz aplikacji (nazwa)

[IHostingEnvironment.ApplicationName](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment.applicationname) właściwość ma wartość automatycznie, gdy [UseStartup](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usestartup) lub [Konfiguruj](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configure) jest wywoływana podczas konstruowania hosta. Wartość jest równa Nazwa zestawu zawierającego punkt wejścia aplikacji. Aby jawnie ustawić wartość, użyj [WebHostDefaults.ApplicationKey](/dotnet/api/microsoft.aspnetcore.hosting.webhostdefaults.applicationkey):

**Klucz**: applicationName  
**Typ**: *ciągu*  
**Domyślne**: Nazwa zestawu zawierającego punkt wejścia aplikacji.  
**Można ustawić przy użyciu**: `UseSetting`  
**Zmienna środowiskowa**: `ASPNETCORE_APPLICATIONKEY`

::: moniker range=">= aspnetcore-2.1"

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.ApplicationKey, "CustomApplicationName")
```

::: moniker-end

::: moniker range="< aspnetcore-2.1"

```csharp
var host = new WebHostBuilder()
    .UseSetting("applicationName", "CustomApplicationName")
```

::: moniker-end

### <a name="capture-startup-errors"></a>Przechwytywania błędów uruchamiania

To ustawienie steruje przechwytywania błędów uruchamiania.

**Klucz**: captureStartupErrors  
**Typ**: *bool* (`true` lub `1`)  
**Domyślne**: wartość domyślna to `false` chyba, że aplikacja jest uruchamiana z Kestrel za usług IIS, w którym domyślnie są `true`.  
**Można ustawić przy użyciu**: `CaptureStartupErrors`  
**Zmienna środowiskowa**: `ASPNETCORE_CAPTURESTARTUPERRORS`

Gdy `false`, błędy podczas uruchamiania wynik na hoście, kończenie. Gdy `true`, host przechwytuje wyjątki podczas uruchamiania i próbuje uruchomić serwer.

::: moniker range=">= aspnetcore-2.0"

```csharp
WebHost.CreateDefaultBuilder(args)
    .CaptureStartupErrors(true)
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
var host = new WebHostBuilder()
    .CaptureStartupErrors(true)
```

::: moniker-end

### <a name="content-root"></a>Zawartość katalogu głównego

To ustawienie określa, gdzie platformy ASP.NET Core rozpoczyna się wyszukiwanie plików zawartości, takich jak widoków MVC. 

**Klucz**: contentRoot  
**Typ**: *ciągu*  
**Domyślne**: wartość domyślna to folder, w którym znajduje się zestaw aplikacji.  
**Można ustawić przy użyciu**: `UseContentRoot`  
**Zmienna środowiskowa**: `ASPNETCORE_CONTENTROOT`

Główny zawartości jest również używane jako podstawową ścieżkę dla [ustawienie katalog główny sieci Web](#web-root). Jeśli ścieżka nie istnieje, host nie można uruchomić.

::: moniker range=">= aspnetcore-2.0"

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\<content-root>")
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
var host = new WebHostBuilder()
    .UseContentRoot("c:\\<content-root>")
```

::: moniker-end

### <a name="detailed-errors"></a>Szczegółowe informacje o błędach

Określa, czy szczegółowe błędy, które mają być przechwytywane.

**Klucz**: detailedErrors  
**Typ**: *bool* (`true` lub `1`)  
**Domyślne**: false  
**Można ustawić przy użyciu**: `UseSetting`  
**Zmienna środowiskowa**: `ASPNETCORE_DETAILEDERRORS`

Po włączeniu (lub gdy <a href="#environment">środowiska</a> ustawiono `Development`), aplikacja przechwytuje szczegółowe wyjątki.

::: moniker range=">= aspnetcore-2.0"

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.DetailedErrorsKey, "true")
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
var host = new WebHostBuilder()
    .UseSetting(WebHostDefaults.DetailedErrorsKey, "true")
```

::: moniker-end

### <a name="environment"></a>Środowisko

Ustawia środowiska Twojej aplikacji.

**Klucz**: środowisko  
**Typ**: *ciągu*  
**Domyślne**: produkcji  
**Można ustawić przy użyciu**: `UseEnvironment`  
**Zmienna środowiskowa**: `ASPNETCORE_ENVIRONMENT`

Środowisko można ustawić dowolną wartość. Wartości zdefiniowane w ramach obejmują `Development`, `Staging`, i `Production`. Wartości nie są z uwzględnieniem wielkości liter. Domyślnie *środowiska* są odczytywane z `ASPNETCORE_ENVIRONMENT` zmiennej środowiskowej. Korzystając z [programu Visual Studio](https://www.visualstudio.com/), zmienne środowiskowe, może być ustawiona w *launchSettings.json* pliku. Aby uzyskać więcej informacji, zobacz <xref:fundamentals/environments>.

::: moniker range=">= aspnetcore-2.0"

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseEnvironment(EnvironmentName.Development)
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
var host = new WebHostBuilder()
    .UseEnvironment(EnvironmentName.Development)
```

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

### <a name="hosting-startup-assemblies"></a>Hosting zestawy startowe

Ustawia hostingu zestawy uruchamiania aplikacji.

**Klucz**: hostingStartupAssemblies  
**Typ**: *ciągu*  
**Domyślne**: pusty ciąg  
**Można ustawić przy użyciu**: `UseSetting`  
**Zmienna środowiskowa**: `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES`

Rozdzielana średnikami ciąg hostingu zestawy startowe załadować podczas uruchamiania.

Mimo że konfiguracja ma domyślnie wartość pustego ciągu, hostingu zestawy startowe zawsze zawierać zestaw aplikacji. Hostingu zestawy startowe są udostępniane, są dodawane do zestawu aplikacji dotyczące ładowania, gdy aplikacja tworzy swoich usług wspólne podczas uruchamiania.

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2")
```

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

### <a name="https-port"></a>HTTPS Port

Ustawienie HTTPS przekierowania portu. Używane w [Wymuszanie protokołu HTTPS](xref:security/enforcing-ssl).

**Klucz**: https_port **typu**: *ciąg*
**domyślne**: nie ustawiono wartość domyślną.
**Można ustawić przy użyciu**: `UseSetting` 
 **zmiennej środowiskowej**: `ASPNETCORE_HTTPS_PORT`

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting("https_port", "8080")
```

### <a name="hosting-startup-exclude-assemblies"></a>Hosting zestawy wykluczania uruchamiania

OPIS ELEMENTU

**Klucz**: hostingStartupExcludeAssemblies  
**Typ**: *ciągu*  
**Domyślne**: pusty ciąg  
**Można ustawić przy użyciu**: `UseSetting`  
**Zmienna środowiskowa**: `ASPNETCORE_HOSTINGSTARTUPEXCLUDEASSEMBLIES`

Rozdzielana średnikami ciąg hostingu uruchamiania zestawów, które mają zostać wykluczone podczas uruchamiania.

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.HostingStartupExcludeAssembliesKey, "assembly1;assembly2")
```

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

### <a name="prefer-hosting-urls"></a>Preferuj hostingu adresów URL

Wskazuje, czy host powinien nasłuchiwać adresy URL skonfigurowano `WebHostBuilder` zamiast konfigurowane przy użyciu `IServer` implementacji.

**Klucz**: preferHostingUrls  
**Typ**: *bool* (`true` lub `1`)  
**Domyślne**: true  
**Można ustawić przy użyciu**: `PreferHostingUrls`  
**Zmienna środowiskowa**: `ASPNETCORE_PREFERHOSTINGURLS`

```csharp
WebHost.CreateDefaultBuilder(args)
    .PreferHostingUrls(false)
```

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

### <a name="prevent-hosting-startup"></a>Zapobiegaj hostingu uruchamiania

Uniemożliwia automatyczne ładowanie obsługi zestawów uruchamiania, w tym hosting zestawy startowe skonfigurowany przez zestaw aplikacji. Aby uzyskać więcej informacji, zobacz <xref:fundamentals/configuration/platform-specific-configuration>.

**Klucz**: preventHostingStartup  
**Typ**: *bool* (`true` lub `1`)  
**Domyślne**: false  
**Można ustawić przy użyciu**: `UseSetting`  
**Zmienna środowiskowa**: `ASPNETCORE_PREVENTHOSTINGSTARTUP`

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting(WebHostDefaults.PreventHostingStartupKey, "true")
```

::: moniker-end

### <a name="server-urls"></a>Adresy URL serwerów

Wskazuje adresy IP lub adresów hosta, portów i protokołów, które serwer powinien nasłuchiwać żądań protokołu.

**Klucz**: adresy URL  
**Typ**: *ciągu*  
**Domyślne**: http://localhost:5000  
**Można ustawić przy użyciu**: `UseUrls`  
**Zmienna środowiskowa**: `ASPNETCORE_URLS`

Ustaw rozdzielone średnikami (;) lista adresów URL prefiksy powinny odpowiadać serwera. Na przykład `http://localhost:123`. Użyj "\*" Aby wskazać, że serwer powinien nasłuchiwać żądań adresy IP lub nazwa hosta przy użyciu określonego portu i protokołu (na przykład `http://*:5000`). Protokół (`http://` lub `https://`) muszą być dołączone do każdego adresu URL. Obsługiwane formaty różnią się między serwerami.

::: moniker range=">= aspnetcore-2.0"

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002")
```

Kestrel ma swój własny konfiguracji punktu końcowego interfejsu API. Aby uzyskać więcej informacji, zobacz <xref:fundamentals/servers/kestrel#endpoint-configuration>.

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
var host = new WebHostBuilder()
    .UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002")
```

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

### <a name="shutdown-timeout"></a>Limit czasu zamykania

Określa ilość czasu oczekiwania dla hosta sieci web zamknąć.

**Klucz**: shutdownTimeoutSeconds  
**Typ**: *int*  
**Domyślne**: 5  
**Można ustawić przy użyciu**: `UseShutdownTimeout`  
**Zmienna środowiskowa**: `ASPNETCORE_SHUTDOWNTIMEOUTSECONDS`

Mimo że akceptuje klucz *int* z `UseSetting` (na przykład `.UseSetting(WebHostDefaults.ShutdownTimeoutKey, "10")`), [UseShutdownTimeout](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useshutdowntimeout) — metoda rozszerzenia ma [TimeSpan](/dotnet/api/system.timespan).

Podczas limitu czasu, hostingu:

* Wyzwalacze [IApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime.applicationstopping).
* Podejmuje próby zatrzymania usług hostowanych, rejestrowanie błędów, których nie można zatrzymać usługi.

Jeśli upłynie limit czasu przed wszystkimi zatrzymania usług hostowanych, wszystkie pozostałe usługi active zostaną zatrzymane podczas zamykania aplikacji. Usługi zatrzymania nawet wtedy, gdy jeszcze nie zakończyło się przetwarzanie. Jeśli usługi wymagają dodatkowego czasu, aby zatrzymać, zwiększ limit czasu.

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseShutdownTimeout(TimeSpan.FromSeconds(10))
```

::: moniker-end

### <a name="startup-assembly"></a>Zestaw startowy

Określa zestaw, aby wyszukać `Startup` klasy.

**Klucz**: startupAssembly  
**Typ**: *ciągu*  
**Domyślne**: zestaw aplikacji  
**Można ustawić przy użyciu**: `UseStartup`  
**Zmienna środowiskowa**: `ASPNETCORE_STARTUPASSEMBLY`

Zestawu według nazwy (`string`) lub wpisz (`TStartup`) mogą być przywoływane. Jeśli wiele `UseStartup` metody są wywoływane, ostatni z nich ma pierwszeństwo.

::: moniker range=">= aspnetcore-2.0"

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseStartup("StartupAssemblyName")
```

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseStartup<TStartup>()
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
var host = new WebHostBuilder()
    .UseStartup("StartupAssemblyName")
```

```csharp
var host = new WebHostBuilder()
    .UseStartup<TStartup>()
```

::: moniker-end

### <a name="web-root"></a>Katalog główny sieci Web

Ustawia ścieżkę względną do statycznych zasobów aplikacji.

**Klucz**: webroot  
**Typ**: *ciągu*  
**Domyślne**: Jeśli nie zostanie określony, wartość domyślna to "(Content Root)/wwwroot", jeśli ścieżka istnieje. Jeśli ścieżka nie istnieje, jest używany dostawca pliku pusta.  
**Można ustawić przy użyciu**: `UseWebRoot`  
**Zmienna środowiskowa**: `ASPNETCORE_WEBROOT`

::: moniker range=">= aspnetcore-2.0"

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseWebRoot("public")
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
var host = new WebHostBuilder()
    .UseWebRoot("public")
```

::: moniker-end

## <a name="override-configuration"></a>Zastąp konfigurację

Użyj [konfiguracji](xref:fundamentals/configuration/index) skonfigurować hosta sieci web. W poniższym przykładzie konfiguracja hosta jest opcjonalnie określony w *hostsettings.json* pliku. Dowolna Konfiguracja ładowane z *hostsettings.json* pliku może być zastąpiona przez argumenty wiersza polecenia. Konfiguracja wbudowanych (w `config`) służy do konfigurowania hosta z [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration). `IWebHostBuilder` Konfiguracja zostanie dodany do konfiguracji aplikacji, ale prawdą nie dotyczy&mdash; `ConfigureAppConfiguration` nie ma wpływu na `IWebHostBuilder` konfiguracji.

::: moniker range=">= aspnetcore-2.0"

Zastępowanie konfiguracji, jaką zapewnia `UseUrls` z *hostsettings.json* config argument po pierwsze, wiersza polecenia konfiguracji drugiego:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args)
    {
        var config = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("hostsettings.json", optional: true)
            .AddCommandLine(args)
            .Build();

        return WebHost.CreateDefaultBuilder(args)
            .UseUrls("http://*:5000")
            .UseConfiguration(config)
            .Configure(app =>
            {
                app.Run(context => 
                    context.Response.WriteAsync("Hello, World!"));
            })
            .Build();
    }
}
```

*hostsettings.JSON*:

```json
{
    urls: "http://*:5005"
}
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

Zastępowanie konfiguracji, jaką zapewnia `UseUrls` z *hostsettings.json* config argument po pierwsze, wiersza polecenia konfiguracji drugiego:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        var config = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("hostsettings.json", optional: true)
            .AddCommandLine(args)
            .Build();

        var host = new WebHostBuilder()
            .UseUrls("http://*:5000")
            .UseConfiguration(config)
            .UseKestrel()
            .Configure(app =>
            {
                app.Run(context => 
                    context.Response.WriteAsync("Hello, World!"));
            })
            .Build();

        host.Run();
    }
}
```

*hostsettings.JSON*:

```json
{
    urls: "http://*:5005"
}
```

::: moniker-end

> [!NOTE]
> [UseConfiguration](/dotnet/api/microsoft.aspnetcore.hosting.hostingabstractionswebhostbuilderextensions.useconfiguration) metody rozszerzenia nie jest obecnie zdolne do analizowania sekcji konfiguracji, zwracany przez `GetSection` (na przykład `.UseConfiguration(Configuration.GetSection("section"))`. `GetSection` Metoda klucze konfiguracji do sekcji żądane filtry, ale klucze nie powoduje usunięcia nazwy sekcji (na przykład `section:urls`, `section:environment`). `UseConfiguration` Metoda oczekuje kluczy, aby dopasować `WebHostBuilder` kluczy (na przykład `urls`, `environment`). Obecność nazwa sekcji na kluczach zapobiega wartości w sekcji Konfigurowanie hosta. Ten problem zostanie rozwiązany w kolejnej wersji. Aby uzyskać więcej informacji i rozwiązania problemu, zobacz [przekazywanie sekcję konfiguracji do WebHostBuilder.UseConfiguration korzysta z kluczami pełną](https://github.com/aspnet/Hosting/issues/839).
>
> `UseConfiguration` tylko kopiuje kluczy z dostarczonego `IConfiguration` konfiguracji konstruktora hosta. W związku z tym, ustawienie `reloadOnChange: true` plików ustawień JSON, INI i XML nie ma wpływu.

Aby określić hosta, uruchom na określony adres URL, żądaną wartość w można przekazać polecenie w wierszu polecenia podczas wykonywania [dotnet, uruchom](/dotnet/core/tools/dotnet-run). Argument wiersza polecenia zastępuje `urls` wartość z *hostsettings.json* pliku, a serwer nasłuchuje na porcie 8080:

```console
dotnet run --urls "http://*:8080"
```

## <a name="manage-the-host"></a>Zarządzać hosta

::: moniker range=">= aspnetcore-2.0"

**Uruchom**

`Run` Metoda uruchamia aplikację sieci web i blokuje wątek wywołujący, aż zamknie hosta:

```csharp
host.Run();
```

**Start**

Uruchamiane w sposób nieblokującej na poziomie hosta, przez wywołanie jego `Start` metody:

```csharp
using (host)
{
    host.Start();
    Console.ReadLine();
}
```

Jeśli lista adresów URL jest przekazywany do `Start` metody nasłuchuje on na określone adresy URL:

```csharp
var urls = new List<string>()
{
    "http://*:5000",
    "http://localhost:5001"
};

var host = new WebHostBuilder()
    .UseKestrel()
    .UseStartup<Startup>()
    .Start(urls.ToArray());

using (host)
{
    Console.ReadLine();
}
```

Aplikacja może inicjuje i uruchamia nowy host, przy użyciu wstępnie skonfigurowanych ustawień domyślnych z `CreateDefaultBuilder` za pomocą innej metody statycznej wygody. Te metody uruchomić serwer bez danych wyjściowych konsoli i przy [WaitForShutdown](/dotnet/api/microsoft.aspnetcore.hosting.webhostextensions.waitforshutdown) poczekaj, aż podziału (Ctrl-C/SIGINT lub SIGTERM):

**Start (RequestDelegate aplikacji)**

Rozpoczynać `RequestDelegate`:

```csharp
using (var host = WebHost.Start(app => app.Response.WriteAsync("Hello, World!")))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Wyślij żądanie w przeglądarce, aby `http://localhost:5000` odpowiedź "Hello World!" `WaitForShutdown` blokuje, dopóki nie wystawiono podziału (Ctrl-C/SIGINT lub SIGTERM). Aplikacja jest wyświetlana `Console.WriteLine` komunikat i czeka na naciśnięcie klawisza zakończyć pracę.

**Uruchom (ciąg adresu url, RequestDelegate aplikacji)**

Uruchom przy użyciu adresu URL i `RequestDelegate`:

```csharp
using (var host = WebHost.Start("http://localhost:8080", app => app.Response.WriteAsync("Hello, World!")))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

Daje ten sam wynik jako **Start (aplikacja RequestDelegate)**, chyba że aplikacja reaguje na `http://localhost:8080`.

**Uruchom (Akcja&lt;IRouteBuilder&gt; routeBuilder)**

Użyj wystąpienia `IRouteBuilder` ([Microsoft.AspNetCore.Routing](https://www.nuget.org/packages/Microsoft.AspNetCore.Routing/)) używać oprogramowania pośredniczącego routingu:

```csharp
using (var host = WebHost.Start(router => router
    .MapGet("hello/{name}", (req, res, data) => 
        res.WriteAsync($"Hello, {data.Values["name"]}!"))
    .MapGet("buenosdias/{name}", (req, res, data) => 
        res.WriteAsync($"Buenos dias, {data.Values["name"]}!"))
    .MapGet("throw/{message?}", (req, res, data) => 
        throw new Exception((string)data.Values["message"] ?? "Uh oh!"))
    .MapGet("{greeting}/{name}", (req, res, data) => 
        res.WriteAsync($"{data.Values["greeting"]}, {data.Values["name"]}!"))
    .MapGet("", (req, res, data) => res.WriteAsync("Hello, World!"))))
{
    Console.WriteLine("Use Ctrl-C to shutdown the host...");
    host.WaitForShutdown();
}
```

W przykładzie, należy użyć następujących żądań przeglądarki:

| Żądanie                                    | Odpowiedź                                 |
| ------------------------------------------ | ---------------------------------------- |
| `http://localhost:5000/hello/Martin`       | Witaj, Martin!                           |
| `http://localhost:5000/buenosdias/Catrina` | Buenos diasem, Catrina!                    |
| `http://localhost:5000/throw/ooops!`       | Zgłasza wyjątek z ciągiem "ooops!" |
| `http://localhost:5000/throw`              | Zgłasza wyjątek z ciągiem "Uh Niestety!" |
| `http://localhost:5000/Sante/Kevin`        | Sante, Jan!                            |
| `http://localhost:5000`                    | Cześć ludzie!                             |

`WaitForShutdown` blokuje, dopóki nie wystawiono podziału (Ctrl-C/SIGINT lub SIGTERM). Aplikacja jest wyświetlana `Console.WriteLine` komunikat i czeka na naciśnięcie klawisza zakończyć pracę.

**Uruchom (ciąg adresu url, Akcja&lt;IRouteBuilder&gt; routeBuilder)**

Użyj adresu URL i wystąpienie `IRouteBuilder`:

```csharp
using (var host = WebHost.Start("http://localhost:8080", router => router
    .MapGet("hello/{name}", (req, res, data) => 
        res.WriteAsync($"Hello, {data.Values["name"]}!"))
    .MapGet("buenosdias/{name}", (req, res, data) => 
        res.WriteAsync($"Buenos dias, {data.Values["name"]}!"))
    .MapGet("throw/{message?}", (req, res, data) => 
        throw new Exception((string)data.Values["message"] ?? "Uh oh!"))
    .MapGet("{greeting}/{name}", (req, res, data) => 
        res.WriteAsync($"{data.Values["greeting"]}, {data.Values["name"]}!"))
    .MapGet("", (req, res, data) => res.WriteAsync("Hello, World!"))))
{
    Console.WriteLine("Use Ctrl-C to shut down the host...");
    host.WaitForShutdown();
}
```

Daje ten sam wynik jako **Start (Akcja&lt;IRouteBuilder&gt; routeBuilder)**, chyba że aplikacja reaguje na `http://localhost:8080`.

**StartWith (Akcja&lt;IApplicationBuilder&gt; aplikacji)**

Podaj delegat konfigurujący `IApplicationBuilder`:

```csharp
using (var host = WebHost.StartWith(app => 
    app.Use(next => 
    {
        return async context => 
        {
            await context.Response.WriteAsync("Hello World!");
        };
    })))
{
    Console.WriteLine("Use Ctrl-C to shut down the host...");
    host.WaitForShutdown();
}
```

Wyślij żądanie w przeglądarce, aby `http://localhost:5000` odpowiedź "Hello World!" `WaitForShutdown` blokuje, dopóki nie wystawiono podziału (Ctrl-C/SIGINT lub SIGTERM). Aplikacja jest wyświetlana `Console.WriteLine` komunikat i czeka na naciśnięcie klawisza zakończyć pracę.

**StartWith (ciąg adresu url, Akcja&lt;IApplicationBuilder&gt; aplikacji)**

Podaj adres URL i delegat konfigurujący `IApplicationBuilder`:

```csharp
using (var host = WebHost.StartWith("http://localhost:8080", app => 
    app.Use(next => 
    {
        return async context => 
        {
            await context.Response.WriteAsync("Hello World!");
        };
    })))
{
    Console.WriteLine("Use Ctrl-C to shut down the host...");
    host.WaitForShutdown();
}
```

Daje ten sam wynik jako **StartWith (Akcja&lt;IApplicationBuilder&gt; aplikacji)**, chyba że aplikacja reaguje na `http://localhost:8080`.

::: moniker-end

::: moniker range="< aspnetcore-2.0"

**Uruchom**

`Run` Metoda uruchamia aplikację sieci web i blokuje wątek wywołujący, aż zamknie hosta:

```csharp
host.Run();
```

**Start**

Uruchamiane w sposób nieblokującej na poziomie hosta, przez wywołanie jego `Start` metody:

```csharp
using (host)
{
    host.Start();
    Console.ReadLine();
}
```

Jeśli lista adresów URL jest przekazywany do `Start` metody nasłuchuje on na określone adresy URL:

```csharp
var urls = new List<string>()
{
    "http://*:5000",
    "http://localhost:5001"
};

var host = new WebHostBuilder()
    .UseKestrel()
    .UseStartup<Startup>()
    .Start(urls.ToArray());

using (host)
{
    Console.ReadLine();
}
```

::: moniker-end

## <a name="ihostingenvironment-interface"></a>Interfejs IHostingEnvironment

[Interfejsu IHostingEnvironment](/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment) zawiera informacje dotyczące aplikacji sieci web, środowisko hostingu. Użyj [iniekcji konstruktora](xref:fundamentals/dependency-injection) uzyskać `IHostingEnvironment` aby można było używać jej właściwości i metody rozszerzenia:

```csharp
public class CustomFileReader
{
    private readonly IHostingEnvironment _env;

    public CustomFileReader(IHostingEnvironment env)
    {
        _env = env;
    }

    public string ReadFile(string filePath)
    {
        var fileProvider = _env.WebRootFileProvider;
        // Process the file here
    }
}
```

A [podejścia opartego na Konwencji](xref:fundamentals/environments#environment-based-startup-class-and-methods) mogą być używane do konfigurowania aplikacji przy uruchamianiu opartych na środowisku. Alternatywnie wstrzyknąć `IHostingEnvironment` do `Startup` konstruktora do użycia w `ConfigureServices`:

```csharp
public class Startup
{
    public Startup(IHostingEnvironment env)
    {
        HostingEnvironment = env;
    }

    public IHostingEnvironment HostingEnvironment { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        if (HostingEnvironment.IsDevelopment())
        {
            // Development configuration
        }
        else
        {
            // Staging/Production configuration
        }

        var contentRootPath = HostingEnvironment.ContentRootPath;
    }
}
```

> [!NOTE]
> Oprócz `IsDevelopment` metody rozszerzenia `IHostingEnvironment` oferuje `IsStaging`, `IsProduction`, i `IsEnvironment(string environmentName)` metody. Aby uzyskać więcej informacji, zobacz <xref:fundamentals/environments>.

`IHostingEnvironment` Usługi może także wstrzyknięte bezpośrednio do `Configure` metody do konfigurowania potoku przetwarzania:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        // In Development, use the developer exception page
        app.UseDeveloperExceptionPage();
    }
    else
    {
        // In Staging/Production, route exceptions to /error
        app.UseExceptionHandler("/error");
    }

    var contentRootPath = env.ContentRootPath;
}
```

`IHostingEnvironment` może być wstrzykuje `Invoke` metody podczas tworzenia niestandardowych [oprogramowania pośredniczącego](xref:fundamentals/middleware/index#write-middleware):

```csharp
public async Task Invoke(HttpContext context, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        // Configure middleware for Development
    }
    else
    {
        // Configure middleware for Staging/Production
    }

    var contentRootPath = env.ContentRootPath;
}
```

## <a name="iapplicationlifetime-interface"></a>Interfejs IApplicationLifetime

[IApplicationLifetime](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime) umożliwia działań po uruchamiania i zamykania. Trzy właściwości w interfejsie są tokenów anulowania, używane do rejestrowania `Action` metod, które definiują zdarzenia uruchamiania i zamykania.

| Token anulowania    | Wyzwalane, gdy&#8230; |
| --------------------- | --------------------- |
| [ApplicationStarted](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstarted) | Host pełni została uruchomiona. |
| [ApplicationStopped](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstopped) | Host jest kończonych łagodne zamykanie. Wszystkie żądania powinny zostać przetworzone. Blokuje zamknięcia, aż do zakończenia tego zdarzenia. |
| [ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstopping) | Wykonuje łagodne zamykanie hosta. Żądania mogą nadal być przetwarzane. Blokuje zamknięcia, aż do zakończenia tego zdarzenia. |

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app, IApplicationLifetime appLifetime)
    {
        appLifetime.ApplicationStarted.Register(OnStarted);
        appLifetime.ApplicationStopping.Register(OnStopping);
        appLifetime.ApplicationStopped.Register(OnStopped);

        Console.CancelKeyPress += (sender, eventArgs) =>
        {
            appLifetime.StopApplication();
            // Don't terminate the process immediately, wait for the Main thread to exit gracefully.
            eventArgs.Cancel = true;
        };
    }

    private void OnStarted()
    {
        // Perform post-startup activities here
    }

    private void OnStopping()
    {
        // Perform on-stopping activities here
    }

    private void OnStopped()
    {
        // Perform post-stopped activities here
    }
}
```

[StopApplication](/dotnet/api/microsoft.aspnetcore.hosting.iapplicationlifetime.stopapplication) żądania zakończenia aplikacji. Następujące klasy używa `StopApplication` bezpiecznie zamknąć aplikację po klasy `Shutdown` metoda jest wywoływana:

```csharp
public class MyClass
{
    private readonly IApplicationLifetime _appLifetime;

    public MyClass(IApplicationLifetime appLifetime)
    {
        _appLifetime = appLifetime;
    }

    public void Shutdown()
    {
        _appLifetime.StopApplication();
    }
}
```

## <a name="scope-validation"></a>Weryfikacja zakresu

::: moniker range=">= aspnetcore-2.0"

[CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) ustawia [ServiceProviderOptions.ValidateScopes](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions.validatescopes) do `true` w przypadku aplikacji środowiska programowania.

::: moniker-end

Gdy `ValidateScopes` ustawiono `true`, domyślny dostawca usługi sprawdza upewnij się, że:

* Usługi o określonym zakresie nie są bezpośrednio lub pośrednio rozwiązane od dostawcy usług w katalogu głównego.
* Usługi o określonym zakresie nie są bezpośrednio lub pośrednio wprowadzony do pojedynczych wystąpień.

Dostawcy usług główny jest tworzone, gdy [BuildServiceProvider](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectioncontainerbuilderextensions.buildserviceprovider) jest wywoływana. Okres istnienia dostawcy usług głównego odnosi się do aplikacji/serwera. okres istnienia, gdy dostawca rozpoczyna się od aplikacji i zostanie usunięty podczas zamykania aplikacji.

Usługi o określonym zakresie są usuwane przez kontener, który je utworzył. Jeśli usługi o określonym zakresie zostanie utworzony w kontenerze katalogu głównego, okres istnienia usługi skutecznie zostanie podwyższony do pojedynczego wystąpienia, ponieważ tylko są usuwane przez nadrzędny kontener, gdy serwer/aplikacji zostanie zamknięta. Sprawdzanie poprawności usługi zakresy przechwytuje tych sytuacji gdy `BuildServiceProvider` jest wywoływana.

Aby zawsze weryfikowały zakresy, w tym w środowisku produkcyjnym, należy skonfigurować [ServiceProviderOptions](/dotnet/api/microsoft.extensions.dependencyinjection.serviceprovideroptions) z [UseDefaultServiceProvider](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usedefaultserviceprovider) w Konstruktorze hosta:

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseDefaultServiceProvider((context, options) => {
        options.ValidateScopes = true;
    })
```

::: moniker range="= aspnetcore-2.0"

## <a name="troubleshooting-systemargumentexception"></a>Rozwiązywanie problemów z System.ArgumentException

**Następujące ma zastosowanie tylko do aplikacji ASP.NET Core 2.0, gdy aplikacja nie wywołać `UseStartup` lub `Configure`.**

Host mogły zostać utworzone przez iniekcję `IStartup` bezpośrednio do kontenera iniekcji zależności zamiast wywoływania `UseStartup` lub `Configure`:

```csharp
services.AddSingleton<IStartup, Startup>();
```

Jeśli host jest wbudowana w ten sposób, może wystąpić następujący błąd:

```
Unhandled Exception: System.ArgumentException: A valid non-empty application name must be provided.
```

Dzieje się tak dlatego, nazwę aplikacji (nazwa bieżącego zestawu) jest wymagany do skanowania w poszukiwaniu `HostingStartupAttributes`. Jeśli aplikacja ręcznie wprowadza `IStartup` do kontenera iniekcji zależności, Dodaj następujące wywołanie do `WebHostBuilder` o podanej nazwie zestawu:

```csharp
WebHost.CreateDefaultBuilder(args)
    .UseSetting("applicationName", "AssemblyName")
```

Możesz też dodać dummy `Configure` do `WebHostBuilder`, który automatycznie ustawia nazwę aplikacji:

```csharp
WebHost.CreateDefaultBuilder(args)
    .Configure(_ => { })
```

Aby uzyskać więcej informacji, zobacz [anonsów: Microsoft.Extensions.PlatformAbstractions został usunięty (komentarz)](https://github.com/aspnet/Announcements/issues/237#issuecomment-323786938) i [przykładowe StartupInjection](https://github.com/aspnet/Hosting/blob/8377d226f1e6e1a97dabdb6769a845eeccc829ed/samples/SampleStartups/StartupInjection.cs).

::: moniker-end

## <a name="additional-resources"></a>Dodatkowe zasoby

* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/linux-nginx>
* <xref:host-and-deploy/linux-apache>
* <xref:host-and-deploy/windows-service>
