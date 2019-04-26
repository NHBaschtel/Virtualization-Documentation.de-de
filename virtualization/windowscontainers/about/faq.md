---
title: Windows-Container – häufig gestellte Fragen
description: Windows Server-Container – häufig gestellte Fragen
keywords: Docker, Container
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 69783f0fc3dcc80eb9614031dc6c9b2c35eeefd1
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/26/2019
ms.locfileid: "9577141"
---
# <a name="frequently-asked-questions"></a>Häufig gestellte Fragen

## <a name="general"></a>Allgemein

### <a name="what-is-wcow-what-is-lcow"></a>Was ist WCOW? Was ist LCOW?

WCOW ist eine Abkürzung für Windows-Container unter Windows und LCOW eine Abkürzung für Linux-Container unter Windows.

### <a name="what-is-the-difference-between-linux-and-windows-server-containers"></a>Was ist der Unterschied zwischen Linux- und Windows Server-Containern?

Linux und Windows Server-Container ähneln sich, dass beide ähnliche Technologien in ihrem Kernel und Kernbetriebssystem Betriebssystem implementieren. Der Unterschied ergibt sich aus der Plattform und Arbeitslasten, die innerhalb des Containers ausgeführt.  

Wenn ein Kunde Windows Server-Container verwendet, können mit vorhandenen Windows Technologien wie .NET, ASP.NET, PowerShell und vieles mehr integriert werden.

### <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>Als Entwickler habe ich meine app für jeden Containertyp umschreiben?

Nein. Windows-containerimages sind für Windows Server-Container und Hyper-V-Isolierung. Die Auswahl des Containertyps erfolgt beim Starten des Containers. Aus Sicht der Entwickler Windows Server-Container und Hyper-V-Isolierung sind auf zwei Arten von dasselbe. Sie bieten die gleiche Erfahrung der Entwicklung, Programmierung und Verwaltung, sind öffnen und erweiterbare und das gleiche Maß an Integration und Support mit Docker.

Ein Entwickler kann ein Container-Abbild, das mit einem Windows Server-Container erstellen und in Hyper-V-Isolierung oder umgekehrt unverändert als das Festlegen der entsprechenden Common Language Runtime-Flag bereitstellen.

Windows Server-Containern bietet höhere Dichte und Leistung, wenn Geschwindigkeit Schlüssel, z. B. niedrigere dreht sich Zeit und schnellere Common Language Runtime-Leistung im Vergleich zu geschachtelten Konfigurationen ist. Hyper-V-Isolierung, True, um den Namen bietet mehr Isolation sicherzustellen, dass in einem Container ausgeführte Code kann nicht gefährden oder Auswirkungen auf das Hostbetriebssystem oder andere Container, die auf dem gleichen Host ausgeführt werden. Dies ist hilfreich für mehrinstanzenfähige Szenarien mit Anforderungen für das Hosten von nicht vertrauenswürdigen Codes, einschließlich SaaS-Anwendungen und computehosting.

### <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Was sind die Voraussetzungen für die Ausführung von Containern unter Windows?

Container wurden für die gesamte Plattform mit Windows Server 2016 eingeführt. Um Container zu verwenden, müssen Sie Folgendes, Windows Server 2016 oder Windows 10 Anniversary Update (Version 1607) oder höher.

### <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>Kann ich Windows-Container im Prozess-isolierten Modus auf Windows 10 Enterprise oder Professional ausführen?

Ab Windows 10 October 2018 Update, wir nicht mehr verbieten ein Benutzers von einem Windows-Container mit Prozessisolation ausgeführt. Allerdings müssen Sie direkt beim Anfordern Prozessisolation mithilfe der `--isolation=process` beim Ausführen von der Containers über flag `docker run`.

Wenn dies etwas, die Sie interessiert sind ist, müssen Sie stellen Sie sicher, dass Ihre Hosts Windows 10, Build 17763 + ausgeführt wird, und Sie haben eine Docker-Version mit Modul 18.09 oder höher.

> [!WARNING]
> Dieses Feature ist nur für die Entwicklung oder Tests vorgesehen. Sie sollten weiterhin Windows Server als Host für die Produktion Bereitstellungen verwenden.
>
> Mithilfe dieses Feature zu verwenden, müssen Sie zudem sicherstellen, dass Ihre Host und Container Versionstags übereinstimmen, andernfalls des Containers möglicherweise nicht gestartet oder kann ein nicht definiertes Verhalten aufweisen.

## <a name="windows-container-management"></a>Windows-Containerverwaltung

### <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Wie werden kann ich meine containerimages verfügbar auf Computern Air ausgeführt?

Die Windows-Container-Basisimages enthalten Artefakte, deren Verteilung von Lizenz eingeschränkt sind. Wenn Sie auf diese Bilder erstellen und mit einer privaten oder öffentlichen Registrierung zu übertragen, werden Sie feststellen, ob die Grundebene nie gedrückt ist. In diesem Fall verwenden wir das Konzept einer fremden Ebene der auf echte Grundebene im Azure-Cloud-Speicher remoten verweist.

Dies kann ein Problem darstellen, bei einem Computer Air ausgeführt, der nur Bilder aus der Adresse Ihrer privaten Container-Registrierung ziehen können. Versuche, befolgen die fremde Ebene, um das Basisimage abrufen würde in diesem Fall fehl. Um das fremden Layer-Verhalten zu überschreiben, können Sie die `--allow-nondistributable-artifacts` Flag in der Docker-Daemon.

> [!IMPORTANT]
> Verwendung von dieses Flag entgegen nicht Ihre Verpflichtung zur Einhaltung der Begriffe der Windows-Container Basisimage Lizenz; Sie müssen keine Windows-Inhalt für die öffentliche oder Drittanbieter-Verteilung bereitstellen. Nutzung in Ihrer eigenen Umgebung ist zulässig.

## <a name="microsofts-open-ecosystem"></a>Das offene Ökosystem von Microsoft

### <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>Nimmt Microsoft an der Open Container Initiative (OCI) teil?

Um zu gewährleisten, dass das Paketerstellungsformat universell bleibt, hat Docker vor Kurzem die Open Container Initiative (OCI) ins Leben gerufen, um sicherzustellen, dass die Paketerstellung für Container ein offenes von den Gründern geleitetes Format bleibt, wobei Microsoft eines der Gründungsmitglieder ist.

> [!TIP]
> Sie haben eine Empfehlung für eine Ergänzung zu den häufig gestellten Fragen? Öffnen Sie ein neues Problem Feedback in den Kommentarabschnitt oder verwenden Sie GitHub, um eine Pull-Anforderung gegen diese Dokumente zu öffnen!
