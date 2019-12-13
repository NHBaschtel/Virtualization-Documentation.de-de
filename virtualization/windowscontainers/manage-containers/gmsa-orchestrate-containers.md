---
title: Orchestrieren von Containern mit einem GMSA
description: Orchestrieren von Windows-Containern mit einem Gruppen verwalteten Dienst Konto (Group Managed Service Account, GMSA).
keywords: docker, Container, Active Directory, GMSA, Orchestrierung, kubernetes, Gruppen verwaltete Dienst Konten, Gruppen verwaltete Dienst Konten
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 3d102aac45a1becf1879a718bb255d753b215006
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910260"
---
# <a name="orchestrate-containers-with-a-gmsa"></a>Orchestrieren von Containern mit einem GMSA

In Produktionsumgebungen verwenden Sie häufig einen containerorchestrator, um Ihre apps und Dienste bereitzustellen und zu verwalten. Jeder Orchestrator hat seine eigenen Verwaltungs Paradigmen und ist dafür verantwortlich, die Spezifikationen der Anmelde Informationen für die Windows-Container Plattform zu akzeptieren.

Stellen Sie beim Orchestrieren von Containern mit Gruppen verwalteten Dienst Konten (gmsas) Folgendes sicher:

> [!div class="checklist"]
> * Alle Container Hosts, für die die Ausführung von Containern mit gmsas geplant werden kann, sind in die Domäne eingetreten.
> * Die Container Hosts haben Zugriff zum Abrufen der Kenn Wörter für alle von Containern verwendeten gmsas.
> * Die Spezifikationen der Anmelde Informationen werden erstellt und in den Orchestrator hochgeladen oder auf alle Container Hosts kopiert, je nachdem, wie der Orchestrator Sie bevorzugt.
> * Container Netzwerke ermöglichen die Kommunikation zwischen Containern und den Active Directory-Domäne Controllern zum Abrufen von GMSA-Tickets.

## <a name="how-to-use-gmsa-with-service-fabric"></a>Verwenden von GMSA mit Service Fabric

Service Fabric unterstützt das Ausführen von Windows-Containern mit einem GMSA, wenn Sie den Speicherort für die Anmelde Informations Spezifikation im Anwendungs Manifest angeben. Sie müssen die Datei mit den Anmelde Informationen erstellen und **im Unterverzeichnis** "-Unterverzeichnis" des docker-Datenverzeichnisses auf jedem Host platzieren, damit Service Fabric Sie finden kann. Sie können das Cmdlet " **Get-alidentialspec** ", das Teil des [PowerShell-Moduls "PowerShell](https://aka.ms/credspec)" von "" ist, ausführen, um zu überprüfen, ob sich Ihre Anmelde Informations Spezifikation am richtigen Speicherort befindet.

Weitere Informationen zum Konfigurieren der Anwendung finden Sie unter [Schnellstart:](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) Bereitstellen von Windows-Containern für Service Fabric und [Einrichten von GMSA für Windows-Container, die auf Service Fabric ausgeführt werden](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) .

## <a name="how-to-use-gmsa-with-docker-swarm"></a>Verwenden von GMSA mit docker Swarm

Wenn Sie ein GMSA mit Containern verwenden möchten, die von Docker Swarm verwaltet werden, führen Sie den Befehl [docker Service Create](https://docs.docker.com/engine/reference/commandline/service_create/) mit dem Parameter `--credential-spec` aus:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Weitere Informationen zur Verwendung von Spezifikationen für Anmelde Informationen mit docker-Diensten finden Sie im [docker Swarm-Beispiel](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) .

## <a name="how-to-use-gmsa-with-kubernetes"></a>Verwenden von GMSA mit Kubernetes

Die Unterstützung für die Planung von Windows-Containern mit gmsas in Kubernetes ist als Alpha Feature in Kubernetes 1,14 verfügbar. Aktuelle Informationen zu diesem Feature und zum Testen in der Kubernetes-Verteilung finden Sie unter [Konfigurieren von GMSA für Windows-Pods und-Container](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) .

## <a name="next-steps"></a>Nächste Schritte

Zusätzlich zum orchestrieren von Containern können Sie gmsas auch für Folgendes verwenden:

- [Konfigurieren von Apps](gmsa-configure-app.md)
- [Ausführen von Containern](gmsa-run-container.md)

Wenn während des Setups Probleme auftreten, finden Sie in unserem [Handbuch zur Problem](gmsa-troubleshooting.md) Behandlung mögliche Lösungen.
