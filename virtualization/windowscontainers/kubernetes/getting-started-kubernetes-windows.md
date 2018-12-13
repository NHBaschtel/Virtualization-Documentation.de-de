---
title: Kubernetes unter Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Wenn einen Windows-Knoten zu einem Kubernetes-Cluster mit v1.12.
keywords: Kubernetes, 1.12, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 0e43b2ac5b19d16721c1ba0dd1f34e339223bdaf
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178903"
---
# <a name="kubernetes-on-windows"></a>Kubernetes unter Windows #
Diese Seite dient als einen Überblick über die erste Schritte mit Kubernetes unter Windows durch das Verknüpfen von Windows-Knoten mit einem Linux-basierten Cluster. Mit der Veröffentlichung von Kubernetes 1.12 auf Windows Server [Version 1803](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1803#kubernetes) Beta können Benutzer die [neuesten Features](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features) in Kubernetes unter Windows nutzen:

  - **vereinfachte netzwerkverwaltung**: Flannel im hostgateway-Modus für die automatische Route Verwaltung zwischen Knoten verwenden
  - **Skalierbarkeitsverbesserungen**: genießen Sie schneller und zuverlässiger Container verursachte Uhrzeiten Dank [deviceless vNICs für Windows Server-Container](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/)
  - **Hyper-V-Isolierung (Alpha)**: zu koordinieren [hyper-V-Container](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers) mit Kernelmodus-Isolation für erhöhte Sicherheit ([finden Sie unter Windows-Containertypen](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types))
  - **Speicher-Plug-Ins**: Verwenden von [FlexVolume Speicher-Plug-in](https://github.com/Microsoft/K8s-Storage-Plugins) mit SMB und iSCSI-Unterstützung für Windows-Container

> [!TIP] 
> Um auf einfache Weise einen Cluster in Azure bereitzustellen, können Sie das Open-Source-Tool ACS-Engine verwenden. Ein schrittweise [Anleitung](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md) ist verfügbar.

## <a name="prerequisites"></a>Voraussetzungen ##

### <a name="plan-ip-addressing-for-your-cluster"></a>Planen der IP-Adressen für den cluster ###
<a name="definitions"></a>Kubernetes-Cluster Vorstellung neuer Subnetze für Pods und Dienste ist es wichtig, um sicherzustellen, dass keine davon mit anderen vorhandenen Netzwerken in Ihrer Umgebung kollidieren. Hier sind alle Leerzeichen Adresse, die freigegeben werden, um Kubernetes bereitstellen müssen:

| Subnetz / Adressbereich | Beschreibung | Standardwert |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**Dienst-Subnetz** | Ein nicht routingfähiges, rein virtuelles Subnetz, das von Pods verwendet wird, um einheitlich auf Dienste zuzugreifen, ohne Rücksicht auf die Netzwerktopologie. Es wird von `kube-proxy` von/auf routingfähige Adressbereiche übersetzt, die auf diesen Knoten ausgeführt werden. | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**Cluster-Subnetz** |  Hierbei handelt es sich um eine globale Subnetz, die von allen Pods im Cluster verwendet wird. Jeder Knoten wird eine kleinere /24 zugewiesen Subnetz aus dieser für deren Pods verwenden. Es muss groß genug für alle Pods im Cluster verwendet werden. *Mindestgröße Subnetz* berechnet: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>Beispiel für einen Cluster mit 5 Knoten 100 Pods pro Knoten: `(5) + (5 *  100) = 505`.  | "10.244.0.0/16" |
| **Kubernetes DNS-Dienst-IP** | IP-Adresse "Kube-Dns"-Dienst, der für DNS-Auflösung und Cluster Dienstermittlung verwendet werden. | "10.96.0.10" |
> [!NOTE]
> Es gibt ein anderes Docker Netzwerk (NAT), das standardmäßig erstellt wird, wenn Sie Docker zu installieren. Es ist nicht erforderlich, Kubernetes unter Windows ausgeführt werden, wie IP-Adressen aus dem Clustersubnetz stattdessen Wir weisen.

### <a name="disable-anti-spoofing-protection"></a>Deaktivieren Sie Anti-spoofing-Schutz ###
> [!Important] 
> Bitte lesen Sie diesen Abschnitt sorgfältig, je nach Bedarf für alle Personen mit erfolgreich VMs Kubernetes unter Windows heute bereitstellen.

Stellen Sie sicher, Spoofing von MAC-Adressen und Virtualisierung für den Windows-Container-Host VMs (Gäste) aktiviert ist. Um dies zu erreichen, sollten Sie die folgenden auf dem Computer hostet die virtuellen Computer (Beispiel für Hyper-V) als Administrator ausführen:

```powershell
Set-VMProcessor -VMName "<name>" -ExposeVirtualizationExtensions $true 
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```
> [!TIP]
> Wenn Sie ein Produkt VMware-basierte je nach Bedarf Virtualisierung verwenden, suchen Sie in den [promisken Modus](https://kb.vmware.com/s/article/1004099) für die MAC-spoofing-Anforderung aktivieren.

>[!TIP]
> Wenn Sie Kubernetes auf Azure IaaS-VMs selbst bereitstellen, suchen Sie in VMs, die [geschachtelte Virtualisierung](https://azure.microsoft.com/en-us/blog/nested-virtualization-in-azure/) für diese Anforderung zu unterstützen.

## <a name="what-you-will-accomplish"></a>Was Sie erreichen ##

Am Ende dieser Anleitung haben Sie:

> [!div class="checklist"]
> * Erstellt einen [Kubernetes-Master](./creating-a-linux-master.md) -Knoten.  
> * Ausgewählt, eine [Netzwerk-Lösung](./network-topologies.md).  
> * Ein [Windows-Worker-Knoten](./joining-windows-workers.md) oder [Linux-Worker-Knoten](./joining-linux-workers.md) , die es angehören.  
> * Ein [Beispiel für Kubernetes-Ressource](./deploying-resources.md)wird bereitgestellt.  
> * [Allgemeine Probleme und Fehler ](./common-problems.md) angesprochen.

## <a name="next-steps"></a>Nächste Schritte ##
In diesem Abschnitt haben wir uns mit wichtigen erforderlichen Komponenten & Annahmen zur Bereitstellung von Kubernetes unter Windows erfolgreich heute erforderlich. Informationen zum Einrichten eines Kubernetes-Masters weiterhin:

> [!div class="nextstepaction"]
> [Erstellen eines Kubernetes-Masters](./creating-a-linux-master.md)