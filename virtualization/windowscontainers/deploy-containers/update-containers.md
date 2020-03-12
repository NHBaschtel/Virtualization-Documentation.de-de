---
title: Aktualisieren von Windows Server-Containern
description: Hier erfahren Sie, wie Windows Container versionsübergreifend erstellen und ausführen kann.
keywords: Metadaten, Container, Version
author: heidilohr
ms. author: helohr
manager: lizross
ms.date: 03/10/2020
ms.openlocfilehash: 12a60398a12437ea733b2da31aae853a7afd2c57
ms.sourcegitcommit: 8eedfdc1fda9d0abb36e28dc2b5fb39891777364
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/10/2020
ms.locfileid: "79037417"
---
# <a name="update-windows-server-containers"></a>Aktualisieren von Windows Server-Containern

Als Teil der täglichen Wartung von Windows Server veröffentlichen wir aktualisierte Windows Server-Basis Betriebssystem-Container Images in regelmäßigen Abständen. Mit diesen Updates können Sie das entwickeln aktualisierter Container Images automatisieren oder Sie manuell aktualisieren, indem Sie die neueste Version abrufen. Windows Server-Container verfügen über keinen Wartungs Stapel wie Windows Server. Sie können in einem Container keine Updates erhalten, wie Sie es mit Windows Server tun. Aus diesem Grund erstellen wir jeden Monat die Windows Server-Basis Betriebssystem-Container Images mit den Updates und veröffentlichen die aktualisierten Container Images.

Andere Container Images, z. b. .net oder IIS, werden basierend auf den aktualisierten Basis Betriebssystem-Container Images neu erstellt und monatlich veröffentlicht.

## <a name="how-to-get-windows-server-container-updates"></a>Vorgehensweise beim erhalten von Windows Server-Container Updates

Wir aktualisieren Windows Server-Basis Betriebssystem-Container Images in Übereinstimmung mit dem Windows-Wartungs Rhythmus. Aktualisierte Container Images werden am zweiten Dienstag jedes Monats veröffentlicht, was gelegentlich als "B"-Release bezeichnet wird, wobei eine Präfix Nummer auf dem releasemonat basiert. Wir nennen beispielsweise unser Februar-Update "2B" und unser März-Update "3B". Dieses monatliche Update Ereignis ist das einzige reguläre Release, das neue Sicherheitskorrekturen umfasst.

Der Server, auf dem diese Container gehostet werden, die als Container Host oder nur als "Host" bezeichnet werden, kann während zusätzlicher Update Ereignisse als "B"-Releases gewartet werden. Weitere Informationen zum Windows Update-Wartungs Rhythmus finden Sie im Blogbeitrag zu [Windows Update-Wartungs Kadenz](https://techcommunity.microsoft.com/t5/windows-it-pro-blog/windows-10-update-servicing-cadence/ba-p/222376) .

Neue Windows Server-Basis Betriebssystem-Container Images werden kurz nach dem zweiten Dienstag jedes Monats in der Microsoft-Container Registry (MCR) an dem zweiten Dienstag eines Monats geöffnet, und die vorgestellten Tags Zielen auf die neueste Version von B ab. Beispiele:

- ltsc2019 [(LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc): docker Pull MCR.Microsoft.com/Windows/ServerCore:ltsc2019
- 1909 [(SAC)](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel): docker Pull MCR.Microsoft.com/Windows/ServerCore:1909

Wenn Sie mit docker Hub vertraut sind als MCR, erhalten Sie in [diesem Blogbeitrag](https://azure.microsoft.com/blog/microsoft-syndicates-container-catalog/) eine ausführlichere Erläuterung.  

Für jede Version wird auch das entsprechende Container Image mit zwei zusätzlichen Tags für die Revisionsnummer und die KB-Artikelnummer für bestimmte Container Image Revisionen veröffentlicht. Beispiel:

- docker Pull MCR.Microsoft.com/Windows/ServerCore:10.0.17763.1040
- docker Pull MCR.Microsoft.com/Windows/ServerCore:1809-KB4546852

In diesen Beispielen wird das Windows Server 2019 Server Core-Container Image mit dem Update der Sicherheitsversion vom 18. Februar abgerufen.  

Eine umfassende Liste der Windows Server-Basis Betriebssystem-Container Images, Versionen und ihrer jeweiligen Tags finden Sie in diesen [Windows-Basis Betriebssystem-Container Images](https://hub.docker.com/_/microsoft-windows-base-os-images) auf docker Hub.

Monatlich Serviced Serviced Windows Server-Images, die von Microsoft auf Azure Marketplace veröffentlicht werden, verfügen ebenfalls über vorinstallierte Basis Betriebssystem Container-Images. Weitere Informationen finden Sie auf unserer [Preisseite für Windows Server Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/microsoftwindowsserver.windowsserver?tab=PlansAndPrice). Wir aktualisieren diese Images in der Regel über fünf Arbeitstage nach der Freigabe "B".

Eine umfassende Liste der Windows Server-Images und-Versionen finden Sie unter [Windows Server Release on Azure Marketplace Update History](https://support.microsoft.com/help/4497947/windows-server-release-on-azure-marketplace-update-history).

## <a name="host-and-container-version-compatibility"></a>Kompatibilität von Host-und Container Versionen

Es gibt zwei Arten von Isolations Modi für Windows-Container: Prozess Isolation und Hyper-V-Isolation. Die Hyper-V-Isolation ist flexibler, wenn es um die Kompatibilität von Host-und Container Versionen geht. Weitere Informationen finden Sie unter [Versions Kompatibilität](version-compatibility.md) und [Isolations Modi](../manage-containers/hyperv-container.md). Dieser Abschnitt konzentriert sich auf Prozess isolierte Container, sofern nichts anderes angegeben ist.

Wenn Sie den Container Host oder das Container Image mit den monatlichen Updates aktualisieren, solange das Host-und das Container Image unterstützt werden (Windows Server-Version 1809 oder höher), müssen die Host-und Container Image Revisionen nicht mit dem Start des Containers verglichen werden. wird normal ausgeführt.

Möglicherweise treten jedoch Probleme auf, wenn Sie Windows Server-Container mit der Sicherheitsupdate Version von Februar, 2020 (auch als "2B" bezeichnet) oder spätere monatliche Sicherheitsupdate Releases verwenden. Weitere Informationen finden Sie in [diesem Microsoft-Support Artikel](https://support.microsoft.com/help/4542617/you-might-encounter-issues-when-using-windows-server-containers-with-t) . Diese Probleme ergaben sich aus einer Sicherheitsänderung, die eine Schnittstelle zwischen Benutzermodus und Kernel Modus erforderte, um die Sicherheit Ihrer Anwendungen sicherzustellen. Diese Probleme treten nur für isolierte Prozess Container auf, weil isolierte Prozess isolierte Container den Kernel Modus mit dem Container Host gemeinsam verwenden. Dies bedeutet, dass Container Images ohne die aktualisierte Benutzermoduskomponente unsicher und mit der neuen gesicherten Kernel Schnittstelle nicht kompatibel sind.

Wir haben seit dem 18. Februar 2020 eine Korrektur veröffentlicht. In dieser neuen Version wurde eine "neue Baseline" eingeführt. Diese neue Baseline folgt diesen Regeln:

- Eine beliebige Kombination aus Hosts und Containern, die sich vor 2B befinden, funktioniert.
- Jede Kombination aus Hosts und Containern, die sich nach 2B befinden, funktioniert.
- Eine beliebige Kombination aus Hosts und Containern auf unterschiedlichen Seiten der neuen Baseline funktioniert nicht. Beispielsweise funktionieren ein 3B-Host und ein 1B-Container nicht.

Wir verwenden die monatliche Sicherheitsupdate Version von März 2020 als Beispiel, um Ihnen zu zeigen, wie diese neuen Kompatibilitäts Regeln funktionieren. In der folgenden Tabelle heißt die Sicherheitsupdate Version von März 2020 "3B", das Update von Februar 2020 ist "2B", und das Update vom Januar 2020 ist "1B".

| Host | Container | Kompatibilität |
|---|---|---|
| 3B | 3B | Ja |
| 3B | 2B | Ja |
| 3B | 1B oder früher | Nein |
| 2B | 3B | Ja |
| 2B | 2B | Ja |
| 2B | 1B oder früher | Nein |
| 1B oder früher | 3B | Nein |
| 1B oder früher | 2B | Nein |
| 1B oder früher | 1B oder früher | Ja |

In der folgenden Tabelle sind die Versionsnummern für Basis Betriebssystem-Container Images mit 1B-, 2B-und 3B-monatlichen Sicherheitsupdate Releases von Windows Server 2016 bis zur neuesten Version von Windows Server, Version 1909, aufgeführt.

| Windows Server-Version (Floating-Tag) | Update Version für 1/14/20 Release (1B)| Update Version für 2/18/20 Release (2B) | Update Version für 3/10/20 Release (3B) |
|---|---|---|---|
| Windows Server 2016 (ltsc2016) | 10.0.14393.3443 (B4534271) | 10.0.14393.3506 (KB4546850) | In den nächsten Tagen frei zugebender Container |
| Windows Server, Version 1803 (1803) | 10.0.17134.1246 (KB4534293) | 10.0.17134.1305 (KB4546851)  | Diese Version hat das Ende der Unterstützung erreicht. Weitere Informationen finden Sie unter Lebens [Zyklus der Basis Image Wartung](base-image-lifecycle.md).|
| Windows Server 2019 (ltsc2019) | 10.0.17763.973 (KB4534273) | 10.0.17763.1040 (KB4546852) | 10.0.17763.1098 (KB4538461) |
| Windows Server, Version 1903 (1903) |10.0.18362.592 (KB4528760) | 10.0.18362.658 (KB4546853) | 10.0.18362.719 (KB4540673) |
| Windows Server, Version 1909 (1909) | 10.0.18363.592 (KB4528760) | 10.0.18363.658 (KB4546853) | 10.0.18363.719 (KB4540673) |

## <a name="troubleshoot-host-and-container-image-mismatches"></a>Problembehandlung bei Host-und Container Image stimmen nicht überein.

Bevor Sie beginnen, stellen Sie sicher, dass Sie sich mit den Informationen unter [Versions Kompatibilität](version-compatibility.md)vertraut machen. Anhand dieser Informationen können Sie herausfinden, ob Ihr Problem durch nicht übereinstimmende Patches verursacht wurde. Wenn Sie als Ursache nicht übereinstimmende Patches erstellt haben, können Sie die Anweisungen in diesem Abschnitt befolgen, um das Problem zu beheben.

### <a name="query-the-version-of-your-container-host"></a>Abfragen der Version des Container Hosts

Wenn Sie auf den Container Host zugreifen können, können Sie den `ver` Befehl ausführen, um die Betriebssystemversion zu erhalten. Wenn Sie z. b. `ver` auf einem System ausführen, auf dem Windows Server 2019 mit der neuesten Sicherheitsupdate Version vom Februar 2020 ausgeführt wird, wird Folgendes angezeigt:

```batch
Microsoft Windows [Version 10.0.17763.1039]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>ver 

Microsoft Windows [Version 10.0.17763.1039]
```

>[!NOTE]
>Die Revisionsnummer in diesem Beispiel wird als 1039 und nicht als 1040 angezeigt, da die Sicherheitsupdate Version Feb 2020 nicht über eine Out-of-Band-Version von 2B für Windows Server verfügt. Es gab nur eine Out-of-Band-2B-Version für Container, die eine Revisionsnummer von 1040 enthielt.

Wenn Sie keinen direkten Zugriff auf den Container Host haben, wenden Sie sich an Ihren IT-Administrator. Wenn Sie in der Cloud ausgeführt werden, überprüfen Sie die Website des cloudanbieters, um herauszufinden, welche Container Host-Betriebssystemversion ausgeführt wird. Wenn Sie z. b. Azure Kubernetes Service (AKS) verwenden, finden Sie die Version des Host Betriebssystems in den Anmerkungen zur Version der [AKS](https://github.com/Azure/AKS/releases)-Version.

### <a name="query-the-version-of-your-container-image"></a>Abfragen der Version Ihres Container Images

Befolgen Sie diese Anweisungen, um herauszufinden, welche Version von Ihrem Container ausgeführt wird:

1. Führen Sie das folgende Cmdlet in PowerShell aus:

    ```powershell
    docker images
    ```

    Die Ausgabe sollte etwa wie folgt aussehen:

     ```powershell
     REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
     mcr.microsoft.com/windows/servercore   ltsc2019            b456290f487c        4 weeks ago         4.84GB
     mcr.microsoft.com/windows              1809                58229ca44fa7        4 weeks ago         12GB
     mcr.microsoft.com/windows/nanoserver   1809                f519d4f3a868        4 weeks ago         251M

2. Run the `docker inspect` command for the Image ID of the container image that isn't working. This will tell you which version the container image is targeting.

   For example, let's say we `run docker inspect` for an ltsc 2019 container image:

   ```powershell
   docker inspect b456290f487c

       "Architecture": "amd64",

        "Os": "windows",

        "OsVersion": "10.0.17763.1039",

        "Size": 4841309825,

        "VirtualSize": 4841309825,
    ```

    In diesem Beispiel wird die Betriebssystemversion des Containers als `10.0.17763.1039`angezeigt.

    Wenn Sie bereits einen Container ausführen, können Sie auch den `ver` Befehl innerhalb des Containers selbst ausführen, um die Version zu erhalten. Wenn Sie z. b. `ver` in einem Server Core-Container Image von Windows Server 2019 mit der neuesten Sicherheitsupdate Version vom Februar 2020 ausführen, wird Folgendes angezeigt:

    ```batch
    Microsoft Windows [Version 10.0.17763.1040]
    (c) 2020 Microsoft Corporation. All rights reserved.

    C:\>ver

    Microsoft Windows [Version 10.0.17763.1040]
    ```
