---
title: Anforderungen von Windows-Containern
description: Anforderungen von Windows-Containern.
keywords: Metadaten, Container
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 4706ea24da1d5ca61b94dfd141883aa2d04ad906
ms.sourcegitcommit: 7c3af076eb8bad98e1c3de0af63dacd842efcfa3
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/07/2018
ms.locfileid: "1844049"
---
# <a name="windows-container-requirements"></a>Anforderungen von Windows-Containern

In diesen Handbüchern sind die Anforderungen für einen Windows-Containerhost aufgeführt.

## <a name="os-requirements"></a>Betriebssystemanforderungen

- Das Feature "Windows-Container" ist nur für Windows Server Build 1709, Windows Server 2016 (Core und mit Desktopdarstellung) und Windows 10 Professional und Enterprise (Anniversary Edition) verfügbar.
- Die Hyper-V-Rolle muss vor der Ausführung des Hyper-V-Containers installiert werden.
- Bei Windows Server-Containerhosts muss Windows auf Laufwerk c:\ installiert werden. Diese Einschränkung gilt nicht, wenn nur Hyper-V Container bereitgestellt werden.

## <a name="virtualized-container-hosts"></a>Virtualisierte Containerhosts

Wenn ein Windows-Containerhost von einem virtuellen Hyper-V-Computer ausgeführt wird und auch Hyper-V-Container hostet, muss die geschachtelte Virtualisierung aktiviert werden. Für die geschachtelte Virtualisierung ist Folgendes erforderlich:

- Mindestens 4GB verfügbarer Arbeitsspeicher (RAM) für den virtualisierten Hyper-V-Host
- Windows Server Build 1709, Windows Server 2016 oder Windows 10 auf dem Hostsystem und Windows Server (Vollständig, Core) auf dem virtuellen Computer
- Ein Prozessor mit Intel VT-x (dieses Feature steht zurzeit nur für Intel-Prozessoren zur Verfügung)
- Die Containerhost-VM benötigt zudem mindestens 2 virtuelle Prozessoren.

## <a name="supported-base-images"></a>Unterstützte Basisimages

Für Windows-Container stehen zwei Basisimages zur Verfügung – Windows Server Core und Nano Server. Nicht alle Konfigurationen unterstützen beide Betriebssystemimages. Diese Tabelle enthält Details zu den unterstützten Konfigurationen.

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
<td><center>Windows Server 2016 (Standard oder Datacenter)</center></td>
<td><center>Server Core/Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server<a href="#warn-1">*</a></center></td>
<td><center> Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Pro/Enterprise</center></td>
<td><center>Nicht verfügbar</center></td>
<td><center>Server Core/Nano Server</center></td>
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


### <a name="nano-server-vs-windows-server-core"></a>Nano Server oder Windows Server Core?

Wie entscheide ich mich zwischen Windows Server Core und Nano Server? Zum Erstellen steht es Ihnen frei, was Sie verwenden. Wenn Sie für Ihre Anwendung jedoch vollständige Kompatibilität mit dem .NET Framework benötigen, dann verwenden Sie [Windows Server Core](https://hub.docker.com/r/microsoft/windowsservercore/). Wenn Ihre Anwendung hingegen für die Cloud erstellt wurde und .NET Core verwendet, dann ist [Nano Server](https://hub.docker.com/r/microsoft/nanoserver/) die bessere Wahl. Nano Server wurde für einen geringstmöglichen Speicherbedarf erstellt, weshalb einige nicht erforderliche Bibliotheken entfernt wurden. Bedenken Sie Folgendes, wenn Sie Nano Server als Grundlage zum Erstellen verwenden:

- Der Bereitstellungsstapel wurde entfernt.
- .NET Core ist nicht enthalten (Sie können jedoch das [.NET Core Nano Server-Image](https://hub.docker.com/r/microsoft/dotnet/) verwenden).
- PowerShell wurde entfernt.
- WMI wurde entfernt.
- Ab Windows Server der Version 1709 werden Anwendungen unter einem Benutzerkontext ausgeführt, sodass Befehle, die Administratorrechte erfordern, fehlschlagen. Sie können das Container-Administratorkonto über das --user-Flag angeben (Beispiel: docker run --user ContainerAdministrator). In Zukunft werden wir jedoch Administratorkonten vollständig aus NanoServer entfernen.

Dies ist lediglich eine Übersicht über die wichtigsten Unterschiede und keine vollständige Liste. Es gibt weitere Komponenten, die hier nicht genannt sind, die ebenfalls nicht vorhanden sind. Aufbauend auf Nano Server können Sie nach Bedarf jederzeit Ebenen hinzufügen. Ein Beispiel hierfür finden Sie unter [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile).

