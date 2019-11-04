---
title: Windows-Container – häufig gestellte Fragen
description: Häufig gestellte Fragen zu Windows Server-Containern
keywords: Docker, Container
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 405b2abc43a4ae2c546de351679deb755e4a9317
ms.sourcegitcommit: 64573b539438de6ec5564b2949642ef12e55fc62
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/01/2019
ms.locfileid: "10274067"
---
# <a name="frequently-asked-questions-about-containers"></a>Häufig gestellte Fragen zu Containern

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Worin besteht der Unterschied zwischen Linux-und Windows Server-Containern?

Sowohl Linux als auch Windows Server implementieren ähnliche Technologien in Ihren Kernel-und Core-Betriebssystemen. Der Unterschied ergibt sich aus der Plattform und Arbeitslasten, die innerhalb des Containers ausgeführt.  

Wenn ein Kunde Windows Server-Container verwendet, kann er in vorhandene Windows-Technologien wie .net, ASP.net und PowerShell integriert werden.

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Was sind die Voraussetzungen für die Ausführung von Containern unter Windows?

Container wurden mit Windows Server 2016 auf der Plattform eingeführt. Um Container verwenden zu können, benötigen Sie entweder Windows Server 2016 oder das Windows 10 Anniversary-Update (Version 1607) oder neuer. Lesen Sie die [System Voraussetzungen](../deploy-containers/system-requirements.md) , um mehr zu erfahren.

## <a name="what-are-wcow-and-lcow"></a>Was sind WCOW und LCOW?

WCOW ist für "Windows-Container unter Windows" kurz. LCOW ist für "Linux-Container unter Windows" kurz.

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>Wie werden Container lizenziert? Gibt es eine Grenze für die Anzahl der Container, die ich ausführen kann?

Das Windows-Container Abbild- [EULA](../images-eula.md) beschreibt eine Verwendung, die von einem Benutzer mit einem gültig lizenzierten Hostbetriebssystem abhängt. Die Anzahl der Container, die ein Benutzer ausführen darf, hängt von der Hostbetriebssystem-Edition und dem [Isolationsmodus](../manage-containers/hyperv-container.md) ab, mit dem ein Container ausgeführt wird, sowie davon, ob diese Container für dev/Test-Zwecke oder in der Produktion ausgeführt werden.

|Host Betriebssystem                                                         |Prozess isolierter Container Grenzwert                   |Hyper-V-isolierter Container Grenzwert                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |Unbegrenzt                                          |2                                                  |
|Windows Server Datacenter                                       |Unbegrenzt                                          |Unbegrenzt                                          |
|Windows 10 pro und Enterprise                                   |Unbegrenzt *(nur für Test-oder Entwicklungszwecke)*|Unbegrenzt *(nur für Test-oder Entwicklungszwecke)*|
|Windows 10-Core und Enterprise                             |Unbegrenzte                                         |Unbegrenzte                                          |

Die Verwendung der Windows Server-Container Bilder wird durchlesen der Anzahl der für diese [Edition](/windows-server/get-started-19/editions-comparison-19.md)unterstützten Virtualisierungs-Gäste bestimmt. <br/>

>[!NOTE]
>\ * Die Verwendung von Containern in einer viel-Edition von Windows hängt davon ab, ob Sie den Microsoft Commercial-Nutzungsbedingungen für Windows 10 Core-Lauf Zeit Bilder oder die Windows 10-Enterprise-Gerätelizenz ("Windows-Vertrag für kommerzielle Nutzung") zugestimmt haben. Zusätzliche Bestimmungen und Einschränkungen in den Windows-kommerziellen Vereinbarungen gelten für ihre Verwendung des Container Bilds in einer Produktionsumgebung. Bitte lesen Sie den [EULA für Container Bilder](../images-eula.md) , um genau zu verstehen, was erlaubt ist und was nicht.

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>Muss ich als Entwickler die APP für die einzelnen Containertypen neu schreiben?

Nein. Windows-Container Bilder sind sowohl in Windows Server-Containern als auch in der Hyper-V-Isolierung üblich. Die Auswahl des Containertyps erfolgt beim Starten des Containers. Aus Entwicklersicht sind Windows Server-Container und Hyper-V-Isolierung zwei Varianten der gleichen Aufgabe. Sie bieten die gleiche Entwicklungs-, Programmier-und Verwaltungserfahrung und sind offen und erweiterbar und umfassen das gleiche Maß an Integration und Unterstützung für docker.

Ein Entwickler kann ein Container Bild mithilfe eines Windows Server-Containers erstellen und es in der Hyper-V-Isolierung oder umgekehrt bereitstellen, ohne andere Änderungen als die Angabe des entsprechenden Lauf Zeit Kennzeichens vorzunehmen.

Windows Server-Container bieten eine größere Dichte und Leistung für den Fall, dass die Geschwindigkeit entscheidend ist, beispielsweise geringere Aufgliederungs Zeiten und schnellere Laufzeitleistung im Vergleich zu geschachtelten Konfigurationen. Die Hyper-V-Isolierung, getreu dem Namen, bietet eine größere Isolierung, um sicherzustellen, dass Code, der in einem Container ausgeführt wird, keine Kompromisse oder Auswirkungen auf das Hostbetriebssystem oder andere Container hat, die auf demselben Host ausgeführt werden. Dies ist nützlich für Multitenant-Szenarien mit Anforderungen für das Hosten von nicht vertrauenswürdigem Code, einschließlich SaaS-Anwendungen und Compute-Hosting.

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>Kann ich Windows-Container im Prozess isolierten Modus unter Windows 10 ausführen?

Beginnend mit dem 2018-Update für Windows 10 Oktober können Sie einen Windows-Container mit Prozessisolierung ausführen, doch müssen Sie zuerst die Prozessisolierung direkt anfordern `--isolation=process` , indem Sie das Flag verwenden `docker run`, wenn Sie Ihre Container ausführen. Die Prozessisolierung ist unter Windows 10 pro, Windows 10 Enterprise, Windows 10-Core und Windows 10-viel Enterprise kompatibel.

Wenn Sie Ihre Windows-Container auf diese Weise ausführen möchten, müssen Sie sicherstellen, dass auf Ihrem Host Windows 10 Build 17763 + ausgeführt wird und Sie über eine Andock Version mit Engine 18,09 oder höher verfügen.

> [!WARNING]
> Dieses Feature ist nur für die Entwicklung und Prüfung vorgesehen, abgesehen von den viel-Core-und-viele-Enterprise-Hosts (nach dem Akzeptieren zusätzlicher Bedingungen und Einschränkungen). Sie sollten Windows Server weiterhin als Host für Produktionsbereitstellungen verwenden. Wenn Sie dieses Feature verwenden, müssen Sie auch sicherstellen, dass Ihre Host-und Container Versions Tags übereinstimmen, da der Container andernfalls möglicherweise nicht gestartet wird oder nicht definiertes Verhalten aufweist.

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Wie kann ich meine Container Bilder auf Air-gapped-Computern zur Verfügung stellen?

Windows-Container-Basisbilder enthalten Artefakte, deren Verteilung durch die Lizenz eingeschränkt ist. Wenn Sie diese Bilder erstellen und Sie an eine private oder öffentliche Registrierung weiterleiten, werden Sie feststellen, dass die Basisschicht nie verschoben wird. Stattdessen verwenden wir das Konzept einer fremd Schicht, die auf die reale Basisschicht verweist, die sich im Azure Cloud-Speicher befindet.

Dies kann die Dinge komplizieren, wenn Sie einen Air-gapped-Computer haben, der nur Bilder von der Adresse Ihrer privaten Container Registrierung abrufen kann. In diesem Fall funktionieren Versuche, die fremd Schicht zu befolgen, um das Basis Bild zu erhalten, nicht. Wenn Sie das Verhalten der fremd Schicht außer Kraft setzen möchten `--allow-nondistributable-artifacts` , können Sie das Flag im docker-Daemon verwenden.

> [!IMPORTANT]
> Die Verwendung dieser Kennzeichnung steht ihrer Verpflichtung zur Einhaltung der Bestimmungen der Windows-Container-Basis Bildlizenz nicht entgegen; Sie dürfen keine Windows-Inhalte für die öffentliche oder fremde Weiterverteilung bereitstellen. Die Verwendung in ihrer eigenen Umgebung ist zulässig.

## <a name="additional-feedback"></a>Weiteres Feedback

Möchten Sie etwas zu den häufig gestellten Fragen hinzufügen? Öffnen Sie ein neues Feedback Problem im Abschnitt Kommentare, oder richten Sie eine Pull-Anforderung für diese Seite mit GitHub ein.
