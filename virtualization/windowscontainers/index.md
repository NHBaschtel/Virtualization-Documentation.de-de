---
title: Dokumentation zu Windows-Containern
description: Dokumentation zu Windows-Containern
keywords: Docker, Container
author: enderb-ms
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 74c9d604-0915-4d89-bc69-0263b76bc66b
ms.openlocfilehash: ac6e99800fcabef31464a81799fc9e329438b0ae
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: de-DE
---
# <a name="windows-containers-documentation"></a>Dokumentation zu Windows-Containern

Windows-Container ermöglichen die Virtualisierung auf Betriebssystemebene und damit die Ausführung mehrerer isolierter Anwendungen auf einem einzigen System. Das Feature stellt zwei verschiedene Arten von Containerlaufzeiten mit zwei unterschiedlichen Anwendungsisolierungsgraden bereit. Windows Server-Container erreichen die Isolierung durch Namespace- und Prozessisolierung . Hyper-V-Container kapseln jeden Container in einer Lightweight-VM. Dieser Dokumentationssatz enthält Schnellstartanleitungen und Bereitstellungshandbücher sowie technische Details zu Verwaltungsvorgängen.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:90%" cellpadding="25" cellspacing="5">
<tr>
<td ><center>![](media/try.png)</center></td>
<td>**Schnellstart**<br /><br />
Schnellstartanleitung für Windows Server<br /><br />
<ul>
<li>[Schritt 1 – Konzepte und Terminologie](quick-start/index.md)<br /><br /></li>
<li>[Schritt 2 – Konfigurieren von Windows Server und des ersten Containers](quick-start/quick-start-windows-server.md)<br /><br /></li>
<li>[Schritt 3 – Containerimages erstellen und mithilfe von Push übertragen](quick-start/quick-start-images.md)<br /><br /></li>
</ul>
Schnellstartanleitung für Windows 10<br /><br />
<ul>
<li>[Schritt 1 – Konzepte und Terminologie](quick-start/index.md)<br /><br /></li>
<li>[Schritt 2 – Konfigurieren von Windows 10 und erster Container](quick-start/quick-start-windows-10.md)<br /><br /></li>
</ul>
</td>
</tr>
<tr>
<td ><center>![](media/1.png)</center></td>
<td>**Bereitstellung**<br /><br />
Hier erfahren Sie, wie Sie Windows-Container unter Windows Server2016 und Nano Server bereitstellen.<br /><br />
<ul>
<li>[Systemanforderungen](deploy-containers/system-requirements.md)<br /><br /></li>
<li>[Bereitstellen eines Containerhosts – Windows Server](deploy-containers/deploy-containers-on-server.md)<br /><br /></li>
<li>[Bereitstellen eines Containerhosts – Nano Server](deploy-containers/deploy-containers-on-nano.md)<br /><br /></li>
<li>[Antivirusoptimierung](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)<br /><br /></li>
</ul>
</td>
</tr>

<tr>
<td ><center>![](media/explore.png)</center></td>
<td>**Docker unter Windows**<br /><br />
Hier finden Sie Informationen Sie zum Verwalten von Docker unter Windows.<br /><br />
<ul>
<li>[Docker-Modul unter Windows](manage-docker/configure-docker-daemon.md)<br /><br /></li>
<li>[Dockerfiles unter Windows](manage-docker/manage-windows-dockerfile.md)<br /><br /></li>
<li>[Optimieren von Dockerfiles](manage-docker/optimize-windows-dockerfile.md)<br /><br /></li>
<li>[Containernetzwerk](manage-containers/container-networking.md)<br /><br /></li>
</ul>
</td>
</tr>

<tr>
<td ><center>![](media/video.png)</center></td>
<td>**Überwachen**<br /><br />
Sie sind an Demos und Interviews mit dem Windows-Container-Team interessiert?<br /><br />
<ul>
<li>[Kanal zu Containern](https://channel9.msdn.com/Blogs/containers)</li>
</ul>
<br />
</td>
</tr>

<tr>
<td ><center>![](media/question.png)</center></td>
<td>**Community**<br /><br />
Hier können Sie mit der Community interagieren, Beispiele testen und weitere Ressourcen finden.<br /><br />
<ul>
<li>[Containerforum](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)<br /><br /></li>
<li>[Videos und Blogs der Community](communitylinks.md)<br /><br /></li>
<li>[Containerressourcen](https://msdn.microsoft.com/virtualization/community/community_overview)<br /><br /></li>
</ul>
</td>
</tr>
</table>
