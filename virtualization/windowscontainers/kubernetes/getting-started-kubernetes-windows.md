---
title: Kubernetes unter Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Wenn einen Windows-Knoten zu einem Kubernetes-Cluster mit v1.13.
keywords: Kubernetes, 1,13, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 2edc940d4132870a6f7d6714bb50faef201136ba
ms.sourcegitcommit: 817a629f762a4a5d4bcff58302f2bc2408bf8be1
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/07/2019
ms.locfileid: "9149910"
---
# <a name="kubernetes-on-windows"></a>Kubernetes unter Windows #
Diese Seite dient als einen Überblick über die erste Schritte mit Kubernetes unter Windows durch Hinzufügen von Windows-Knoten zu einem Linux-basierten Cluster. Mit der Veröffentlichung von Kubernetes 1.13 auf Windows Server- [Version 1809](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)können Benutzer über die [neuesten Features](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features) in Kubernetes unter Windows Beta nutzen:

  - **überlagern Sie Netzwerke**: Verwenden des Flannel-Vxlan-Modus zum Konfigurieren eines virtuellen Überlagerung-Netzwerks
    - erfordert entweder Windows Server 2019 mit KB4482887 installiert oder [Windows Server vNext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) -Build 18317 +
    - Kubernetes v1.14 erfordert (oder höher) mit `WinOverlay` Feature Tor aktiviert
    - erfordert Flannel v0.11.0 (oder höher)
  - **vereinfachte netzwerkverwaltung**: Flannel im hostgateway-Modus für die automatische Route Verwaltung zwischen Knoten verwenden
  - **Skalierbarkeitsverbesserungen**: genießen Sie schneller und zuverlässiger Container Start Zeiten Dank [deviceless vNICs für Windows Server-Container](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/)
  - **Hyper-V-Isolierung (Alpha)**: zu koordinieren [hyper-V-Container](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) mit Kernelmodus-Isolation für erhöhte Sicherheit ([finden Sie unter Windows-Containertypen](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types))
    - Kubernetes v1.10 erfordert (oder höher) mit `HyperVContainer` Feature Tor aktiviert
  - **Speicher-Plug-Ins**: Verwenden von [FlexVolume Speicher-Plug-in](https://github.com/Microsoft/K8s-Storage-Plugins) mit SMB und iSCSI-Unterstützung für Windows-Container

> [!TIP] 
> Wenn Sie einen Cluster in Azure bereitstellen möchten, kann das open-Source-AKS-Engine-Tool. Ein schrittweise [Anleitung](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md) ist verfügbar.

## <a name="prerequisites"></a>Voraussetzungen ##

### <a name="plan-ip-addressing-for-your-cluster"></a>Planen der IP-Adressen für den cluster ###
<a name="definitions"></a>Kubernetes-Cluster Vorstellung neuer Subnetze für Pods und Dienste ist es wichtig, um sicherzustellen, dass keine davon mit anderen vorhandenen Netzwerken in Ihrer Umgebung kollidieren. Hier sind alle Leerzeichen Adresse, die freigegeben werden, um Kubernetes bereitstellen zu können:

| Subnetz / Adressbereich | Beschreibung | Standardwert |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Dienst-Subnetz** | Ein nicht routingfähiges, rein virtuelles Subnetz, das von Pods verwendet wird, um einheitlich auf Dienste zuzugreifen, ohne Rücksicht auf die Netzwerktopologie. Es wird von `kube-proxy` von/auf routingfähige Adressbereiche übersetzt, die auf diesen Knoten ausgeführt werden. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**Cluster-Subnetz** |  Dies ist eine globale Subnetz, die von allen Pods im Cluster verwendet wird. Jeder Knoten wird eine kleinere /24 zugewiesen Subnetz aus dieser für deren Pods verwenden. Es muss groß genug für alle Pods im Cluster verwendet werden. *Mindestgröße Subnetz* berechnet: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Beispiel für einen Cluster mit 5 Knoten für 100 Pods pro Knoten: `(5) + (5 *  100) = 505`.  | "10.244.0.0/16" |
| **Kubernetes DNS-Dienst-IP** | IP-Adresse "Kube-Dns"-Dienst, der für DNS-Auflösung & Cluster-Dienstermittlung verwendet wird. | "10.96.0.10" |
> [!NOTE]
> Es gibt ein anderes Docker Netzwerk (NAT), das standardmäßig erstellt wird, wenn Sie Docker zu installieren. Es ist nicht erforderlich, Kubernetes unter Windows ausgeführt werden, wie IP-Adressen aus dem Clustersubnetz stattdessen Wir weisen.



## <a name="what-you-will-accomplish"></a>Was Sie erreichen ##

Am Ende dieser Anleitung haben Sie:

> [!div class="checklist"]
> * Erstellt einen [Kubernetes-Master](./creating-a-linux-master.md) -Knoten.  
> * Eine [Lösung](./network-topologies.md)wird ausgewählt.  
> * Ein [Windows-Worker-Knoten](./joining-windows-workers.md) oder [Linux-Worker-Knoten](./joining-linux-workers.md) , die es angehören.  
> * Eine [Beispiel für Kubernetes-Ressource](./deploying-resources.md)wird bereitgestellt.  
> * [Allgemeine Probleme und Fehler ](./common-problems.md) angesprochen.

## <a name="next-steps"></a>Nächste Schritte ##
In diesem Abschnitt erwähnten wir wichtige erforderlichen Komponenten & Annahmen zur Bereitstellung von Kubernetes unter Windows erfolgreich heute erforderlich. Informationen zum Einrichten eines Kubernetes-Masters weiterhin:

> [!div class="nextstepaction"]
> [Erstellen eines Kubernetes-Masters](./creating-a-linux-master.md)