---
title: Übersicht über die Windows-Container Orchestrierung
description: Informationen zu Windows-Container-Orchestrators.
keywords: Docker, Container
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 99a3b47a9d80e21c246fb3b4f61d650557eb37fa
ms.sourcegitcommit: e9dda81f1f68359ece9ef132a184a30880bcdb1b
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 09/27/2019
ms.locfileid: "10161787"
---
# <a name="windows-container-orchestration-overview"></a>Übersicht über die Windows-Container Orchestrierung

Aufgrund ihrer geringen Größe und Anwendungsausrichtung eignen sich Container ideal für Agile Bereitstellungsumgebungen und auf microservice basierende Architekturen. Eine Umgebung, die Container und Microserver verwendet, kann jedoch Hunderte oder Tausende von Komponenten aufweisen, die nachverfolgt werden können. Möglicherweise können Sie ein paar Dutzend virtuelle Maschinen oder physische Server manuell verwalten, aber es gibt keine Möglichkeit, eine Containerumgebung mit Produktionsmaßstab ohne Automatisierung ordnungsgemäß zu verwalten. Diese Aufgabe sollte auf Ihren Orchestrator zurückgehen, bei dem es sich um einen Prozess handelt, der eine große Anzahl von Containern automatisiert und verwaltet und wie Sie miteinander interagieren.

Orchestrators führen die folgenden Aufgaben aus:

- Terminplanung: Wenn ein Container Bild und eine Ressourcenanforderung angegeben wird, findet der Orchestrator eine geeignete Maschine, auf der der Container ausgeführt werden kann.
- Affinität/antiaffinität: Geben Sie an, ob eine Gruppe von Containern für die Leistung oder weit auseinander zur Verfügbarkeit in der Nähe ausgeführt werden soll.
- Statusüberwachung: Achten Sie auf Containerfehler und planen Sie diese automatisch erneut.
- Failover: behalten Sie den Überblick, was auf jedem Computer ausgeführt wird, und planen Sie Container von fehlerhaften Computern auf fehlerhafte Knoten um.
- Skalierung: Fügen Sie Container Instanzen hinzu, oder entfernen Sie Sie, um den Bedarf manuell oder automatisch abzugleichen.
- Netzwerk: Stellen Sie ein Overlay-Netzwerk bereit, das Container für die Kommunikation über mehrere Hostcomputer koordiniert.
- Dienstermittlung: Ermöglichen Sie den Containern, sich auch dann gegenseitig automatisch zu suchen, wenn diese zwischen Hostcomputern wechseln und sich die IP-Adressen ändern.
- Koordinierte Anwendungsupgrades: Verwalten Sie Containerupgrades, um die Ausfallzeiten der Anwendung zu vermeiden und ermöglichen Sie ein Rollback, wenn ein Fehler auftritt.

## <a name="orchestrator-types"></a>Orchestrator-Typen

Azure bietet zwei Container-Orchestrators: Azure Kubernetes Service (AKS) und Service Fabric.

Der [Azure Kubernetes-Dienst (AKS)](/azure/aks/) vereinfacht das Erstellen, konfigurieren und Verwalten eines Clusters von virtuellen Computern, die für die Ausführung von Containeranwendungen vorkonfiguriert sind. Auf diese Weise können Sie Ihre vorhandenen Fähigkeiten verwenden und auf eine große und wachsende Community-Expertise zurückgreifen, um Container basierte Anwendungen auf Microsoft Azure bereitzustellen und zu verwalten. Durch die Verwendung von AKS können Sie die Vorteile der Enterprise-Features von Azure nutzen, während die Portabilität von Anwendungen über Kubernetes und das Bildformat Andocken beibehalten wird.

[Azure Service Fabric](/azure/service-fabric/) ist eine Plattform für verteilte Systeme, mit der das Verpacken, Bereitstellen und Verwalten von skalierbaren und zuverlässigen Microservices und Containern vereinfacht wird. Service Fabric behandelt die großen Herausforderungen bei der Entwicklung und Verwaltung von systemeigenen Cloudanwendungen. Entwickler und Administratoren können komplexe Infrastrukturprobleme vermeiden und sich auf die Implementierung kritischer, anspruchsvoller Workloads konzentrieren, die skalierbar, zuverlässig und überschaubar sind. Service Fabric ist die Plattform der nächsten Generation für das Erstellen und Verwalten von Anwendungen der Ebene1, der Unternehmensklasse und der Cloudebene, die im Container ausgeführt werden.

## <a name="getting-started"></a>Erste Schritte

Informationen zum Einstieg in die Bereitstellung des Azure Kubernetes-Diensts finden Sie im [Kubernetes-Setup Leit Faden](../kubernetes/getting-started-kubernetes-windows.md).

Informationen zum Einstieg in die Bereitstellung von Azure Service Fabric finden Sie unter [Service Fabric-Schnellstart](/azure/service-fabric/service-fabric-quickstart-containers.md).