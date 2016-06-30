---
title: Anforderungen von Windows-Containern
description: Anforderungen von Windows-Containern.
keywords: metadata, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
translationtype: Human Translation
ms.sourcegitcommit: cc216f56acd5e547d05a48beea57450ba5fcb28b
ms.openlocfilehash: 12ae565f012dc87a2cab883c0486322db42b1dcc

---

# Anforderungen von Windows-Containern

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

In diesen Handbüchern sind die Anforderungen für einen Windows-Containerhost aufgeführt.

## Betriebssystemanforderungen

- Die Rolle „Windows-Container“ ist nur unter Windows Server 2016 TP5 (Vollversion und Core) und Nano Server sowie Windows 10 (Insiders-Build 14352 und höher) verfügbar.
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
<td><center>Windows 10 Insider-Versionen</center></td>
<td><center>Nicht verfügbar</center></td>
<td><center>Nano Server-Image</center></td>
</tr>
</tbody>
</table>



<!--HONumber=Jun16_HO4-->


