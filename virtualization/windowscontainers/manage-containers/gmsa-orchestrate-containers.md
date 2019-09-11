---
title: Orchestrieren von Containern mit einem gMSA
description: 'So wird es gemacht: orchestrieren von Windows-Containern mit einem Group Managed Service-Konto (gMSA)'
keywords: docker, Container, Active Directory, GMSA, Orchestrierung, kubernetes, Gruppen verwaltetes Dienstkonto, Gruppen verwaltete Dienstkonten
author: Heidilohr
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b4dac775dc7a4ee6375f0d803e921527e66aae5b
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079739"
---
## <a name="orchestrate-containers-with-a-gmsa"></a>Orchestrieren von Containern mit einem gMSA

In Produktionsumgebungen verwenden Sie häufig einen Container-Orchestrator, um Ihre apps und Dienste bereitzustellen und zu verwalten. Jeder Orchestrator verfügt über eigene Verwaltungs Paradigmen und ist für die Annahme von Anmeldeinformationen für die Windows-Container Plattform verantwortlich.

Wenn Sie Container mit Gruppen verwalteten Dienstkonten (gMSAs) orchestrieren, stellen Sie Folgendes sicher:

> [!div class="checklist"]
> * Alle Container Hosts, die für die Ausführung von Containern mit gMSAs geplant werden können, sind Domänen verbunden.
> * Die Container Hosts können die Kennwörter für alle von Containern verwendeten gMSAs abrufen.
> * Die Anmeldeinformationsdateien werden erstellt und in den Orchestrator hochgeladen oder auf jeden Container Host kopiert, je nachdem, wie der Orchestrator Sie bevorzugt.
> * Container Netzwerke ermöglichen es den Containern, mit den Active Directory-Domänencontrollern zu kommunizieren, um gMSA-Tickets abzurufen.

### <a name="how-to-use-gmsa-with-service-fabric"></a>Verwenden von gMSA mit Service Fabric

Service Fabric unterstützt die Ausführung von Windows-Containern mit einem gMSA, wenn Sie den Speicherort der Anmeldeinformationen in Ihrem Anwendungsmanifest angeben. Sie müssen die Anmeldeinformationen-spec-Datei erstellen und im **CredentialSpecs** -Unterverzeichnis des docker-Datenverzeichnisses auf jedem Host platzieren, damit es von Service Fabric gefunden werden kann. Sie können das Cmdlet **Get-CredentialSpec** , Teil des CredentialSpec- [PowerShell-Moduls](https://aka.ms/credspec), ausführen, um zu überprüfen, ob sich Ihre Anmeldeinformationen am richtigen Speicherort befinden.

Weitere Informationen zum Konfigurieren der Anwendung finden Sie unter [Schnellstart: Bereitstellen von Windows-Containern in Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) und [Einrichten von gMSA für Windows-Container, die auf Service Fabric ausgeführt](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) werden.

### <a name="how-to-use-gmsa-with-docker-swarm"></a>Verwenden von gMSA mit Andock barem Schwarm

Wenn Sie einen gMSA mit Containern verwenden möchten, die von Docker Swarm verwaltet werden, führen Sie den `--credential-spec` Befehl [Andock Dienst erstellen](https://docs.docker.com/engine/reference/commandline/service_create/) mit dem folgenden Parameter aus:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Weitere Informationen zum Verwenden von Anmeldeinformationen mit Andock Diensten finden Sie im [Beispiel für andocker Swarm](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) .

### <a name="how-to-use-gmsa-with-kubernetes"></a>Verwenden von gMSA mit Kubernetes

Die Unterstützung für die Planung von Windows-Containern mit gMSAs in Kubernetes ist als Alpha-Feature in Kubernetes 1,14 verfügbar. Unter [Konfigurieren von gMSA für Windows-Pods und-Containern](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) finden Sie die neuesten Informationen zu diesem Feature und erfahren, wie Sie es in ihrer Kubernetes-Distribution testen.

## <a name="next-steps"></a>Nächste Schritte

Zusätzlich zu Orchestrierungs Containern können Sie gMSAs auch für folgende Zwecke verwenden:

- [Konfigurieren von apps](gmsa-configure-app.md)
- [Ausführen von Containern](gmsa-run-container.md)

Wenn während des Setups Probleme auftreten, finden Sie in unserem [Leitfaden zur Problembehandlung](gmsa-troubleshooting.md) mögliche Lösungen.
