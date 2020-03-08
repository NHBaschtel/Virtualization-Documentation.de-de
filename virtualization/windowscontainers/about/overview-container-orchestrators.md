---
title: Übersicht über die Windows-Container Orchestrierung
description: Erfahren Sie mehr über Windows-containerorchestratoren.
keywords: Docker, Container
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 23dd1e56ba68a679945779f5e7dbc15225412934
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853905"
---
# <a name="windows-container-orchestration-overview"></a>Übersicht über die Windows-Container Orchestrierung

Aufgrund ihrer geringen Größe und Anwendungs Ausrichtung eignen sich Container ideal für Agile-Übermittlungs Umgebungen und auf microservice basierende Architekturen. Eine Umgebung, in der Container und-Dienste verwendet werden, kann jedoch Hunderte oder Tausende von Komponenten enthalten, die nachverfolgt werden können. Möglicherweise sind Sie in der Lage, einige Dutzend virtuelle Computer oder physische Server manuell zu verwalten, aber es gibt keine Möglichkeit, eine Container Umgebung in der Produktionsumgebung ohne Automatisierung ordnungsgemäß zu verwalten. Diese Aufgabe sollte in ihren Orchestrator fallen, einem Prozess, bei dem eine große Anzahl von Containern automatisiert und verwaltet wird und wie Sie miteinander interagieren.

Orchestratoren führen die folgenden Aufgaben aus:

- Zeitplanung: bei Angabe eines Container Images und einer Ressourcen Anforderung findet der Orchestrator einen geeigneten Computer, auf dem der Container ausgeführt werden soll.
- Affinität/antiaffinität: Geben Sie an, ob eine Gruppe von Containern für die Leistung oder weit auseinander geführt werden soll, um die Verfügbarkeit zu unterscheiden.
- Statusüberwachung: Achten Sie auf Containerfehler und planen Sie diese automatisch erneut.
- Failover: verfolgen Sie, was auf jedem Computer ausgeführt wird, und planen Sie Container von fehlgeschlagenen Computern auf fehlerfreie Knoten.
- Skalierung: Fügen Sie Container Instanzen hinzu, oder entfernen Sie diese, um Sie manuell oder automatisch abzugleichen.
- Netzwerk: Geben Sie ein Überlagerungs Netzwerk an, mit dem Container auf mehrere Host Computer verteilt werden.
- Dienstermittlung: Ermöglichen Sie den Containern, sich auch dann gegenseitig automatisch zu suchen, wenn diese zwischen Hostcomputern wechseln und sich die IP-Adressen ändern.
- Koordinierte Anwendungsupgrades: Verwalten Sie Containerupgrades, um die Ausfallzeiten der Anwendung zu vermeiden und ermöglichen Sie ein Rollback, wenn ein Fehler auftritt.

## <a name="orchestrator-types"></a>Orchestrator-Typen

Azure bietet zwei containerorchestratoren: Azure Kubernetes Service (AKS) und Service Fabric.

[Azure Kubernetes Service (AKS)](/azure/aks/) vereinfacht das Erstellen, konfigurieren und Verwalten eines Clusters mit virtuellen Computern, die zum Ausführen von Container Anwendungen vorkonfiguriert sind. Dies ermöglicht es Ihnen, Ihre vorhandenen Fähigkeiten zu nutzen und sich auf ein großes und wachsendes Fachwissen der Community zu verlassen, um Container basierte Anwendungen auf Microsoft Azure bereitzustellen und zu verwalten. Mithilfe von AKS können Sie die Vorteile der Features von Azure für Unternehmen nutzen, während Sie weiterhin die Portabilität von Anwendungen über Kubernetes und das docker-Image Format aufrechterhalten.

[Azure Service Fabric](/azure/service-fabric/) ist eine Plattform für verteilte Systeme, mit der das Verpacken, Bereitstellen und Verwalten von skalierbaren und zuverlässigen Microservices und Containern vereinfacht wird. Service Fabric behandelt die großen Herausforderungen bei der Entwicklung und Verwaltung von systemeigenen Cloudanwendungen. Entwickler und Administratoren können komplexe Infrastrukturprobleme vermeiden und sich auf die Implementierung kritischer, anspruchsvoller Workloads konzentrieren, die skalierbar, zuverlässig und überschaubar sind. Service Fabric ist die Plattform der nächsten Generation für das Erstellen und Verwalten von Anwendungen der Ebene 1, der Unternehmensklasse und der Cloudebene, die im Container ausgeführt werden.

## <a name="getting-started"></a>Erste Schritte

Informationen zu den ersten Schritten bei der Bereitstellung von Azure Kubernetes Service finden Sie im [Kubernetes-Installationshandbuch](../kubernetes/getting-started-kubernetes-windows.md)

Informationen zu den ersten Schritten bei der Bereitstellung von Azure Service Fabric finden Sie im [Service Fabric Schnellstart](/azure/service-fabric/service-fabric-quickstart-containers.md).
