---
uid: signalr/overview/security/persistent-connection-authorization
title: Uwierzytelnianie i autoryzacja połączeń trwałych SignalR | Dokumentacja firmy Microsoft
author: pfletcher
description: W tym temacie opisano sposób wymusić autoryzację dla trwałego połączenia. Aby uzyskać ogólne informacje na temat integracji zabezpieczeń do aplikacji SignalR...
ms.author: riande
ms.date: 06/10/2014
ms.assetid: e264677b-9c01-47ec-94f9-3cd8f08f94af
msc.legacyurl: /signalr/overview/security/persistent-connection-authorization
msc.type: authoredcontent
ms.openlocfilehash: e7ae160cbe4c5f6cdb393768758f5bdec4203dbf
ms.sourcegitcommit: 45ac74e400f9f2b7dbded66297730f6f14a4eb25
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 08/16/2018
ms.locfileid: "41756057"
---
<a name="authentication-and-authorization-for-signalr-persistent-connections"></a>Uwierzytelnianie i autoryzacja połączeń trwałych SignalR
====================
przez [Patrick Fletcher](https://github.com/pfletcher), [Tom FitzMacken](https://github.com/tfitzmac)

> W tym temacie opisano sposób wymusić autoryzację dla trwałego połączenia. Aby uzyskać ogólne informacje na temat integracji zabezpieczeń do aplikacji SignalR zobacz [wprowadzenie do zabezpieczeń](introduction-to-security.md). 
> 
> ## <a name="software-versions-used-in-this-topic"></a>Wersje oprogramowania używaną w tym temacie
> 
> 
> - [Visual Studio 2013](https://www.microsoft.com/visualstudio/eng/2013-downloads)
> - .NET 4.5
> - SignalR w wersji 2
>   
> 
> 
> ## <a name="previous-versions-of-this-topic"></a>Poprzednie wersje tego tematu
> 
> Aby uzyskać informacje dotyczące starszych wersji biblioteki SignalR, zobacz [starsze wersje biblioteki SignalR](../older-versions/index.md).
> 
> ## <a name="questions-and-comments"></a>Pytania i komentarze
> 
> Jak się podoba w tym samouczku, i co można było ulepszyć proces w komentarzach u dołu strony, wystaw opinię. Jeśli masz pytania, na które nie są bezpośrednio związane z tego samouczka, możesz zamieścić je do [forum ASP.NET SignalR](https://forums.asp.net/1254.aspx/1?ASP+NET+SignalR) lub [StackOverflow.com](http://stackoverflow.com/).


## <a name="enforce-authorization"></a>Wymuszanie autoryzacji

Aby wymusić reguł autoryzacji, korzystając z [PersistentConnection](https://msdn.microsoft.com/library/microsoft.aspnet.signalr.persistentconnection(v=vs.111).aspx) konieczne jest przesłonięcie `AuthorizeRequest` metody. Nie można użyć `Authorize` atrybutu z połączeń trwałych. `AuthorizeRequest` Metoda jest wywoływana przez SignalR Framework przed każdym żądaniem, aby sprawdzić, czy użytkownik jest autoryzowany do wykonania żądanej akcji. `AuthorizeRequest` Metoda nie jest wywoływana przez klienta; zamiast tego należy uwierzytelnić użytkownika za pośrednictwem mechanizmu uwierzytelniania standardowego w Twojej aplikacji.

W poniższym przykładzie pokazano sposób ograniczania żądań do uwierzytelnionych użytkowników.

[!code-csharp[Main](persistent-connection-authorization/samples/sample1.cs)]

Można dodać wszelka logika autoryzacji dostosowanego w metodzie AuthorizeRequest; takich jak sprawdzanie, czy użytkownik należy do określonej roli.
