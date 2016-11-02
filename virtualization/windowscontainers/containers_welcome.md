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
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 144532bf835f8f8a67d378ec03e707b9517d5653

---

# Dokumentation zu Windows-Containern

Windows-Container ermöglichen die Virtualisierung auf Betriebssystemebene und damit die Ausführung mehrerer isolierter Anwendungen auf einem einzigen System. Das Feature stellt zwei verschiedene Arten von Containerlaufzeiten mit zwei unterschiedlichen Anwendungsisolierungsgraden bereit. Windows Server-Container erreichen die Isolierung durch Namespace- und Prozessisolierung . Hyper-V-Container kapseln jeden Container in einer Lightweight-VM. Dieser Dokumentationssatz enthält Schnellstartanleitungen und Bereitstellungshandbücher sowie technische Details zu Verwaltungsvorgängen.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:90%" cellpadding="25" cellspacing="5">
<tr>
<td ><center>![](media/try.png)</center></td>
<td>**Schnellstart**<br /><br />
Schnellstartanleitung für Windows Server<br /><br />
<ul>
<li>[Schritt 1 – Konzepte und Terminologie](quick_start/quick_start.md)<br /><br /></li>
<li>[Schritt 2 – Konfigurieren von Windows Server und des ersten Containers](quick_start/quick_start_windows_server.md)<br /><br /></li>
<li>[Schritt 3 – Containerimages erstellen und mithilfe von Push übertragen](quick_start/quick_start_images.md)<br /><br /></li>
</ul>
Schnellstartanleitung für Windows 10<br /><br />
<ul>
<li>[Schritt 1 – Konzepte und Terminologie](quick_start/quick_start.md)<br /><br /></li>
<li>[Schritt 2 – Konfigurieren von Windows 10 und erster Container](quick_start/quick_start_windows_10.md)<br /><br /></li>
</ul>
</td>
</tr>
<tr>
<td ><center>![](media/1.png)</center></td>
<td>**Bereitstellung**<br /><br />
Hier erfahren Sie, wie Sie Windows-Container unter Windows Server 2016 und Nano Server bereitstellen.<br /><br />
<ul>
<li>[Systemanforderungen](deployment/system_requirements.md)<br /><br /></li>
<li>[Bereitstellen eines Containerhosts – Windows Server](deployment/deployment.md)<br /><br /></li>
<li>[Bereitstellen eines Containerhosts – Nano Server](deployment/deployment_nano.md)<br /><br /></li>
<li>[Antivirusoptimierung](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)<br /><br /></li>
</ul>
</td>
</tr>

<tr>
<td ><center>![](media/explore.png)</center></td>
<td>**Docker unter Windows**<br /><br />
Hier finden Sie Informationen Sie zum Verwalten von Docker unter Windows.<br /><br />
<ul>
<li>[Docker-Modul unter Windows](docker/configure_docker_daemon.md)<br /><br /></li>
<li>[Dockerfiles unter Windows](docker/manage_windows_dockerfile.md)<br /><br /></li>
<li>[Verwalten von Containerdaten](management/manage_data.md)<br /><br /></li>
<li>[Optimieren von Dockerfiles](docker/optimize_windows_dockerfile.md)<br /><br /></li>
<li>[Containernetzwerk](management/container_networking.md)<br /><br /></li>
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
<li>[Containerressourcen](https://msdn.microsoft.com/virtualization/community/community_overview)<br /><br /></li>
</ul>
</td>
</tr>
</table>



<!--HONumber=Oct16_HO4-->


