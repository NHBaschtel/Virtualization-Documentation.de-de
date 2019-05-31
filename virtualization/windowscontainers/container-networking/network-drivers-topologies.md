---
title: Windows-Container Netzwerke
description: Netzwerktreiber und Topologien für die Windows-Container.
keywords: Docker, Container
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: f044cf6f9d0457dd4cc9b444dcbeebc97f22f17b
ms.sourcegitcommit: bea2c90f31a38fc7fda356619f0dd812f79d008f
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 05/31/2019
ms.locfileid: "9685287"
---
# <a name="windows-container-network-drivers"></a>Windows-Container Netzwerktreiber  

Zusätzlich zur Nutzung des standardmäßigen NAT-Netzwerks, das von Docker unter Windows erstellt wurde, können Benutzer benutzerdefinierte Containernetzwerke festlegen. Benutzerdefinierte Netzwerke können mithilfe des Befehls docker CLI [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) erstellt werden. Unter Windows stehen folgende Netzwerktreibertypen zur Verfügung:

- **nat** – einem Netzwerk hinzugefügte Container, die mit einem NAT-Treiber erstellt wurden werden mit einem *internen* Hyper-V-Switch verbunden und enthalten eine vom Benutzer angegebene (``--subnet``) IP-Präfix-IP-Adresse. Portweiterleitung/-zuordnung vom Containerhost zu Containerendpunkten wird unterstützt.
  
  >[!NOTE]
  > NAT-Netzwerke, die unter Windows Server 2019 (oder höher) erstellt wurden, werden nach einem Neustart nicht mehr beibehalten.

  > Wenn Sie das Windows 10 Creators-Update (oder höher) installiert haben, werden mehrere NAT-Netzwerke unterstützt.
  
- **transparent** – einem Netzwerk hinzugefügte Container, die mit einem NAT-Treiber erstellt wurden und direkt mit dem physischen Netzwerk über einen *externen* Hyper-V-Switch verbunden werden. IPs aus dem physischen Netzwerk können mithilfe eines externen DHCP-Servers statisch (erfordert eine benutzerdefinierte ``--subnet``-Option) oder dynamisch zugewiesen werden.
  
  >[!NOTE]
  >Aufgrund der folgenden Anforderung wird die Verbindung ihrer Container Hosts über ein transparentes Netzwerk auf Azure VMS nicht unterstützt.
  
  > Erfordert: Wenn dieser Modus in einem Virtualisierungs-Szenario (Container Host ist eine VM) verwendet wird, _ist eine Mac-Adressen-Spoofing erforderlich_.

- **Überlagerung** - Wenn das Docker-Modul im [Schwarmmodus](../manage-containers/swarm-mode.md) ausgeführt wird, können Container, die mit einem Überlagerungsnetzwerks verbunden sind mit anderen, an dasselbe Netzwerk angeschlossen Containern, über mehrere Containerhosts kommunizieren. Jedes Überlagerungsnetzwerk, das für einen Schwarmcluster erstellt wird, wird mit einem eigenen IP-Subnetz erstellt, das durch ein privates IP-Präfix definiert ist. Der Überlagerungsnetzwerktreiber verwendet VXLAN Kapselung. **Kann mit Kubernetes verwendet werden, wenn Sie die geeignete Steuerelement Netzwerkebenen (Flannel oder OVN) verwenden.**
  > Erfordert: Stellen Sie sicher, dass Ihre Umgebung diese erforderlichen [voraus](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) setzungen für das Erstellen von Overlay-Netzwerken erfüllt.

  > Erfordert: erfordert Windows Server 2016 mit [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217), Windows 10 Creators Update oder einer neueren Version.

  >[!NOTE]
  >Unter Windows Server 2019 mit andocker EE 18,03 und höher können Overlay-Netzwerke, die von Docker Swarm erstellt wurden, VFP-NAT-Regeln für ausgehende Verbindungen nutzen. Das bedeutet, dass der angegebene Container 1 IP-Adresse erhält. Das bedeutet auch, dass ICMP-basierte Tools wie `ping` oder `Test-NetConnection` sollten mit ihren TCP/UDP-Optionen in Debugging-Situationen konfiguriert werden.

- **l2bridge** -verbundener Container, der mit einem Netzwerk mit dem Treiber "l2bridge" erstellt wurde und in demselben IP-Subnetz wie der Containerhost ist und mit dem physischen Netzwerk über einen *externen* Hyper-V-Switch verbunden ist. Die IP-Adressen müssen statisch aus dem gleichen Präfix wie der Containerhost zugewiesen werden. Alle Containerendpunkte auf dem Host verfügen aufgrund der Layer-2-Adressübersetzung beim Eingang und -Ausgang über dieselbe MAC-Adresse als Host (Umschreiben der MAC-Adresse).
  > Erfordert: erfordert Windows Server 2016, Windows 10 Creators Update oder eine neuere Version.

  > Erfordert: [OutboundNAT-Richtlinie](./advanced.md#specify-outboundnat-policy-for-a-network) für externe Konnektivität.

- **l2tunnel** – Ähnlich wie l2bridge _sollte dieser Treiber nur in einem Microsoft-Cloudstapel verwendet werden_. Pakete, die aus einem Container kommen, werden an den Virtualisierungshost gesendet, wenn SDN-Richtlinien angewendet werden.


## <a name="network-topologies-and-ipam"></a>Netzwerktopologien und IPAM

Die folgende Tabelle zeigt, wie die Netzwerkkonnektivität für interne (Container-zu-Container) und externe Verbindungen für jeden Netzwerktreiber bereitgestellt wird.

### <a name="networking-modesdocker-drivers"></a>Netzwerkmodi/docker Treiber

  | Docker-Netzwerktreiber für Windows | Typische Betriebssysteme | Container-zu-Container (einzelner Knoten) | Container-zu-extern (einzelner Knoten + Multi-Node) | Container-zu-Container (Multi-Node) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (Standard)** | Für Entwickler geeignet | <ul><li>Gleiches Subnetz: Überbrückte Verbindung über Hyper-V Virtual Switch</li><li> Übergreifendes Subnetz: nicht unterstützt (nur ein internes NAT-Präfix)</li></ul> | Weitergeleitet durch Management-vNIC (WinNAT-gebunden) | Nicht direkt unterstützt: Verfügbarmachen von Ports über Host ist erforderlich |
  | **Transparent** | Gut für Entwickler oder kleine Bereitstellungen | <ul><li>Gleiches Subnetz: Überbrückte Verbindung über Hyper-V Virtual Switch</li><li>Subnetzübergreifend: Weitergeleitet vom Container-Host</li></ul> | Vom Container-Host mit direktem Zugriff (physischer) Netzwerkadapter weitergeleitet | Vom Container-Host mit direktem Zugriff (physischer) Netzwerkadapter weitergeleitet |
  | **Überlagerung** | Gut für Multi-Node; erforderlich für andocker Swarm, verfügbar in Kubernetes | <ul><li>Gleiches Subnetz: Überbrückte Verbindung über Hyper-V Virtual Switch</li><li>Subnetzübergreifend: Netzwerkverkehr wird gekapselt und weitergeleitet über Mgmt-vNIC</li></ul> | Nicht direkt unterstützt – erfordert einen zweiten Containerendpunkt mit NAT-Netzwerk-Verbindung | Gleich/Subnetzübergreifend: Netzwerkverkehr wird mit VXLAN gekapselt und weitergeleitet über Mgmt-vNIC |
  | **L2Bridge** | Verwendet für Kubernetes und Microsoft-SDN | <ul><li>Gleiches Subnetz: Überbrückte Verbindung über Hyper-V Virtual Switch</li><li> Subnetzübergreifend: Container-MAC-Adresse werden über Internetzugangs-/-ausgangspunkte neu geschrieben und geroutet</li></ul> | Container-MAC-Adresse werden über Internetzugangs-/-ausgangspunkte neu geschrieben | <ul><li>Gleiches Subnetz: Überbrückte Verbindung</li><li>Übergreifendes Subnetz: Weiterleitung über Mgmt-vNIC auf WSv1709 und höher</li></ul> |
  | **L2Tunnel**| Nur Azure | Gleich/Subnetzübergreifend: auf dem virtuellen Hyper-V-Switch des physischen Host angeheftet, auf den Richtlinie angewendet wird | Datenverkehr muss virtuellen Azure-Netzwerk-Gateway durchlaufen. | Gleich/Subnetzübergreifend: auf dem virtuellen Hyper-V-Switch des physischen Host angeheftet, auf den Richtlinie angewendet wird |

### <a name="ipam"></a>IPAM

IP-Adressen werden von jedem Netzwerktreiber unterschiedlich belegt und diesem zugewiesen. Windows verwendet den Host-Netzwerkdienst (HNS), um dem NAT-Treiber IPAM bereitzustellen und arbeitet mit dem Docker-Schwarmmodus (interner KVS), um IPAM für die Überlagerung bereitzustellen. Alle anderen Netzwerktreiber verwenden eine externe IPAM.

| NAT-Netzwerkmodus/Treiber | IPAM |
| -------------------------|:----:|
| NAT | Dynamische IP-Zuweisung und Zuordnung durch den Host-Netzwerkdienst (HNS) aus dem internen NAT-Subnetz-Präfix |
| Transparent | Statisch oder dynamisch (mit externem DHCP-Server) IP-Verteilung und Zuweisung von IP-Adressen innerhalb des Container-Host-Netzwerkpräfix |
| Überlagerung | Dynamische IP-Zuweisung vom Docker-Modul-Schwarmmodus, verwaltet Präfixe und Zuweisung über HNS |
| L2Bridge | Statische IP-Zuweisung und Zuweisung von IP-Adressen innerhalb des Netzwerkpräfix des Container Hosts (kann auch über HNS zugewiesen werden) |
| L2Tunnel | Nur Azure – dynamische IP-Verteilung und Zuweisung von Plug-In |

### <a name="service-discovery"></a>Dienstermittlung

Die Dienstermittlung wird nur für bestimmte Windows-Netzwerktreiber unterstützt.

|  | Lokale Dienstermittlung  | Globale Dienstermittlung |
| :---: | :---------------     |  :---                |
| nat | JA | JA mit Docker EE |  
| overlay | JA | Ja mit andocker EE oder Kuben-DNS |
| transparent | NO | NEIN |
| l2bridge | NEIN | Ja mit Kuben-DNS |
