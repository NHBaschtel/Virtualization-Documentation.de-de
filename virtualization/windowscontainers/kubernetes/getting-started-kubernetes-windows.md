---
title: Kubernetes unter Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Verknüpfen eines Windows-Knotens mit einem Kubernetes-Cluster mit v 1.14
keywords: kubernetes, 1,14, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 18734f102042ec951255061dcd82229e18d29a15
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998387"
---
# <a name="kubernetes-on-windows"></a>Kubernetes unter Windows

Diese Seite dient als Übersicht über die ersten Schritte mit Kubernetes unter Windows, indem Windows-Knoten zu einem Linux-basierten Cluster hinzugefügt werden. Mit der Veröffentlichung von Kubernetes 1,14 unter Windows Server, [Version 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes), können Benutzer die folgenden Features in Kubernetes unter Windows nutzen:

- **Overlay-Netzwerk**: Verwenden Sie Flanell im vxlan-Modus, um ein virtuelles Overlay-Netzwerk zu konfigurieren.
    - erfordert entweder Windows Server 2019 mit installiertem [KB4489899](https://support.microsoft.com/help/4489899) oder [Windows Server vNext erhielten Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/) Build 18317 +
    - erfordert Kubernetes v 1.14 (oder höher) mit `WinOverlay` aktiviertem Feature Gate
    - erfordert Flanell v 0.11.0 (oder höher)
- **vereinfachte Netzwerkverwaltung**: Verwenden Sie Flanell im Host-Gateway-Modus für die automatische Routen Verwaltung zwischen Knoten.
- **Verbesserungen**bei der Skalierbarkeit: nutzen Sie die schnelleren und zuverlässigeren Container-Anlaufzeiten Dank [Geräte-vNICs für Windows Server-Container](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716).
- **Hyper-v-Isolierung (Alpha)**: orchestriert die [Hyper-v-Isolierung](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) mit der Kernelmodus-Isolierung für erhöhte Sicherheit. Weitere Informationen finden Sie unter [Windows-Containertypen](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types).
    - erfordert Kubernetes v 1.10 (oder höher) mit `HyperVContainer` aktiviertem Feature Gate.
- **Speicher-Plugins**: Verwenden Sie das [FlexVolume-Speicher-Plug](https://github.com/Microsoft/K8s-Storage-Plugins) -in mit SMB-und iSCSI-Unterstützung für Windows-Container.

>[!TIP]
>Wenn Sie einen Cluster auf Azure bereitstellen möchten, ist dies mit dem Open Source AKS-Engine-Tool einfach. Weitere Informationen finden Sie in der Schritt-für-Schritt- [Anleitung](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md).

## <a name="prerequisites"></a>Voraussetzungen

### <a name="plan-ip-addressing-for-your-cluster"></a>Planen der IP-Adressierung für Ihren Cluster

<a name="definitions"></a>Da Kubernetes-Cluster neue Subnetze für Hülsen und Dienste einführen, ist es wichtig, sicherzustellen, dass keiner von Ihnen mit anderen vorhandenen Netzwerken in Ihrer Umgebung kollidiert. Hier sind alle Adressräume, die freigegeben werden müssen, damit Kubernetes erfolgreich bereitgestellt werden kann:

| Subnetz/Adressbereich | Beschreibung | Standardwert |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Dienst-Subnetz** | Ein nicht routingfähiges, rein virtuelles Subnetz, das von Pods für den gleichmäßigen Zugriff auf Dienste verwendet wird, ohne sich um die Netzwerktopologie zu kümmern. Es wird von `kube-proxy` von/auf routingfähige Adressbereiche übersetzt, die auf diesen Knoten ausgeführt werden. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**Cluster-Subnetz** |  Hierbei handelt es sich um ein globales Subnetz, das von allen Pods des Clusters verwendet wird. Jedem Knoten wird ein kleineres/24 Subnetz zugewiesen, damit Ihre Pods verwendet werden können. Es muss groß genug sein, um alle in Ihrem Cluster verwendeten Pods aufnehmen zu können. So berechnen Sie eine *minimale* subnetgröße: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Beispiel für einen 5-Knoten-Cluster für 100-Pods `(5) + (5 *  100) = 505`pro Knoten:.  | "10.244.0.0/16" |
| **Kubernetes-DNS-Dienst-IP** | Die IP-Adresse des Diensts "Kuben-DNS", der für die DNS-Auflösung #a0 Clusterdienstermittlung verwendet wird. | "10.96.0.10" |

> [!NOTE]
> Es gibt ein weiteres Andock Netzwerk (NAT), das standardmäßig beim Installieren von Docker erstellt wird. Es ist nicht erforderlich, Kubernetes unter Windows zu verwenden, da stattdessen IPS aus dem Cluster-Subnetz zugewiesen werden.

## <a name="what-you-will-accomplish"></a>Was Sie erreichen

Am Ende dieser Anleitung haben Sie:

> [!div class="checklist"]
> * Hat einen [Kubernetes-Master](./creating-a-linux-master.md) Knoten erstellt.  
> * Eine [Netzwerklösung](./network-topologies.md)ausgewählt.  
> * Einem [Windows-Worker-Knoten](./joining-windows-workers.md) oder einem Linux-Worker- [Knoten](./joining-linux-workers.md) beigetreten.  
> * Bereitstelleneiner [Beispiel-Kubernetes-Ressource](./deploying-resources.md)  
> * [Allgemeine Probleme und Fehler ](./common-problems.md) angesprochen.

## <a name="next-steps"></a>Nächste Schritte

In diesem Abschnitt haben wir über wichtige Voraussetzungen #a0 Annahmen gesprochen, die zur erfolgreichen Bereitstellung von Kubernetes unter Windows heute erforderlich sind. Weitere Informationen zum Einrichten eines Kubernetes-Masters finden Sie hier:

>[!div class="nextstepaction"]
>[Erstellen eines Kubernetes-Masters](./creating-a-linux-master.md)