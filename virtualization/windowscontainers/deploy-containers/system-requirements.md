---
title: Anforderungen von Windows-Containern
description: Anforderungen von Windows-Containern.
keywords: Metadaten, Container
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 478305ff2298a0392935f9857febc445c1199b83
ms.sourcegitcommit: 5300274fd7b88c6cf5e37b2f4c02779efaa3a613
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 01/08/2019
ms.locfileid: "8996044"
---
# <a name="windows-container-requirements"></a>Anforderungen von Windows-Containern

In diesen Handbüchern sind die Anforderungen für einen Windows-Containerhost aufgeführt.

## <a name="os-requirements"></a>Betriebssystemanforderungen

- Das Windows-Container-Feature ist nur verfügbar auf Windows Server 2016 (Core und mit Desktopdarstellung), Windows 10 Professional und Enterprise (Anniversary Edition) und höher.
- Die Hyper-V-Rolle muss vor der Ausführung des Hyper-V-Containers installiert werden.
- Bei Windows Server-Containerhosts muss Windows auf Laufwerk c:\ installiert werden. Diese Einschränkung gilt nicht, wenn nur Hyper-V Container bereitgestellt werden.

## <a name="virtualized-container-hosts"></a>Virtualisierte Containerhosts

Wenn ein Windows-Containerhost von einem virtuellen Hyper-V-Computer ausgeführt wird und auch Hyper-V-Container hostet, muss die geschachtelte Virtualisierung aktiviert werden. Für die geschachtelte Virtualisierung ist Folgendes erforderlich:

- Mindestens 4GB verfügbarer Arbeitsspeicher (RAM) für den virtualisierten Hyper-V-Host
- Windows Server 2019, Windows Server Version 1803, Windows Server, Version 1709, Windows Server 2016 oder Windows 10 auf dem Hostsystem und Windows Server (vollständig, Core) auf dem virtuellen Computer.
- Ein Prozessor mit Intel VT-x (dieses Feature steht zurzeit nur für Intel-Prozessoren zur Verfügung)
- Die Containerhost-VM benötigt zudem mindestens 2 virtuelle Prozessoren.

## <a name="supported-base-images"></a>Unterstützte Basisimages

Windows-Container werden vier Basisimages zur Verfügung angeboten: Windows Server Core, Nano Server, Windows und IoT Core. Nicht alle Konfigurationen unterstützen beide Betriebssystemimages. Diese Tabelle enthält Details zu den unterstützten Konfigurationen.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Hostbetriebssystem</center></th>
<th><center>Windows Server-Container</center></th>
<th><center>Hyper-V-Container</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>WindowsServer 2016 / 2019 (Standard oder Datacenter)</center></td>
<td><center>Servercore, Nano Server Windows</center></td>
<td><center>Servercore, Nano Server Windows</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Nano Server</center></td>
<td><center>Servercore, Nano Server Windows</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro/Enterprise</center></td>
<td><center>Nicht verfügbar</center></td>
<td><center>Servercore, Nano Server Windows</center></td>
</tr>
<tr valign="top">
<td><center>IoT Core</center></td>
<td><center>IoT Core</center></td>
<td><center>Nicht verfügbar</center></td>
</tr>
</tbody>
</table>

> [!Warning]  
> <span id="warn-1">Ab Version 1709 von Windows Server ist Nano Server nicht mehr als Containerhost verfügbar.</span>


### <a name="memory-requirements"></a>Arbeitsspeicheranforderungen
Einschränkungen des für Container verfügbaren Speichers können über [Ressourcenkontrollen](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls) oder durch Überladen eines Containerhosts konfiguriert werden.  Wieviel Arbeitsspeicher zum Starten eines Containers und zum Ausführen grundlegender Befehle (ipconfig, dir, usw.) mindestens vorhanden sein muss, ist unten aufgeführt.  __Bitte beachten Sie, dass diese Werte weder eine gemeinsame Nutzung von Ressourcen durch Container noch die Anforderungen der Anwendung berücksichtigen, die im Container ausgeführt wird.  Beispielsweise kann ein Host mit 512 MB freiem Speicher mehrere Server Core-Container unter Hyper-V-Isolierung ausführen, da diese Container Ressourcen gemeinsam nutzen.__

#### <a name="windows-server-2016"></a>Windows Server 2016
| Base Image  | Windows Server-Container | Hyper-V-Isolierung    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB Auslagerungsdatei |
| Server Core | 50 MB                     | 325 MB + 1 GB Auslagerungsdatei |

#### <a name="windows-server-version-1709"></a>Windows Server, Version 1709
| Base Image  | Windows Server-Container | Hyper-V-Isolierung    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB Auslagerungsdatei |
| Server Core | 45 MB                     | 360 MB + 1 GB Auslagerungsdatei |


### <a name="base-image-differences"></a>Basisimage Unterschiede

Wie entscheide rechts Basisimage für die Erstellung? Während Sie mit erstellen können Sie eine beliebige, gelten die folgenden allgemeinen Richtlinien für jedes Bild:

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): Wenn die Anwendung im gesamten .NET Framework benötigt, ist dies die beste Bild verwenden.
- [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): für Anwendungen, die nur .NET Core erfordern, Nano Server wird ein viel schmaler Bild bereitstellen.
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): finden Sie unter Umständen Anwendung erforderlich ist eine Komponente oder DLL-Datei, die nicht in der Server Core oder Nano Server-images, z. B. GDI-Bibliotheken. Dieses Image enthält den vollständigen Abhängigkeitseigenschaften Satz von Windows.
- [IoT Core](https://hub.docker.com/_/microsoft-windows-iotcore): Dieses Bild wird speziell für [IoT-Anwendungen](https://developer.microsoft.com/en-us/windows/iot). Wenn auf einem IoT Core-Host, sollten Sie diese Container-Image verwenden.

Für die meisten Benutzer werden Windows Server Core oder Nano Server die am besten geeignete Bild verwenden. Nachfolgend finden Sie einige Dinge zu beachten, wie Sie Informationen zum Erstellen auf Nano Server vorstellen:

- Der Bereitstellungsstapel wurde entfernt.
- .NET Core ist nicht enthalten (Sie können jedoch das [.NET Core Nano Server-Image](https://hub.docker.com/r/microsoft/dotnet/) verwenden).
- PowerShell wurde entfernt.
- WMI wurde entfernt.
- Ab Windows Server der Version 1709 werden Anwendungen unter einem Benutzerkontext ausgeführt, sodass Befehle, die Administratorrechte erfordern, fehlschlagen. Sie können das Container-Administratorkonto über das --user-Flag angeben (Beispiel: docker run --user ContainerAdministrator). In Zukunft werden wir jedoch Administratorkonten vollständig aus NanoServer entfernen.

Dies ist lediglich eine Übersicht über die wichtigsten Unterschiede und keine vollständige Liste. Es gibt weitere Komponenten, die hier nicht genannt sind, die ebenfalls nicht vorhanden sind. Aufbauend auf Nano Server können Sie nach Bedarf jederzeit Ebenen hinzufügen. Ein Beispiel hierfür finden Sie unter [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).

