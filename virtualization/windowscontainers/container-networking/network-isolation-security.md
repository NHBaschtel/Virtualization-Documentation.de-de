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
ms.openlocfilehash: 77e515e5e73d1d9e32b839c568ff3e4faf961c3c
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380114"
---
# <a name="network-isolation-and-security"></a>Netzwerkisolation und Sicherheit

## <a name="isolation-with-network-namespaces"></a>Isolation mit Netzwerknamespaces

Jeder Containerendpunkt befindet sich in einem eigenen __Netzwerk-Namespace__. Der Verwaltungshost vNIC und die Host-Netzwerkstapel befinden sich im Standard-Netzwerknamespace. Um eine Netzwerkisolation der Container auf dem gleichen Host zu erzwingen, wird ein Netzwerk-Namespace für jeden Windows Server-Container erstellt, und Container unter Hyper-V-Isolation, in dem der Netzwerkadapter für den Container installiert wird, ausgeführt. Windows Server-Container verwenden eine Host-vNIC für die Verbindung mit dem virtuellen Switch. Hyper-V-Isolation wird eine synthetische VM-NIC (nicht verfügbar gemacht, die Utility-VM) mit dem virtuellen Switch angefügt.

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

  >[!NOTE]
  >Vor Windows Server, Version 1709 und Windows 10 Fall Creators Update war die standardmäßige eingehende Regel alle verweigert. Benutzer, die mit älteren Versionen können eingehende Zulassungsregeln mit erstellen ``docker run -p`` (portweiterleitung).

### <a name="hyper-v-isolation"></a>Hyper-V-Isolierung

Container, die in Hyper-V-Isolierung ausgeführt haben ihren eigenen isolierten Kernel und führen Sie eine eigene Instanz des Windows-Firewall daher mit der folgenden Konfiguration:

* Standardmäßig ALLE ERLAUBEN in beiden Windows-Firewall (Dienstprogramm für den virtuellen Computer) und VFP

![Text](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes-pods

In einem [Kubernetes-Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/)wird ein Infrastrukturcontainer erstellt, dem ein Endpunkt zugeordnet ist. Container, die zum gleichen Pod, einschließlich Infrastruktur- und Worker-Container, gehören Teilen einen gemeinsamen Netzwerknamespace (dieselbe IP und Port-Speicherplatz).

![Text](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>Anpassen der standardmäßigen Port-ACLs

Wenn Sie die Standardport-ACLs, lesen Sie bitte zuerst unsere Host Networking Service-Dokumentation (Link zu bald hinzugefügt werden) ändern möchten. Sie müssen die Richtlinien innerhalb der folgenden Komponenten aktualisieren:

>[!NOTE]
>Für Hyper-V-Isolierung im transparenten und NAT-Modus können Sie derzeit die standardmäßigen Port-ACLs nicht neu programmieren. Dies wird durch ein X in der Tabelle angegeben.

| Netzwerktreiber | Windows Server-Container | Hyper-V-Isolierung  |
| -------------- |-------------------------- | ------------------- |
| Transparent | Windows-Firewall | X |
| NAT | Windows-Firewall | X |
| L2Bridge | Beide | VFP |
| L2Tunnel | Beide | VFP |
| Überlagerung  | Beide | VFP |