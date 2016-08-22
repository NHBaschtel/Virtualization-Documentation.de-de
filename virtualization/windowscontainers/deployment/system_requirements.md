---
title: Anforderungen von Windows-Containern
description: Anforderungen von Windows-Containern.
keywords: Metadaten, Container
author: neilpeterson
manager: timlt
ms.date: 08/17/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: fac57150de3ffd6c7d957dd628b937d5c41c1b35
ms.openlocfilehash: f76dc45e6035c72fd7b07f25d4b4c55f2a95aafb

---

# Anforderungen von Windows-Containern

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

In diesen Handbüchern sind die Anforderungen für einen Windows-Containerhost aufgeführt.

## Betriebssystemanforderungen

- Das Feature „Windows-Container“ ist nur für Windows Server 2016 (Core und mit Desktopdarstellung), Nano Server und Windows 10 Professional und Enterprise (Anniversary Edition) verfügbar.
- Wenn Hyper-V-Container ausgeführt werden, muss die Rolle „Hyper-V“ installiert werden.
- Bei Windows Server-Containerhosts muss Windows auf Laufwerk c:\\ installiert werden. Wenn nur Hyper-V-Container bereitgestellt werden, gilt diese Einschränkung nicht.

## Virtualisierte Containerhosts

Wenn ein Windows-Containerhost von einem virtuellen Hyper-V-Computer ausgeführt wird und auch Hyper-V-Container hostet, muss die geschachtelte Virtualisierung aktiviert werden. Für die geschachtelte Virtualisierung ist Folgendes erforderlich:

- Mindestens 4 GB verfügbarer Arbeitsspeicher (RAM) für den virtualisierten Hyper-V-Host
- Windows Server 2016 Technical Preview 5 oder Windows 10 Build 10565 auf dem Hostsystem und Windows Server Technical Preview 5 (Vollversion, Core) oder Nano Server auf dem virtuellen Computer.
- Ein Prozessor mit Intel VT-x (dieses Feature steht zurzeit nur für Intel-Prozessoren zur Verfügung)
- Die Containerhost-VM benötigt zudem mindestens 2 virtuelle Prozessoren.

## Unterstützte Betriebssystemimages

Windows Server Technical Preview 5 wird mit zwei Container-Betriebssystemimages, Windows Server Core und Nano Server angeboten. Nicht alle Konfigurationen unterstützen beide Betriebssystemimages. Diese Tabelle enthält Details zu den unterstützten Konfigurationen.

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
<td><center>Vollständige Benutzeroberfläche für Windows Server 2016</center></td>
<td><center>Server Core-Image</center></td>
<td><center>Nano Server-Image</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Core</center></td>
<td><center>Server Core-Image</center></td>
<td><center> Nano Server-Image</center></td>
</tr>
<tr valign="top">
<td><center>Windows Server 2016 Nano</center></td>
<td><center> Nano Server-Image</center></td>
<td><center>Nano Server-Image</center></td>
</tr>
<tr valign="top">
<td><center>Windows 10 Anniversary Edition</center></td>
<td><center>Nicht verfügbar</center></td>
<td><center>Nano Server-Image</center></td>
</tr>
</tbody>
</table>



<!--HONumber=Aug16_HO3-->


