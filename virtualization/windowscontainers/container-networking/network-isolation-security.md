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
ms.openlocfilehash: b39ec17ac04995e8e1ce8795b5721df7a291e31c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910590"
---
# <a name="network-isolation-and-security"></a>Netzwerk Isolation und-Sicherheit

## <a name="isolation-with-network-namespaces"></a>Isolation mit Netzwerknamespaces

Jeder Containerendpunkt befindet sich in einem eigenen __Netzwerk-Namespace__. Der Verwaltungshost vNIC und die Host-Netzwerkstapel befinden sich im Standard-Netzwerknamespace. Um die Netzwerk Isolation zwischen Containern auf demselben Host zu erzwingen, wird für jeden Windows Server-Container ein Netzwerk Namespace erstellt, und Container werden unter Hyper-V-Isolation ausgeführt, in der der Netzwerkadapter für den Container installiert ist. Windows Server-Container verwenden eine Host-vNIC für die Verbindung mit dem virtuellen Switch. Die Hyper-V-Isolation verwendet eine synthetische VM-NIC (die nicht für die VM des-Hilfsprogramms verfügbar gemacht wird), um Sie an den virtuellen Switch

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
  >Vor Windows Server, Version 1709 und Windows 10 Fall Creators Update, lautete die Standardregel "Alle ablehnen". Benutzer, die diese älteren Releases ausführen, können eingehende Zulassungsregeln mit ``docker run -p`` erstellen (Port Weiterleitung).

### <a name="hyper-v-isolation"></a>Hyper-V-Isolierung

Container, die in Hyper-V-Isolation ausgeführt werden, verfügen über einen eigenen isolierten Kernel und führen daher eine eigene Instanz der Windows-Firewall mit der folgenden Konfiguration aus:

* Standardmäßig ALLE ERLAUBEN in beiden Windows-Firewall (Dienstprogramm für den virtuellen Computer) und VFP

![Text](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes-Pods

In einem [Kubernetes Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)wird zuerst ein Infrastruktur Container erstellt, an den ein Endpunkt angefügt wird. Container, die zum selben Pod gehören, einschließlich Infrastruktur-und workercontainern, nutzen einen gemeinsamen Netzwerk Namespace (gleicher IP-und Port-Speicherplatz).

![Text](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>Anpassen der standardmäßigen Port-ACLs

Wenn Sie die Standardport-ACLs ändern möchten, lesen Sie zuerst unsere Dokumentation zum Host Netzwerkdienst (der Link wird in Kürze hinzugefügt). Sie müssen Richtlinien in den folgenden Komponenten aktualisieren:

>[!NOTE]
>Für die Hyper-V-Isolation im transparenten und NAT-Modus können Sie die Standardport-ACLs zurzeit nicht neu programmieren. Dies wird durch ein X in der Tabelle angegeben.

| Netzwerktreiber | Windows Server-Container | Hyper-V-Isolierung  |
| -------------- |-------------------------- | ------------------- |
| Transparent | Windows-Firewall | X |
| NAT | Windows-Firewall | X |
| L2Bridge | Beide | VFP |
| L2Tunnel | Beide | VFP |
| Überlagerung  | Beide | VFP |