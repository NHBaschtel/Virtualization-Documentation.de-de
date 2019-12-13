---
title: Konfigurieren Ihrer APP für die Verwendung eines Gruppen verwalteten Dienst Kontos
description: Konfigurieren von Apps für die Verwendung von Gruppen verwalteten Dienst Konten (Group Managed Service Accounts, gmsas) für Windows-Container.
keywords: docker, Container, Active Directory, GMSA, apps, Anwendungen, Gruppen verwaltete Dienst Konten, Gruppen verwaltete Dienst Konten, Konfiguration
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 6635381d5f7ddbebf7bdea4624af241b9f6a6864
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909790"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>Konfigurieren Ihrer APP für die Verwendung eines GMSA

In der typischen Konfiguration erhält ein Container nur ein Gruppen verwaltetes Dienst Konto (Group Managed Service Account, GMSA), das immer dann verwendet wird, wenn das Container Computer Konto versucht, sich bei Netzwerkressourcen zu authentifizieren. Dies bedeutet, dass Ihre APP als **Lokales System** oder **Netzwerkdienst** ausgeführt werden muss, wenn Sie die GMSA-Identität verwenden muss.

## <a name="run-an-iis-app-pool-as-network-service"></a>Ausführen eines IIS-App-Pools als Netzwerkdienst

Wenn Sie eine IIS-Website in Ihrem Container gehostet haben, müssen Sie nur für die Nutzung des GMSA-Diensts Ihre APP-Pool-Identität auf **Netzwerkdienst**festlegen. Sie können dies in ihrer dockerfile-Datei durchführen, indem Sie den folgenden Befehl hinzufügen:

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

Wenn Sie zuvor statische Benutzer Anmelde Informationen für Ihren IIS-App-Pool verwendet haben, sollten Sie den GMSA als Ersatz für diese Anmelde Informationen verwenden. Sie können das GMSA zwischen Entwicklungs-, Test-und Produktionsumgebungen ändern, und IIS übernimmt automatisch die aktuelle Identität, ohne das Container Image ändern zu müssen.

## <a name="run-a-windows-service-as-network-service"></a>Ausführen eines Windows-Dienstanbieter als Netzwerkdienst

Wenn Ihre containerisierte App als Windows-Dienst ausgeführt wird, können Sie den Dienst für die Ausführung als **Netzwerkdienst** in ihrer dockerfile-Datei festlegen:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>Beliebige Konsolen-Apps als Netzwerkdienst ausführen

Bei generischen Konsolen-apps, die nicht in IIS oder Service Manager gehostet werden, ist es oft am einfachsten, den Container als **Netzwerkdienst** auszuführen, sodass die APP automatisch den GMSA-Kontext erbt. Diese Funktion ist ab Windows Server-Version 1709 verfügbar.

Fügen Sie der dockerfile-Datei die folgende Zeile hinzu, damit Sie standardmäßig als Netzwerkdienst ausgeführt wird:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

Sie können auch eine einmalige Verbindung mit einem Container als Netzwerkdienst herstellen, indem Sie `docker exec`. Dies ist besonders nützlich, wenn Sie Konnektivitätsprobleme in einem laufenden Container beheben, wenn der Container normalerweise nicht als Netzwerkdienst ausgeführt wird.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>Nächste Schritte

Zusätzlich zum Konfigurieren von-Apps können Sie gmsas auch für Folgendes verwenden:

- [Ausführen von Containern](gmsa-run-container.md)
- [Orchestrieren von Containern](gmsa-orchestrate-containers.md)

Wenn während des Setups Probleme auftreten, finden Sie in unserem [Handbuch zur Problem](gmsa-troubleshooting.md) Behandlung mögliche Lösungen.
