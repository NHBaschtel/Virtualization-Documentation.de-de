---
title: Windows-Container – häufig gestellte Fragen
description: Windows-Container – häufig gestellte Fragen
keywords: Docker, Container
author: PatrickLang
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 36ef6df0b9736d88fec627e4cb56df023f1a7708
ms.sourcegitcommit: 4336d7617c30d26a987ad3450b048e17404c365d
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 01/11/2019
ms.locfileid: "9001009"
---
# <a name="frequently-asked-questions"></a>Häufig gestellte Fragen

## <a name="general"></a>Allgemein

### <a name="what-is-wcow-what-is-lcow"></a>Was ist WCOW? Was ist LCOW?

WCOW ist eine Abkürzung für Windows-Container unter Windows und LCOW eine Abkürzung für Linux-Container unter Windows.

### <a name="what-is-the-difference-between-linux-and-windows-server-containers"></a>Was ist der Unterschied zwischen Linux- und Windows Server-Containern?

Linux- und Windows Server-Container ähneln sich, denn beide implementieren ähnliche Technologien in ihrem Kernel und Kernbetriebssystem. Der Unterschied ergibt sich aus der Plattform und Arbeitslasten, die innerhalb des Containers ausgeführt.  
Wenn ein Kunde Windows Server-Container verwendet, können vorhandene Windows-Technologien wie .NET, ASP.NET, PowerShell und vieles mehr integriert werden.

### <a name="as-a-developer-do-i-have-to-re-write-my-app-for-each-type-of-container"></a>Muss ich als Entwickler meine App für jeden Containertyp umschreiben?

Nein. Windows-Containerimages sind für Windows Server-Container und Hyper-V-Container gleich. Die Auswahl des Containertyps erfolgt beim Starten des Containers. Aus Sicht der Entwickler sind Windows Server-Containern und Hyper-V zwei Arten von dasselbe. Diese Entwicklung, Programmierung und Verwaltung genauso bieten, werden offene und erweiterbare und das gleiche Maß an Integration und Support über Docker enthält.

Ein Entwickler kann ein Container-Abbild, das mit einem Windows Server-Container erstellen und als Hyper-V-Container oder umgekehrt unverändert als das Festlegen der entsprechenden Common Language Runtime-Flag bereitstellen.

Windows Server-Containern bietet höhere Dichte und Leistung (z. B. niedrigere dreht sich Zeit, schnellere Common Language Runtime-Leistung im Vergleich zu geschachtelten Konfigurationen) Wenn Geschwindigkeit Schlüssel ist. Hyper-V-Container bieten mehr Isolation sicherzustellen, dass in einem Container ausgeführte Code kann nicht gefährden oder Auswirkungen auf das Hostbetriebssystem oder andere Container, die auf dem gleichen Host ausgeführt wird. Dies ist hilfreich für mehrinstanzenfähige Szenarien (mit Anforderungen für das Hosten von nicht vertrauenswürdigem Code), einschließlich SaaS-Anwendungen und Computehosting.

### <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Was sind die Voraussetzungen für die Ausführung von Containern unter Windows?

Container wurden auf die Plattform ab Windows Server 2016 eingeführt. Sie müssen ausführen, Windows Server 2016 oder Windows 10 Anniversary Update (Version 1607) oder höher Container verwendet werden.

### <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10-enterprise-or-professional"></a>Kann ich Windows-Container im Prozess isoliert-Modus unter Windows 10 Enterprise oder Professional ausführen?

Ab Windows 10 Oktober 2018 Update, wir nicht mehr verbieten ein Benutzers vom Prozessisolation mit einem Windows-Container. Sie müssen direkt Prozessisolation anfordern, indem die `--isolation=process` beim Ausführen von der Containers, über das flag `docker run`.

Ist dies eine Aktion, die Sie interessiert sind, müssen Sie stellen Sie sicher, dass der Host Windows 10, Build 17763 + ausgeführt wird, und Sie haben eine Docker-Version mit Modul 18.09 oder höher.

> [!WARNING]
> Dieses Feature ist nur für die Entwicklung oder Tests vorgesehen. Sie sollten weiterhin Windows Server als Host für die Produktion Bereitstellungen verwenden.
>
> Verwenden Sie dieses Feature, müssen Sie zudem sicherstellen, dass der Host und Container Versionstags stimmen, andernfalls Container möglicherweise nicht gestartet oder kann nicht definierten Verhalten aufweisen.

## <a name="windows-container-management"></a>Windows-Containerverwaltung

### <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Wie werden kann ich meine containerimages verfügbar auf Computern Air ausgeführt?

Die Windows-Container-Basisimages enthalten Artefakte, deren Verteilung von Lizenz eingeschränkt sind. Wenn Sie auf diese Bilder zu erstellen und mit einer privaten oder öffentlichen Registrierung zu übertragen, werden Sie feststellen, dass die Basisebene niemals per Push übertragen werden. In diesem Fall verwenden wir das Konzept einer fremden Ebene der auf die tatsächlichen Basisebene, die in Azure-Cloud-Speicher verweist.

Dies kann ein Problem darstellen, wenn Sie einen Computer mit Air ausgeführt, die _nur_ Pull Bilder aus der Adresse der _Container des privaten Registrierung_ können haben. In diesem Fall schlägt fehl, Versuche, folgen den fremde Ebene, um das Basisimage abrufen. Um das fremden Ebene Verhalten zu überschreiben, können Sie die `--allow-nondistributable-artifacts` -Flag in der Docker-Daemon.

> [!IMPORTANT]
> Verwendung von dieses Flag entgegen nicht Ihre Verpflichtung zur Einhaltung der Bestimmungen des Windows-Container-Basis-Image-Lizenz; Sie müssen nicht die Windows-Inhalt für die öffentliche oder 3. Party-Verteilung bereitstellen. Verwendung in Ihrer Umgebung ist zulässig.

## <a name="microsofts-open-ecosystem"></a>Das offene Ökosystem von Microsoft

### <a name="is-microsoft-participating-in-the-open-container-initiative-oci"></a>Nimmt Microsoft an der Open Container Initiative (OCI) teil?

Um zu gewährleisten, dass das Paketerstellungsformat universell bleibt, hat Docker vor Kurzem die Open Container Initiative (OCI) ins Leben gerufen, um sicherzustellen, dass die Paketerstellung für Container ein offenes von den Gründern geleitetes Format bleibt, wobei Microsoft eines der Gründungsmitglieder ist.

> [!TIP]
> Haben Sie eine Empfehlung für zusätzlich zu den häufig gestellten Fragen? Wir empfehlen Ihnen, um ein neues Feedback Problem unten oder öffnen Sie eine Veröffentlichungsanforderung gegen diese Dokumente mit Ihrem!
