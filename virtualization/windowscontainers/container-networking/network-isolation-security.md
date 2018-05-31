---
title: Windows-Containernetzwerk
description: Netzwerkisolation und Sicherheit in Windows-Containern.
keywords: Docker, Container
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 7203989483cb07423b70ff8cc644f715ba4be274
ms.sourcegitcommit: ec186664e76d413d3bf75f2056d5acb556f4205d
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/11/2018
ms.locfileid: "1876071"
---
# <a name="network-isolation-and-security"></a>Netzwerkisolation und Sicherheit

## <a name="isolation-with-network-namespaces"></a>Isolation mit Netzwerknamespaces
Jeder Containerendpunkt befindet sich in einem eigenen __Netzwerk-Namespace__. Der Verwaltungshost vNIC und die Host-Netzwerkstapel befinden sich im Standard-Netzwerknamespace. Um eine Netzwerkisolation der Container zu erzwingen, die sich auf dem gleichen Host befinden, wird für jeden Windows Server- und Hyper-V-Container ein Netzwerknamespace erstellt, in dem der Netzwerkadapter für den Container installiert wird. Windows Server-Container verwenden eine Host-vNIC für die Verbindung mit dem virtuellen Switch. Hyper-V-Container verwenden eine synthetische VM-NIC (nicht für die Utility-VM verfügbar gemacht) für die Verbindung mit dem virtuellen Switch.


![Text](media/network-compartment-visual.png)


```powershell 
Get-NetCompartment
```

## <a name="network-security"></a>Netzwerksicherheit
Je nachdem, welche Container und Netzwerktreiber verwendet werden, werden Port-ACLs durch eine Kombination von Windows-Firewall und[VFP](https://www.microsoft.com/en-us/research/project/azure-virtual-filtering-platform/) erzwungen.

### <a name="windows-server-containers"></a>Windows Server-Container
Verwenden der Windows-Hosts-Firewall (optimiert mit Netzwerknamespaces) und VFP
  * Ausgehender Standardwert: ALLE ZULASSEN
  * Eingehender Standardwert: ALLE ZULASSEN, Alle (TCP, UDP, ICMP, IGMP) nicht angeforderten Netzwerkdatenverkehr
    * VERWEIGERT JEDEN anderen Netzwerk-Datenverkehr über diese Protokolle

  > Hinweis: Vor Windows Server, Version 1709 und Windows10 Fall Creators Update, war die standardmäßige Regel *eingehend* und es wurden alle verweigert. Benutzer mit älteren Versionen können eingehende Zulassungsregeln mit ``docker run -p`` erstellen (Portweiterleitung)


### <a name="hyper-v-containers"></a>Hyper-V Container
Hyper-V Container haben ihren eigenen isolierten Kernel und führen eine eigene Instanz des Windows-Firewall mit der folgenden Konfiguration aus:
  * Standardmäßig ALLE ERLAUBEN in beiden Windows-Firewall (Dienstprogramm für den virtuellen Computer) und VFP


![Text](media/windows-firewall-containers.png)


### <a name="kubernetes-pods"></a>Kubernetes-Pods
In [Kubernetes-Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/), wird ein Infrastrukturcontainer erstellt, dem ein Endpunkt zugeordnet ist. Container (einschließlich Infrastruktur- und Worker-Container), die zum gleichen Pod gehören, teilen einen gemeinsamen Netzwerk-Namespace (dieselbe IP und Port-Speicherplatz).


![Text](media/pod-network-compartment.png)


### <a name="customizing-default-port-acls"></a>Anpassen der standardmäßigen Port-ACLs
Wenn Sie die Standardport-ACLs ändern möchten, finden Sie Informationen auf der HNS-Dokumentation (Link kommt bald). Sie müssen die Richtlinien innerhalb der folgenden Komponenten aktualisieren:

> HINWEIS: Für Hyper-V Container im transparenten und NAT-Modus können Sie die Standardport-ACLs derzeit nicht neu programmieren. Dies wird durch ein X in der Tabelle angegeben.

| Netzwerktreiber | Windows Server-Container | Hyper-V-Container  |
| -------------- |-------------------------- | ------------------- |
| Transparent | Windows-Firewall | X |
| NAT | Windows-Firewall | X |
| L2Bridge | Beide | VFP |
| L2Tunnel | Beide | VFP |
| Überlagerung  | Beide | VFP |