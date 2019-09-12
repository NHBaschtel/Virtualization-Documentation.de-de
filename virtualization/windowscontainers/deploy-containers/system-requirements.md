---
title: Anforderungen von Windows-Containern
description: Anforderungen von Windows-Containern.
keywords: Metadaten, Container
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: deployment-article
ms.prod: windows-containers
ms.assetid: 3c3d4c69-503d-40e8-973b-ecc4e1f523ed
ms.openlocfilehash: df5d8e17d0d512f7f53fcacf6c2c2a2652f3e7c0
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107864"
---
# <a name="windows-container-requirements"></a>Anforderungen von Windows-Containern

In diesem Leitfaden sind die Anforderungen für einen Windows-Container Host aufgelistet.

## <a name="os-requirements"></a>Betriebssystemanforderungen

- Das Windows-Container Feature steht nur unter Windows Server 2016 (Core und mit Desktop Experience), Windows 10 Professional und Enterprise (Anniversary Edition) und höher zur Verfügung.
- Die Hyper-v-Rolle muss installiert sein, bevor die Hyper-v-Isolierung ausgeführt werden kann.
- Bei Windows Server-Containerhosts muss Windows auf Laufwerk c:\ installiert werden. Diese Einschränkung gilt nicht, wenn nur Hyper-V-isolierte Container bereitgestellt werden.

## <a name="virtualized-container-hosts"></a>Virtualisierte Container Hosts

Wenn ein Windows-Container Host von einem virtuellen Hyper-v-Computer ausgeführt wird und auch die Hyper-v-Isolierung hosten soll, muss die geschachtelte Virtualisierung aktiviert sein. Für die geschachtelte Virtualisierung ist Folgendes erforderlich:

- Mindestens 4GB verfügbarer Arbeitsspeicher (RAM) für den virtualisierten Hyper-V-Host
- Windows Server 2019, Windows Server, Version 1803, Windows Server, Version 1709, Windows Server 2016 oder Windows 10 auf dem Hostsystem und Windows Server (Full, Core) auf dem virtuellen Computer.
- Ein Prozessor mit Intel VT-x (dieses Feature steht zurzeit nur für Intel-Prozessoren zur Verfügung)
- Der Container-Host-VM benötigt auch mindestens zwei virtuelle Prozessoren.

### <a name="memory-requirements"></a>Arbeitsspeicheranforderungen

Einschränkungen des für Container verfügbaren Speichers können über [Ressourcenkontrollen](https://docs.microsoft.com/virtualization/windowscontainers/manage-containers/resource-controls) oder durch Überladen eines Containerhosts konfiguriert werden.  Nachfolgend sind die Mindestanforderungen für den Arbeitsspeicher zum Starten eines Containers und zum Ausführen von grundlegenden Befehlen (ipconfig, dir usw.) aufgeführt.

>[!NOTE]
>Bei diesen Werten wird die Ressourcenfreigabe zwischen Containern oder Anforderungen der im Container ausgeführten Anwendung nicht berücksichtigt.  Beispielsweise kann ein Host mit 512 MB freiem Speicher mehrere Server Core-Container unter Hyper-V-Isolierung ausführen, da diese Container Ressourcen gemeinsam nutzen.

#### <a name="windows-server-2016"></a>Windows Server 2016

| Basis Bild  | Windows Server-Container | Hyper-V-Isolierung    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 40 MB                     | 130 MB + 1 GB Auslagerungsdatei |
| Server Core | 50 MB                     | 325 MB + 1 GB Auslagerungsdatei |

#### <a name="windows-server-version-1709"></a>Windows Server, Version 1709

| Basis Bild  | Windows Server-Container | Hyper-V-Isolierung    |
| ----------- | ------------------------ | -------------------- |
| Nano Server | 30 MB                     | 110 MB + 1 GB Auslagerungsdatei |
| Server Core | 45 MB                     | 360 MB + 1 GB Auslagerungsdatei |
