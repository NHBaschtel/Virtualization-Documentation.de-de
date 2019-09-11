---
title: Konfigurieren der APP für die Verwendung eines Gruppen verwalteten Dienstkontos
description: Konfigurieren von Apps für die Verwendung von Gruppen-Managed-Service-Konten (gMSAs) für Windows-Container.
keywords: docker, Container, Active Directory, GMSA, apps, Anwendungen, Group Managed Service-Konto, Gruppen-Managed-Service-Konten, Konfiguration
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 234909d7f0cb0f30ee7fbf4796dd0381bfbff89f
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079747"
---
# <a name="configure-your-app-to-use-a-gmsa"></a>Konfigurieren der APP für die Verwendung eines gMSA

In der typischen Konfiguration erhält ein Container nur ein Group Managed Service-Konto (gMSA), das verwendet wird, wenn das Container Computerkonto versucht, sich bei Netzwerkressourcen zu authentifizieren. Dies bedeutet, dass Ihre APP als **Lokales System** oder als **Netzwerkdienst** ausgeführt werden muss, wenn Sie die gMSA-Identität verwenden muss.

## <a name="run-an-iis-app-pool-as-network-service"></a>Ausführen eines IIS-App-Pools als Netzwerkdienst

Wenn Sie eine IIS-Website in Ihrem Container hosten, müssen Sie lediglich die gMSA-Identität Ihres App-Pools auf **Netzwerkdienst**einstellen. Sie können dies in Ihrem Dockerfile tun, indem Sie den folgenden Befehl hinzufügen:

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

Wenn Sie zuvor statische Benutzeranmeldeinformationen für Ihren IIS-App-Pool verwendet haben, sollten Sie die gMSA als Ersatz für diese Anmeldeinformationen verwenden. Sie können die gMSA zwischen dev-, Test-und Produktionsumgebungen ändern, und IIS übernimmt automatisch die aktuelle Identität, ohne das Container Bild ändern zu müssen.

## <a name="run-a-windows-service-as-network-service"></a>Ausführen eines Windows-Diensts als Netzwerkdienst

Wenn Ihre Container-App als Windows-Dienst ausgeführt wird, können Sie den Dienst so einrichten, dass er in Ihrem Dockerfile als **Netzwerkdienst** ausgeführt wird:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

## <a name="run-arbitrary-console-apps-as-network-service"></a>Ausführen beliebiger Konsolen-Apps als Netzwerkdienst

Bei generischen Konsolen-apps, die nicht in IIS oder Service Manager gehostet werden, ist es häufig am einfachsten, den Container als **Netzwerkdienst** auszuführen, damit die APP automatisch den gMSA-Kontext erbt. Dieses Feature steht ab Version 1709 von Windows Server zur Verfügung.

Fügen Sie Ihrer Dockerfile die folgende Zeile hinzu, damit Sie standardmäßig als Netzwerkdienst ausgeführt wird:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

Sie können auch eine einmalige Verbindung mit `docker exec`einem Container als Netzwerkdienst herstellen. Dies ist besonders hilfreich, wenn Sie Verbindungsprobleme in einem ausgeführten Container beheben möchten, wenn der Container normalerweise nicht als Netzwerkdienst ausgeführt wird.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="next-steps"></a>Nächste Schritte

Neben der Konfiguration von Apps können Sie auch gMSAs verwenden, um Folgendes zu tun:

- [Ausführen von Containern](gmsa-run-container.md)
- [Orchestrierungs Container](gmsa-orchestrate-containers.md)

Wenn während des Setups Probleme auftreten, finden Sie in unserem [Leitfaden zur Problembehandlung](gmsa-troubleshooting.md) mögliche Lösungen.
