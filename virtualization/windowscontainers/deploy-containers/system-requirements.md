---
title: Anforderungen von Windows-Containern
description: Anforderungen von Windows-Containern.
keywords: Metadaten, Container
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: 942676be30760cbe1701d75f7d2fbca9539ce03b
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/26/2019
ms.locfileid: "9577061"
---
# <a name="windows-container-requirements"></a>Anforderungen von Windows-Containern

Dieses Handbuch beschreibt die Anforderungen für einen Windows-containerhost aufgeführt.

## <a name="os-requirements"></a>Betriebssystemanforderungen

- Das Feature "Windows-Container" ist nur verfügbar auf Windows Server 2016 (Core und mit Desktopdarstellung), Windows 10 Professional und Enterprise (Anniversary Edition) und höher.
- Hyper-V-Rolle muss installiert werden, vor dem Ausführen von Hyper-V-Isolierung
- Bei Windows Server-Containerhosts muss Windows auf Laufwerk c:\ installiert werden. Diese Einschränkung gilt nicht, wenn nur Hyper-V isolierte Container bereitgestellt werden.

## <a name="virtualized-container-hosts"></a>Virtualisierte containerhosts

Wenn ein Windows-containerhost von einem virtuellen Hyper-V-Computer ausgeführt wird und wird auch Hyper-V-Isolierung hostet, müssen die geschachtelte Virtualisierung aktiviert werden. Für die geschachtelte Virtualisierung ist Folgendes erforderlich:

- Mindestens 4GB verfügbarer Arbeitsspeicher (RAM) für den virtualisierten Hyper-V-Host
- Windows Server 2019, Windows Server Version 1803, Windows Server Version 1709, Windows Server 2016 oder Windows 10 auf dem Hostsystem und Windows Server (vollständig, Core) auf dem virtuellen Computer.
- Ein Prozessor mit Intel VT-x (dieses Feature steht zurzeit nur für Intel-Prozessoren zur Verfügung)
- Die containerhost-VM benötigt für mindestens zwei virtuelle Prozessoren.

## <a name="supported-base-images"></a>Unterstützte Basisimages

Windows-Container werden vier Container-Basisimages angeboten: Windows Server Core, Nano Server, Windows und IoT Core. Nicht alle Konfigurationen unterstützen beide Betriebssystemimages. Diese Tabelle enthält Details zu den unterstützten Konfigurationen.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Hostbetriebssystem</center></th>
<th><center>Windows Server-container</center></th>
<th><center>Hyper-V-Isolierung</center></th>
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

> [!WARNING]  
> Ab Windows Server Version 1709, ist Nano Server nicht mehr als containerhost verfügbar.

### <a name="memory-requirements"></a>Arbeitsspeicheranforderungen

Einschränkungen des für Container verfügbaren Speichers können über [Ressourcenkontrollen](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls) oder durch Überladen eines Containerhosts konfiguriert werden.  Die Mindestmenge an wieviel Arbeitsspeicher zum Starten eines Containers und zum Ausführen grundlegender Befehle (Ipconfig, Dir, usw.) sind unten aufgeführt.

>[!NOTE]
>Diese Werte berücksichtigen nicht von Ressourcen durch Container noch die Anforderungen der Anwendung, die im Container ausgeführte.  Beispielsweise kann ein Host mit 512 MB freiem Speicher mehrere Server Core-Container unter Hyper-V-Isolierung ausführen, da diese Container Ressourcen gemeinsam nutzen.

#### <a name="windows-server-2016"></a>Windows Server 2016

| Basis-image  | Windows Server-container | Hyper-V-Isolierung    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB Auslagerungsdatei |
| Server Core | 50 MB                     | 325 MB + 1 GB Auslagerungsdatei |

#### <a name="windows-server-version-1709"></a>Windows Server, Version 1709

| Basis-image  | Windows Server-container | Hyper-V-Isolierung    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB Auslagerungsdatei |
| Server Core | 45 MB                     | 360 MB + 1 GB Auslagerungsdatei |

### <a name="base-image-differences"></a>Basis-Image Unterschiede

Wie entscheide rechts Basisimage für die Erstellung? Während Sie mit erstellen können Sie eine beliebige, gelten die folgenden allgemeinen Richtlinien für jedes Bild:

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): Wenn die Anwendung im gesamten .NET Framework benötigt, ist dies die beste Bild verwenden.
- [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): für Anwendungen, die nur .NET Core erfordern Nano Server wird ein viel schmaler Bild bereitstellen.
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): finden Sie unter Umständen Anwendung erforderlich ist eine Komponente oder ein DLL, die nicht in der Server Core oder Nano Server-images, z. B. GDI-Bibliotheken. Dieses Bild führt den vollständigen Abhängigkeitseigenschaften Satz von Windows.
- [IoT Core](https://hub.docker.com/_/microsoft-windows-iotcore): Dieses Bild wird speziell für [IoT-Anwendungen](https://developer.microsoft.com/en-us/windows/iot). Sie sollten diese Container-Image verwenden, wenn auf einem IoT Core-Host.

Für die meisten Benutzer werden Windows Server Core oder Nano Server die am besten geeignete Bild verwenden. Die folgenden Dinge bedenken, wie Sie Informationen zum Erstellen auf Nano Server vorstellen:

- Der Bereitstellungsstapel wurde entfernt.
- .NET Core ist nicht enthalten (Sie können jedoch das [.NET Core Nano Server-Image](https://hub.docker.com/r/microsoft/dotnet/) verwenden).
- PowerShell wurde entfernt.
- WMI wurde entfernt.
- Ab Windows Server der Version 1709 werden Anwendungen unter einem Benutzerkontext ausgeführt, sodass Befehle, die Administratorrechte erfordern, fehlschlagen. Sie können angeben, die Container-Administratorkonto über das--User-Flag (z. B. Docker run Benutzer ContainerAdministrator) jedoch in der Zukunft wir Administratorkonten vollständig aus NanoServer entfernen möchten.

Dies ist lediglich eine Übersicht über die wichtigsten Unterschiede und keine vollständige Liste. Es gibt weitere Komponenten, die hier nicht genannt sind, die ebenfalls nicht vorhanden sind. Aufbauend auf Nano Server können Sie nach Bedarf jederzeit Ebenen hinzufügen. Ein Beispiel hierfür finden Sie unter [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).