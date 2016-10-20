---
title: Anforderungen von Windows-Containern
description: Anforderungen von Windows-Containern.
keywords: Metadaten, Container
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: d4c453e800d4057b3ad0be06c28e7f23b81443f0
ms.openlocfilehash: 008eff4731a8835b0b3f664edc9955f85c12a629

---

# Anforderungen von Windows-Containern

In diesen Handbüchern sind die Anforderungen für einen Windows-Containerhost aufgeführt.

## Betriebssystemanforderungen

- Das Feature „Windows-Container“ ist nur für Windows Server 2016 (Core und mit Desktopdarstellung), Nano Server und Windows 10 Professional und Enterprise (Anniversary Edition) verfügbar.
- Die Hyper-V-Rolle muss vor der Ausführung des Hyper-V-Containers installiert werden.
- Bei Windows Server-Containerhosts muss Windows auf der Partition „C:\.“ installiert seinDiese Einschränkung entfällt, wenn nur Hyper-V-Container bereitgestellt werden.

## Virtualisierte Containerhosts

Wenn ein Windows-Containerhost von einem virtuellen Hyper-V-Computer ausgeführt wird und auch Hyper-V-Container hostet, muss die geschachtelte Virtualisierung aktiviert werden. Für die geschachtelte Virtualisierung ist Folgendes erforderlich:

- Mindestens 4 GB verfügbarer Arbeitsspeicher (RAM) für den virtualisierten Hyper-V-Host
- Windows Server 2016 oder Windows 10 auf dem Hostsystem und Windows Server (Vollständig, Core) oder Nano Server auf dem virtuellen Computer
- Ein Prozessor mit Intel VT-x (dieses Feature steht zurzeit nur für Intel-Prozessoren zur Verfügung)
- Die Containerhost-VM benötigt zudem mindestens 2 virtuelle Prozessoren.

## Unterstützte Basisimages

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
<td><center>Windows Server 2016 mit Desktop</center></td>
<td><center>Server Core/Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Server Core/Nano Server</center></td>
<td><center>Server Core/Nano Server</center></td>
</tr>
<tr valign="top">
<td><center>Nano Server</center></td>
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

## Abgleichen der Containerhostversion und der Containerimageversionen
### Windows Server-Container
Da sich Windows Server-Container und der zugrunde liegende Host einen einzelnen Kernel teilen, muss das Basisimage des Containers mit dem des Hosts übereinstimmen.  Wenn die Versionen nicht übereinstimmen, ist es möglich, dass der Container dennoch startet. Allerdings kann der volle Funktionsumfang nicht gewährleistet werden. Nicht übereinstimmende Versionen werden daher nicht unterstützt.  Das Windows-Betriebssystem hat vier Versionierungsgrade: die Hauptversion, die Nebenversion, den Build und die Revision (z.B. 10.0.14393.0). Die Buildnummer wird nur geändert, wenn neue Versionen des Betriebssystems veröffentlicht werden. Die Revisionsnummer wird aktualisiert, wenn Windows-Updates angewendet werden. Das Starten von Windows Server-Containern wird verhindert, wenn die Build-Nummer nicht übereinstimmt, z.B. 10.0.14300.1030 (Technical Preview 5) und 10.0.14393 (Windows Server 2016 RTM). Wenn die Buildnummer übereinstimmt, sich die Revisionsnummer aber unterscheidet, z.B. 10.0.14393 (Windows Server 2016 RTM) und 10.0.14393.206 (Windows Server 2016 GA), wird der Container trotzdem gestartet. Obwohl sie nicht technisch blockiert werden, ist dies eine Konfiguration, die möglicherweise nicht unter allen Umständen ordnungsgemäß funktioniert und daher nicht für Produktionsumgebungen unterstützt wird. 

Mithilfe von HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion können Sie abfragen, welche Version auf einem Windows-Host installiert ist.  Prüfen Sie die Tags auf Docker Hub oder die Image-Hash-Tabelle in der Beschreibung des Images, um zu überprüfen, welche Version Ihr Basisimage verwendet.  Auf der Seite [Windows 10-Updateverlauf](https://support.microsoft.com/en-us/help/12387/windows-10-update-history) wird aufgeführt, wann die einzelnen Builds und Revisionen veröffentlicht wurden.

In diesem Beispiel ist 14393 die Hauptbuildnummer und 321 die Revision.
```none
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

### Hyper-V-Container
Im Gegensatz zu Windows Server-Containern, bei denen sich die Container und der Host den Kernel teilen, verwendet jeder Hyper-V-Container eine eigene Instanz des Windows-Kernels.  Aus diesem Grund müssen die Versionen von Containerhost und Containerimage nicht übereinstimmen.  Aktuell können Builds mit einer Nummer größer gleich Windows Server 2016 GA (10.0.14393.206) die Windows Server 2016 GA-Images von Windows Server Core oder Nano Server unabhängig von der Revisionsnummer in einer unterstützten Konfiguration ausführen.  In Zukunft werden wir aufgrund des Kundenfeedbacks spezifische Informationen dazu anbieten, wie weit Buildnummern auseinanderliegen können.  Es ist wichtig, sich bewusst zu machen, dass die von den Windows-Updates bereitgestellte vollständige Funktionalität und Zuverlässigkeit sowie die umfassenden Sicherheitsgarantien nur gewährleistet sind, wenn Sie auf allen Systemen die neuesten Versionen installiert haben.  


<!--HONumber=Oct16_HO2-->


