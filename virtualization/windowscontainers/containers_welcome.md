---
title: Dokumentation zu Windows-Containern
description: Dokumentation zu Windows-Containern
keywords: Docker, Container
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 74c9d604-0915-4d89-bc69-0263b76bc66b
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 2c9821ef7ac414640790b3cfdb7fd457710a67f4

---

# Dokumentation zu Windows-Containern

Windows-Container ermöglichen die Virtualisierung auf Betriebssystemebene und damit die Ausführung mehrerer isolierter Anwendungen auf einem einzigen System. Das Feature stellt zwei verschiedene Arten von Containerlaufzeiten mit zwei unterschiedlichen Anwendungsisolierungsgraden bereit. Windows Server-Container erreichen die Isolierung durch Namespace- und Prozessisolierung . Hyper-V-Container kapseln jeden Container in einer Lightweight-VM. Dieser Dokumentationssatz enthält Schnellstartanleitungen und Bereitstellungshandbücher sowie technische Details zu Verwaltungsvorgängen.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:90%" cellpadding="25" cellspacing="5">
<tr>
<td ><center>![](media/try.png)</center></td>
<td>**Schnellstart**<br /><br />
Testen Sie Windows Server- und Hyper-V-Container mithilfe der folgenden Schnellstartanleitungen.<br /><br />
<ul>
<li>[1 – Konzepte und Terminologie](quick_start/quick_start.md)<br /><br /></li>
<li>[2 – Container unter Windows Server](quick_start/quick_start_windows_server.md)<br /><br /></li>
<li>[3 – Containerimages unter Windows Server](quick_start/quick_start_images.md)<br /><br /></li>
<li>[4 – Container unter Windows 10](quick_start/quick_start_windows_10.md)<br /><br /></li>
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



<!--HONumber=Sep16_HO2-->


