---
title: Anforderungen von Windows-Containern
description: Anforderungen von Windows-Containern.
keywords: Metadaten, Container
author: enderb-ms
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: ecc11468bbd5aad2638da3c4f733e4d5068f0056
ms.sourcegitcommit: 77a6195318732fa16e7d5be727bdb88f52f6db46
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/20/2017
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
<td><center>Nano Server*</center></td>
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
* Ab Version 1709 von Windows Server ist Nano Server nicht mehr als Containerhost verfügbar.

### <a name="memory-requirments"></a>Arbeitsspeicheranforderungen
Einschränkungen des für Container verfügbaren Speichers können über [Ressourcenkontrollen](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls) oder durch Überladen eines Containerhosts konfiguriert werden.  Wieviel Arbeitsspeicher zum Starten eines Containers und zum Ausführen grundlegender Befehle (ipconfig, dir, usw.) mindestens vorhanden sein muss, ist unten aufgeführt.  Bitte beachten Sie, dass diese Werte weder eine gemeinsame Nutzung von Ressourcen durch Container noch die Anforderungen der Anwendung berücksichtigen, die im Container ausgeführt wird.

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

Dies ist lediglich eine Übersicht über die wichtigsten Unterschiede und keine vollständige Liste. Es gibt weitere Komponenten, die hier nicht genannt sind, die ebenfalls nicht vorhanden sind. Aufbauend auf Nano Server können Sie nach Bedarf jederzeit Ebenen hinzufügen. Ein Beispiel hierfür finden Sie unter [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.0/sdk/nanoserver/amd64/Dockerfile).

## <a name="matching-container-host-version-with-container-image-versions"></a>Abgleichen der Containerhostversion und der Containerimageversionen
### <a name="windows-server-containers"></a>Windows Server-Container
Da sich Windows Server-Container und der zugrunde liegende Host einen einzelnen Kernel teilen, muss das Basisimage des Containers mit dem des Hosts übereinstimmen.  Wenn die Versionen nicht übereinstimmen, ist es möglich, dass der Container dennoch startet. Allerdings kann der volle Funktionsumfang nicht gewährleistet werden. Das Windows-Betriebssystem hat vier Versionierungsgrade: die Hauptversion, die Nebenversion, den Build und die Revision (z.B. 10.0.14393.103). Der Buildnummer (d.h. 14393) ändert sich nur, wenn neue Versionen des Betriebssystems, z.B. Version 1709, 1803 veröffentlicht werden, Fall Creators Update usw.... Die Revisionsnummer (d.h. 103) wird aktualisiert, wenn Windows-Updates angewendet werden.
#### <a name="build-number-new-release-of-windows"></a>Buildnummer (neue Version von Windows)
Der Start eines Windows Server-Containers wird verhindert, wenn die Buildnummer zwischen dem Containerhost und dem Containerimage unterschiedlich ist – zum Beispiel 10.0.14393.* (Windows Server2016) und 10.0.16299.* (Windows Server Version 1709).  
#### <a name="revision-number-patching"></a>Revisionsnummer (Patches)
Der Start eines Windows Server-Containers wird _nicht_ verhindert, wenn die Revisionsnummer zwischen dem Containerhost und dem Containerimage unterschiedlich ist – zum Beispiel 10.0.14393.1914 (Windows Server2016 mit KB4051033) und 10.0.14393.1944 (Windows Server 2016 mit KB4053579).  
Für auf Windows Server2016 basierte Hosts-Images muss die Revisionsnummer des Containerimages dem Host entsprechen, um einer unterstützten Konfiguration zu entsprechen.  Ab Windows Server Version 1709 ist dies ist nicht mehr gültig, und die Revisionsnummern von Host und Containerimage müssen nicht mehr identisch sein.  Es empfiehlt sich immer, Ihre Systeme mit den neuesten Patches und Updates auf dem neusten Stand zu halten.
#### <a name="practical-application"></a>Praktische Anwendung
Beispiel 1: Der Container-Host führt Windows Server2016 mit KB4041691 aus.  Alle Windows Server-Container, die auf diesem Host bereitgestellt werden, müssen auf dem Basisimages für Container 10.0.14393.1770 basieren.  Wenn KB4053579 auf dem Host angewendet wird, müssen die Containerimages zur gleichen Zeit aktualisiert werden, um Support zu erhalten.
Beispiel 2: Der Container-Host führt Windows Server Version1709 mit KB4043961 aus.  Alle auf diesem Host bereitgestellten Windows Server-Container müssen auf einem Containerbasis-Image von Windows Server-Version 1709 (10.0.16299) basieren, müssen allerdings nicht mit dem Host KB übereinstimmen.  Wenn KB4054517 auf dem Host angewendet wird, müssen die Containerimages nicht aktualisiert werden. Sie sollten jedoch auf dem neusten Stand sein, damit alle Sicherheitsprobleme vollständig behoben werden.
#### <a name="querying-version"></a>Abfrageversion
Methode 1: Die in Version eingeführte 1709 Cmd-Eingabeaufforderung und der „ver”-Befehl geben jetzt die Revisionsdetails zurück.
```
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125] 
```
Methode 2: Erstellen Sie eine Abfrage für den folgenden Registrierungsschlüssel: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\AktuelleVersion Zum Beispiel:
```
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```
Oder
```
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

Prüfen Sie die Tags auf Docker Hub oder die Image-Hash-Tabelle in der Beschreibung des Images, um zu überprüfen, welche Version Ihr Basisimage verwendet.  Auf der Seite [Windows 10-Updateverlauf](https://support.microsoft.com/en-us/help/12387/windows-10-update-history) wird aufgeführt, wann die einzelnen Builds und Revisionen veröffentlicht wurden.

### <a name="hyper-v-isolation-for-containers"></a>Hyper-V-Isolation für Container
Windows-Container können mit oder ohne Hyper-V-Isolation ausgeführt werden.  Hyper-V-Isolation erstellt mithilfe eines optimierten VMs eine Sicherheitsbegrenzung um den Container herum.  Im Gegensatz zu Windows Standard-Containern, bei denen sich die Container und der Host den Kernel teilen, verwendet jeder isolierte Hyper-V-Container eine eigene Instanz des Windows-Kernels.  Aus diesem Grund können Sie verschiedene Betriebssystemversionen im Container-Host und -Image ausführen (siehe Kompatibilitätsmatrix unten).  

Um einen Container mit Hyper-V auszuführen, fügen Sie einfach das Tag „--isolation=hyper-v” zu Ihrem Docker-Ausführungsbefehl hinzu.

### <a name="compatibility-matrix"></a>Kompatibilitätsmatrix
Windows Server-Builds nach 2016 GA (10.0.14393.206) können die Windows Server 2016 GA-Images von Windows Server Core oder Nano Server unabhängig von der Revisionsnummer in einer unterstützten Konfiguration ausführen.
Ein Host auf Windows-Server, Version 1709 kann ebenfalls auf Windows Server2016 basierte Container ausführen, das Gegenteil wird jedoch nicht unterstützt.

Es ist wichtig, sich bewusst zu machen, dass die von den Windows-Updates bereitgestellte vollständige Funktionalität und Zuverlässigkeit sowie die umfassenden Sicherheitsgarantien nur gewährleistet sind, wenn Sie auf allen Systemen die neuesten Versionen installiert haben.  
