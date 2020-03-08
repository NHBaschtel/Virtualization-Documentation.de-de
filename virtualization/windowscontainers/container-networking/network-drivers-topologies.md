---
title: Windows-Container Netzwerk
description: Netzwerktreiber und Topologien für die Windows-Container.
keywords: Docker, Container
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: f54c715f474c50c4b3073912adc4e0ab1c42d662
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853924"
---
# <a name="windows-container-network-drivers"></a>Treiber für Windows-Container Netzwerk  

Zusätzlich zur Nutzung des standardmäßigen NAT-Netzwerks, das von Docker unter Windows erstellt wurde, können Benutzer benutzerdefinierte Containernetzwerke festlegen. Benutzerdefinierte Netzwerke können mithilfe des Befehls docker CLI [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) erstellt werden. Unter Windows stehen folgende Netzwerktreibertypen zur Verfügung:

- **nat** – einem Netzwerk hinzugefügte Container, die mit einem NAT-Treiber erstellt wurden werden mit einem *internen* Hyper-V-Switch verbunden und enthalten eine vom Benutzer angegebene (``--subnet``) IP-Präfix-IP-Adresse. Portweiterleitung/-zuordnung vom Containerhost zu Containerendpunkten wird unterstützt.
  
  >[!NOTE]
  > Auf Windows Server 2019 (oder höher) erstellte NAT-Netzwerke werden nach dem Neustart nicht mehr beibehalten.

  > Wenn Sie das Windows 10 Creators Update installiert haben (oder höher), werden mehrere NAT-Netzwerke unterstützt.
  
- **transparent** – einem Netzwerk hinzugefügte Container, die mit einem NAT-Treiber erstellt wurden und direkt mit dem physischen Netzwerk über einen *externen* Hyper-V-Switch verbunden werden. IPs aus dem physischen Netzwerk können mithilfe eines externen DHCP-Servers statisch (erfordert eine benutzerdefinierte ``--subnet``-Option) oder dynamisch zugewiesen werden.
  
  >[!NOTE]
  >Aufgrund der folgenden Anforderung wird das Verbinden Ihrer Container Hosts über ein transparentes Netzwerk auf Azure-VMS nicht unterstützt.
  
  > Erfordert Folgendes: Wenn dieser Modus in einem Virtualisierungsszenario (Container Host ist eine VM) verwendet wird, _ist das Spoofing von Mac-Adressen erforderlich_.

- **Überlagerung** - Wenn das Docker-Modul im [Schwarmmodus](../manage-containers/swarm-mode.md) ausgeführt wird, können Container, die mit einem Überlagerungsnetzwerks verbunden sind mit anderen, an dasselbe Netzwerk angeschlossen Containern, über mehrere Containerhosts kommunizieren. Jedes Überlagerungsnetzwerk, das für einen Schwarmcluster erstellt wird, wird mit einem eigenen IP-Subnetz erstellt, das durch ein privates IP-Präfix definiert ist. Der Überlagerungsnetzwerktreiber verwendet VXLAN Kapselung. **Kann mit Kubernetes verwendet werden, wenn geeignete Netzwerk steuerungsflächen (z. b. Flannel) verwendet werden.**
  > Erfordert: Stellen Sie sicher, dass Ihre Umgebung diese erforderlichen [Voraussetzungen](https://docs.docker.com/network/overlay/#operations-for-all-overlay-networks) zum Erstellen von Überlagerungs Netzwerken erfüllt.

  > Erfordert: auf Windows Server 2019 ist hierfür [KB4489899](https://support.microsoft.com/help/4489899)erforderlich.

  > Erfordert: auf Windows Server 2016 ist hierfür [KB4015217](https://support.microsoft.com/help/4015217/windows-10-update-kb4015217)erforderlich.

  >[!NOTE]
  >Unter Windows Server 2019 nutzen Überlagerungs Netzwerke, die von Docker Swarm erstellt wurden, VFP-NAT-Regeln für ausgehende Verbindungen. Dies bedeutet, dass ein bestimmter Container 1 IP-Adresse erhält. Dies bedeutet auch, dass ICMP-basierte Tools wie `ping` oder `Test-NetConnection` mit ihren TCP/UDP-Optionen in Debuggingsituationen konfiguriert werden sollten.

- **l2bridge** ähnlich wie `transparent` Netzwerkmodus werden Container, die mit einem Netzwerk verbunden sind, das mit dem Treiber "l2bridge" erstellt wurde, über einen *externen* Hyper-V-Switch mit dem physischen Netzwerk verbunden. Der Unterschied in l2bridge besteht darin, dass Container Endpunkte die gleiche Mac-Adresse wie der Host aufweisen, da der Vorgang für die Übersetzung der Layer-2-Adressübersetzung (Mac Re-Write) bei eingehenden und ausgehenden Daten erfolgt. In Clustering-Szenarien trägt dies zu einer Verringerung der Belastung von Switches bei, die Mac-Adressen von manchmal kurzlebigen Containern erlernen müssen. L2bridge-Netzwerke können auf zwei verschiedene Arten konfiguriert werden:
  1. L2bridge Network ist mit dem gleichen IP-Subnetz wie der Container Host konfiguriert.
  2. L2bridge Network ist mit einem neuen benutzerdefinierten IP-Subnetz konfiguriert.
  
  In Configuration 2 müssen Benutzer einen Endpunkt auf dem Host Netzwerk Depot hinzufügen, das als Gateway fungiert, und Routing Funktionen für das angegebene Präfix konfigurieren. 
  > Erfordert: erfordert Windows Server 2016, Windows 10 Creators Update oder eine spätere Version.


- **l2bridge** ähnlich wie `transparent` Netzwerkmodus werden Container, die mit einem Netzwerk verbunden sind, das mit dem Treiber "l2bridge" erstellt wurde, über einen *externen* Hyper-V-Switch mit dem physischen Netzwerk verbunden. Der Unterschied in l2bridge besteht darin, dass Container Endpunkte die gleiche Mac-Adresse wie der Host aufweisen, da der Vorgang für die Übersetzung der Layer-2-Adressübersetzung (Mac Re-Write) bei eingehenden und ausgehenden Daten erfolgt. In Clustering-Szenarien trägt dies zu einer Verringerung der Belastung von Switches bei, die Mac-Adressen von manchmal kurzlebigen Containern erlernen müssen. L2bridge-Netzwerke können auf zwei verschiedene Arten konfiguriert werden:
  1. L2bridge Network ist mit dem gleichen IP-Subnetz wie der Container Host konfiguriert.
  2. L2bridge Network ist mit einem neuen benutzerdefinierten IP-Subnetz konfiguriert.
  
  In Configuration 2 müssen Benutzer einen Endpunkt auf dem Host Netzwerk Depot hinzufügen, das als Gateway fungiert, und Routing Funktionen für das angegebene Präfix konfigurieren. 
  >[!TIP]
  >Weitere Informationen zum Konfigurieren und Installieren von l2bridge finden Sie [hier](https://techcommunity.microsoft.com/t5/networking-blog/l2bridge-container-networking/ba-p/1180923).

- **l2tunnel** ähnlich wie bei l2bridge _sollte dieser Treiber jedoch nur in einem Microsoft Cloud Stapel (Azure) verwendet werden_. Pakete, die aus einem Container kommen, werden an den Virtualisierungshost gesendet, wenn SDN-Richtlinien angewendet werden.


## <a name="network-topologies-and-ipam"></a>Netzwerktopologien und IPAM

Die folgende Tabelle zeigt, wie die Netzwerkkonnektivität für interne (Container-zu-Container) und externe Verbindungen für jeden Netzwerktreiber bereitgestellt wird.

### <a name="networking-modesdocker-drivers"></a>Netzwerk Modi/docker-Treiber

  | Docker-Netzwerktreiber für Windows | Typische Verwendungen | Container-zu-Container (einzelner Knoten) | Container-zu-extern (einzelner Knoten + mehrere Knoten) | Container-zu-Container (mehrere Knoten) |
  |-------------------------------|:------------:|:------------------------------------:|:------------------------------------------------:|:-----------------------------------:|
  | **NAT (Standard)** | Für Entwickler geeignet | <ul><li>Gleiches Subnetz: Überbrückte Verbindung über Hyper-V Virtual Switch</li><li> Cross-Subnetz: nicht unterstützt (nur ein internes NAT-Präfix)</li></ul> | Weitergeleitet durch Management-vNIC (WinNAT-gebunden) | Nicht direkt unterstützt: Verfügbarmachen von Ports über Host ist erforderlich |
  | **Sichtigen** | Gut für Entwickler oder kleine Bereitstellungen | <ul><li>Gleiches Subnetz: Überbrückte Verbindung über Hyper-V Virtual Switch</li><li>Subnetzübergreifend: Weitergeleitet vom Container-Host</li></ul> | Vom Container-Host mit direktem Zugriff (physischer) Netzwerkadapter weitergeleitet | Vom Container-Host mit direktem Zugriff (physischer) Netzwerkadapter weitergeleitet |
  | **Überlagerung** | Gut für mehrere Knoten; erforderlich für docker Swarm, verfügbar in Kubernetes | <ul><li>Gleiches Subnetz: Überbrückte Verbindung über Hyper-V Virtual Switch</li><li>Subnetzübergreifend: Netzwerkverkehr wird gekapselt und weitergeleitet über Mgmt-vNIC</li></ul> | Nicht direkt unterstützt: erfordert einen zweiten Container Endpunkt, der an das NAT-Netzwerk unter Windows Server 2016 oder VFP-NAT-Regel auf Windows Server 2019 angeschlossen ist.  | Gleich/Subnetzübergreifend: Netzwerkverkehr wird mit VXLAN gekapselt und weitergeleitet über Mgmt-vNIC |
  | **L2Bridge** | Verwendet für Kubernetes und Microsoft-SDN | <ul><li>Gleiches Subnetz: Überbrückte Verbindung über Hyper-V Virtual Switch</li><li> Subnetzübergreifend: Container-MAC-Adresse werden über Internetzugangs-/-ausgangspunkte neu geschrieben und geroutet</li></ul> | Container-MAC-Adresse werden über Internetzugangs-/-ausgangspunkte neu geschrieben | <ul><li>Gleiches Subnetz: Überbrückte Verbindung</li><li>Cross-Subnetz: weitergeleitet über Mgmt vNIC auf WSv1809 und höher</li></ul> |
  | **L2Tunnel**| Nur Azure | Gleich/Subnetzübergreifend: auf dem virtuellen Hyper-V-Switch des physischen Host angeheftet, auf den Richtlinie angewendet wird | Datenverkehr muss virtuellen Azure-Netzwerk-Gateway durchlaufen. | Gleich/Subnetzübergreifend: auf dem virtuellen Hyper-V-Switch des physischen Host angeheftet, auf den Richtlinie angewendet wird |

### <a name="ipam"></a>IPAM

IP-Adressen werden von jedem Netzwerktreiber unterschiedlich belegt und diesem zugewiesen. Windows verwendet den Host-Netzwerkdienst (HNS), um dem NAT-Treiber IPAM bereitzustellen und arbeitet mit dem Docker-Schwarmmodus (interner KVS), um IPAM für die Überlagerung bereitzustellen. Alle anderen Netzwerktreiber verwenden eine externe IPAM.

| NAT-Netzwerkmodus/Treiber | IPAM |
| -------------------------|:----:|
| NAT | Dynamische IP-Zuweisung und Zuweisung durch den Host Netzwerkdienst (HNS) aus dem internen NAT-Subnetzpräfix |
| Transparent | Statisch oder dynamisch (mit externem DHCP-Server) IP-Verteilung und Zuweisung von IP-Adressen innerhalb des Container-Host-Netzwerkpräfix |
| Überlagerung | Dynamische IP-Zuweisung vom Docker-Modul-Schwarmmodus, verwaltet Präfixe und Zuweisung über HNS |
| L2Bridge | Statische IP-Zuweisung und Zuweisung von IP-Adressen im Netzwerk Präfix des Container Hosts (kann auch über HNS zugewiesen werden) |
| L2Tunnel | Nur Azure – dynamische IP-Verteilung und Zuweisung von Plug-In |

### <a name="service-discovery"></a>Dienstermittlung

Die Dienstermittlung wird nur für bestimmte Windows-Netzwerktreiber unterstützt.

|  | Lokale Dienstermittlung  | Globale Dienstermittlung |
| :---: | :---------------     |  :---                |
| nat | JA | JA mit Docker EE |  
| Überlagerung | JA | Ja mit docker EE oder Kube-DNS |
| transparent | NEIN | NEIN |
| l2bridge | NEIN | Ja mit Kube-DNS |
