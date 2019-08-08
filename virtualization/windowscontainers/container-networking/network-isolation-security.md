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
ms.openlocfilehash: d5081104f1614a91d6441a5e879a439f1df1bf77
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998537"
---
# <a name="network-isolation-and-security"></a>Netzwerkisolierung und-Sicherheit

## <a name="isolation-with-network-namespaces"></a>Isolierung mit Netzwerk-Namespaces

Jeder Containerendpunkt befindet sich in einem eigenen __Netzwerk-Namespace__. Der Verwaltungshost vNIC und die Host-Netzwerkstapel befinden sich im Standard-Netzwerknamespace. Um die Netzwerkisolierung zwischen Containern auf demselben Host zu erzwingen, wird ein Netzwerk Namespace für jeden Windows Server-Container erstellt, und Container werden unter Hyper-V-Isolierung ausgeführt, in die der Netzwerkadapter für den Container installiert ist. Windows Server-Container verwenden eine Host-vNIC für die Verbindung mit dem virtuellen Switch. Die Hyper-V-Isolierung verwendet eine synthetische VM-NIC (nicht für das Dienstprogramm VM verfügbar gemacht), die an den virtuellen Switch angefügt werden soll.

![Text](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>Netzwerksicherheit

Je nachdem, welche Container und Netzwerktreiber verwendet werden, werden Port-ACLs durch eine Kombination von Windows-Firewall und[VFP](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/) erzwungen.

### <a name="windows-server-containers"></a>Windows Server-Container

Verwenden der Windows-Hosts-Firewall (optimiert mit Netzwerknamespaces) und VFP

* Ausgehender Standardwert: ALLE ZULASSEN
* Eingehender Standardwert: ALLE ZULASSEN, Alle (TCP, UDP, ICMP, IGMP) nicht angeforderten Netzwerkdatenverkehr
  * VERWEIGERT JEDEN anderen Netzwerk-Datenverkehr über diese Protokolle

  >[!NOTE]
  >Vor Windows Server, Version 1709 und Windows 10 Fall Creators Update, war die standardmäßige eingehende Regel "Alle verweigern". Benutzer, die diese älteren Versionen ausführen, können eingehende ``docker run -p`` Zulassungsregeln mit (Portweiterleitung) erstellen.

### <a name="hyper-v-isolation"></a>Hyper-V-Isolierung

Container, die in der Hyper-V-Isolierung ausgeführt werden, verfügen über einen eigenen isolierten Kernel und führen daher eine eigene Instanz der Windows-Firewall mit der folgenden Konfiguration aus:

* Standardmäßig ALLE ERLAUBEN in beiden Windows-Firewall (Dienstprogramm für den virtuellen Computer) und VFP

![Text](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes-Pods

In einem [Kubernetes-Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)wird zunächst ein Infrastrukturcontainer erstellt, an den ein Endpunkt angefügt ist. Container, die zum gleichen Pod gehören, einschließlich Infrastruktur und worker-Container, geben einen gemeinsamen Netzwerk-Namespace (gleichen IP-und Portbereich) frei.

![Text](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>Anpassen der standardmäßigen Port-ACLs

Wenn Sie die standardmäßigen Port-ACLs ändern möchten, lesen Sie zuerst unsere Dokumentation zum Host-Netzwerkdienst (Link wird in Kürze hinzugefügt). Sie müssen Richtlinien innerhalb der folgenden Komponenten aktualisieren:

>[!NOTE]
>Bei der Hyper-V-Isolierung im transparenten und NAT-Modus können Sie die Standardport-ACLs zurzeit nicht erneut programmieren. Dies wird durch ein X in der Tabelle angegeben.

| Netzwerktreiber | Windows Server-Container | Hyper-V-Isolierung  |
| -------------- |-------------------------- | ------------------- |
| Transparent | Windows-Firewall | X |
| NAT | Windows-Firewall | X |
| L2Bridge | Beide | VFP |
| L2Tunnel | Beide | VFP |
| Überlagerung  | Beide | VFP |