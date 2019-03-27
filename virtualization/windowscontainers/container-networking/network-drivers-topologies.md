---
title: Windows-Containernetzwerk
description: Netzwerktreiber und Topologien für die Windows-Container.
keywords: Docker, Container
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: ed554acc0aaacf967cbd29d49ef67fa7e916b497
ms.sourcegitcommit: 1715411ac2768159cd9c9f14484a1cad5e7f2a5f
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/26/2019
ms.locfileid: "9263497"
---
# <a name="windows-container-network-drivers"></a>Windows-Container-Netzwerktreiber  

Zusätzlich zur Nutzung des standardmäßigen NAT-Netzwerks, das von Docker unter Windows erstellt wurde, können Benutzer benutzerdefinierte Containernetzwerke festlegen. Benutzerdefinierte Netzwerke können über den CLI Docker-Befehl [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) erstellt werden. Unter Windows stehen folgende Netzwerktreibertypen zur Verfügung:

- **nat** – einem Netzwerk hinzugefügte Container, die mit einem NAT-Treiber erstellt wurden werden mit einem *internen* Hyper-V-Switch verbunden und enthalten eine vom Benutzer angegebene (``--subnet``) IP-Präfix-IP-Adresse. Portweiterleitung/-zuordnung vom Containerhost zu Containerendpunkten wird unterstützt.
  > Mit dem Windows 10 Creators Update werden jetzt mehrere NAT-Netzwerke unterstützt.

- **transparent** – einem Netzwerk hinzugefügte Container, die mit einem NAT-Treiber erstellt wurden und direkt mit dem physischen Netzwerk über einen *externen* Hyper-V-Switch verbunden werden. IPs aus dem physischen Netzwerk können mithilfe eines externen DHCP-Servers statisch (erfordert eine benutzerdefinierte ``--subnet``-Option) oder dynamisch zugewiesen werden. 
  > Hinweis: aufgrund von der folgenden Anforderung, Herstellen einer Verbindung containerhosts über ein transparentes Netzwerk wird nicht unterstützt auf Azure-VMs.
  
  > Erfordert Folgendes: Wenn dieser Modus verwendet wird in einem Szenario mit Virtualisierung (Container-Host ist eine virtuelle Maschine) _Spoofing von MAC-Adressen erforderlich ist_.

- **Überlagerung** - Wenn das Docker-Modul im [Schwarmmodus](../manage-containers/swarm-mode.md) ausgeführt wird, können Container, die mit einem Überlagerungsnetzwerks verbunden sind mit anderen, an dasselbe Netzwerk angeschlossen Containern, über mehrere Containerhosts kommunizieren. Jedes Überlagerungsnetzwerk, das für einen Schwarmcluster erstellt wird, wird mit einem eigenen IP-Subnetz erstellt, das durch ein privates IP-Präfix definiert ist. Der Überlagerungsnetzwerktreiber verwendet VXLAN Kapselung. **Kann mit Kubernetes verwendet werden, wenn Sie die geeignete Steuerelement Netzwerkebenen (Flannel oder OVN) verwenden.**
  > Erfordert: Stellen Sie sicher, dass Ihre Umgebung diese *erforderlichen* [Komponenten](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) zum Erstellen von überlagerungsnetzwerke erfüllt.

  > Erfordert: Erfordert Windows Server 2016 mit [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217), Windows 10 Creators Update oder einer neueren Version.

  > Hinweis: Auf Windows Server 2019 Docker EE 18.03 ausgeführt wird und .NET-Apps, erstellt, indem der Docker-Schwarm überlagerungsnetzwerke nutzen Sie VFP NAT-Regeln für ausgehende Konnektivität. Dies bedeutet, dass Container erhält und 1 IP-Adresse empfängt. Es bedeutet auch, dass tools wie ICMP-basierte `ping` oder `Test-NetConnection` sollte ihre TCP/UDP-Optionen in Situationen Debuggen Verwendung konfiguriert werden.

- **l2bridge** -verbundener Container, der mit einem Netzwerk mit dem Treiber "l2bridge" erstellt wurde und in demselben IP-Subnetz wie der Containerhost ist und mit dem physischen Netzwerk über einen *externen* Hyper-V-Switch verbunden ist. Die IP-Adressen müssen statisch aus dem gleichen Präfix wie der Containerhost zugewiesen werden. Alle Containerendpunkte auf dem Host verfügen aufgrund der Layer-2-Adressübersetzung beim Eingang und -Ausgang über dieselbe MAC-Adresse als Host (Umschreiben der MAC-Adresse).
  > Erfordert Folgendes: Wenn dieser Modus verwendet wird in einem Szenario mit Virtualisierung (Container-Host ist eine virtuelle Maschine) _Spoofing von MAC-Adressen erforderlich ist_.
  
  > Erfordert: Erfordert Windows Server 2016, Windows 10 Creators Update oder einer neueren Version.

- **l2tunnel** – ähnlich wie l2bridge, jedoch _dieser Treiber sollte nur in einem Microsoft-Cloud-Stapel, z. B. Azure verwendet werden_. Pakete, die aus einem Container kommen, werden an den Virtualisierungshost gesendet, wenn SDN-Richtlinien angewendet werden.


## <a name="network-topologies-and-ipam"></a>Netzwerktopologien und IPAM
Die folgende Tabelle zeigt, wie die Netzwerkkonnektivität für interne (Container-zu-Container) und externe Verbindungen für jeden Netzwerktreiber bereitgestellt wird.

### <a name="networking-modes--docker-drivers"></a>Netzwerk-Modi/Docker-Treiber

  | Docker-Netzwerktreiber für Windows | Typische Anwendungsfälle | Container-zu-Container (Single-Node) | Container-zu-extern (Einzelknoten + mehrere Knoten) | Container-zu-Container (mehrere Knoten) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (Standard)** | Für Entwickler geeignet | <ul><li>Gleiches Subnetz: Überbrückte Verbindung über Hyper-V Virtual Switch</li><li> Plattformübergreifendes Subnetz: nicht unterstützt (nur ein NAT-internen-Präfix)</li></ul> | Weitergeleitet durch Management-vNIC (WinNAT-gebunden) | Nicht direkt unterstützt: Verfügbarmachen von Ports über Host ist erforderlich |
  | **Transparent** | Gut für Entwickler oder kleine Bereitstellungen | <ul><li>Gleiches Subnetz: Überbrückte Verbindung über Hyper-V Virtual Switch</li><li>Subnetzübergreifend: Weitergeleitet vom Container-Host</li></ul> | Vom Container-Host mit direktem Zugriff (physischer) Netzwerkadapter weitergeleitet | Vom Container-Host mit direktem Zugriff (physischer) Netzwerkadapter weitergeleitet |
  | **Überlagerung** | Gut für mit mehreren Knoten; erforderlich für Docker-Schwarms, in Kubernetes verfügbar | <ul><li>Gleiches Subnetz: Überbrückte Verbindung über Hyper-V Virtual Switch</li><li>Subnetzübergreifend: Netzwerkverkehr wird gekapselt und weitergeleitet über Mgmt-vNIC</li></ul> | Nicht direkt unterstützt – erfordert einen zweiten Containerendpunkt mit NAT-Netzwerk-Verbindung | Gleich/Subnetzübergreifend: Netzwerkverkehr wird mit VXLAN gekapselt und weitergeleitet über Mgmt-vNIC |
  | **L2Bridge** | Verwendet für Kubernetes und Microsoft-SDN | <ul><li>Gleiches Subnetz: Überbrückte Verbindung über Hyper-V Virtual Switch</li><li> Subnetzübergreifend: Container-MAC-Adresse werden über Internetzugangs-/-ausgangspunkte neu geschrieben und geroutet</li></ul> | Container-MAC-Adresse werden über Internetzugangs-/-ausgangspunkte neu geschrieben | <ul><li>Gleiches Subnetz: Überbrückte Verbindung</li><li>Subnetzübergreifend: weitergeleitet über Mgmt-vNIC bei WSv1709 und höher</li></ul> |
  | **L2Tunnel**| Nur Azure | Gleich/Subnetzübergreifend: auf dem virtuellen Hyper-V-Switch des physischen Host angeheftet, auf den Richtlinie angewendet wird | Datenverkehr muss virtuellen Azure-Netzwerk-Gateway durchlaufen. | Gleich/Subnetzübergreifend: auf dem virtuellen Hyper-V-Switch des physischen Host angeheftet, auf den Richtlinie angewendet wird |

### <a name="ipam"></a>IPAM 
IP-Adressen werden von jedem Netzwerktreiber unterschiedlich belegt und diesem zugewiesen. Windows verwendet den Host-Netzwerkdienst (HNS), um dem NAT-Treiber IPAM bereitzustellen und arbeitet mit dem Docker-Schwarmmodus (interner KVS), um IPAM für die Überlagerung bereitzustellen. Alle anderen Netzwerktreiber verwenden eine externe IPAM.

| NAT-Netzwerkmodus/Treiber | IPAM |
| -------------------------|:----:|
| NAT | Dynamische IP-Zuweisung und Zuweisung vom Host Networking Service (HNS) aus internem NAT-Subnetzpräfix |
| Transparent | Statisch oder dynamisch (mit externem DHCP-Server) IP-Verteilung und Zuweisung von IP-Adressen innerhalb des Container-Host-Netzwerkpräfix |
| Überlagerung | Dynamische IP-Zuweisung vom Docker-Modul-Schwarmmodus, verwaltet Präfixe und Zuweisung über HNS |
| L2Bridge | Statische IP-Verteilung und Zuweisung von IP-Adressen innerhalb des Container-Host-Netzwerkpräfix (kann auch über HNS zugewiesen werden) |
| L2Tunnel | Nur Azure – dynamische IP-Verteilung und Zuweisung von Plug-In |

### <a name="service-discovery"></a>Dienstermittlung
Die Dienstermittlung wird nur für bestimmte Windows-Netzwerktreiber unterstützt.

|  | Lokale Dienstermittlung  | Globale Dienstermittlung |
| :---: | :---------------     |  :---                |
| nat | JA | JA mit Docker EE |  
| overlay | JA | Ja mit Docker EE oder Kube-dns |
| transparent | NO | NEIN |
| l2bridge | NEIN | Ja mit Kube-dns |
