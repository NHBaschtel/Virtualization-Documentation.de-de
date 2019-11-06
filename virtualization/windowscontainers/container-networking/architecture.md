---
title: Windows-Container Netzwerke
description: Einfache Einführung in die Architektur der Windows-Container-Netzwerke.
keywords: Docker, Container
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 8d2ddb80aa05b511dbc8c9532654b18956e340da
ms.sourcegitcommit: 7fd95333bd7fd2ef3627b0b5c558067e0bd0e09f
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/05/2019
ms.locfileid: "10276515"
---
# <a name="windows-container-networking"></a>Windows-Container Netzwerke

>[!IMPORTANT]
>Bitte verweisen Sie auf das [Dock Container-Netzwerk](https://docs.docker.com/engine/userguide/networking/) für allgemeine docker-Netzwerk Befehle,-Optionen und-Syntax. * * * mit Ausnahme von Fällen, die in nicht [unterstützten Features und Netzwerkoptionen](#unsupported-features-and-network-options)beschrieben sind, werden alle docker-Netzwerk Befehle unterstützt unter Windows mit der gleichen Syntax wie unter Linux. Die Windows-und Linux-Netzwerkstapel sind jedoch unterschiedlich, und als solche werden Sie feststellen, dass einige Linux-Netzwerk Befehle (beispielsweise ifconfig) unter Windows nicht unterstützt werden.

## <a name="basic-networking-architecture"></a>Grundlegende Netzwerkarchitektur

Dieses Thema enthält eine Übersicht darüber, wie Dockerhost Netzwerke unter Windows erstellt und verwaltet. Windows-Container funktionieren in Bezug auf Netzwerke ähnlich wie virtuelle Computer. Jeder Container verfügt über einen virtuellen Netzwerkadapter (vNIC). der mit einem virtuellen Hyper-V-Switch (vSwitch) verbunden ist. Windows unterstützt fünf verschiedene [Netzwerktreiber oder Modi](./network-drivers-topologies.md), die über Docker erstellt werden können: *nat*, *overlay*, *transparent*, *l2bridge*, and *l2tunnel*. Sie sollten den Netzwerktreiber auswählen, der Ihren Bedürfnissen im Hinblick auf Ihre physische Netzwerkinfrastruktur und die Netzwerkanforderungen (Netzwerk mit einem oder mehreren Hosts) am besten gerecht wird.

![Text](media/windowsnetworkstack-simple.png)

Wenn die Docker-Engine das erste Mal ausgeführt wird, wird ein standardmäßiges NAT-Netzwerk erstellt ('nat'), das einen internen vSwitch und eine Windows-Komponente mit dem Namen `WinNAT` verwendet. Falls vorhandene externe vSwitches auf dem Host existieren, die über PowerShell oder den Hyper-V-Manager erstellt wurden, stehen diese ebenfalls über den *transparenten* Netzwerktreiber für Docker zur Verfügung und werden dann angezeigt, wenn Sie den Befehl ``docker network ls`` ausführen.  

![Text](media/docker-network-ls.png)

- Eine **interne** Vswitch ist eine, die nicht direkt mit einem Netzwerkadapter auf dem Container Host verbunden ist.
- Eine **externe** Vswitch ist eine direkt mit einem Netzwerkadapter auf dem Container Host verbunden.

![Text](media/get-vmswitch.png)

Das NAT-Netzwerk ist das Standardnetzwerk bei Containern, die unter Windows ausgeführt werden. Container, die unter Windows ohne Flags oder Argumente zur Implementierung bestimmter Konfigurationen ausgeführt werden, werden an das standardmäßige NAT-Netzwerk angeschlossen und einer IP-Adresse aus dem NAT-Netzwerk aus dessen internen Präfix-IP-Bereich zugewiesen. Der verwendete interne IP-Standardpräfix für NAT ist 172.16.0.0/16. 

## <a name="container-network-management-with-host-network-service"></a>Verwaltung des Containermetzwerks mit dem Host Network Service (HNS)

Der Host Network Service (HNS) mit der Host Compute Service (HCS) arbeiten zusammen, um Container zu erstellen und einem Netzwerk Endpunkte hinzuzufügen.

### <a name="network-creation"></a>Erstellen eines Netzwerks

- HNS erstellt einen neuen virtuellen Switch auf jedem Netzwerk
- HNS erstellt NAT und IP-Adresspools nach Bedarf

### <a name="endpoint-creation"></a>Endpunkt-Erstellung

- HNS erstellt Netzwerk-Namespace pro Containerendpunkt
- HNS/HCS platziert v(m)NIC in den Netzwerk-Namespace
- HNS erstellt (vSwitch)-Ports
- HNS weist Endpunkt IP-Adressen, DNS-Informationen, Routen usw. (unterliegen Netzwerkmodus) zu

### <a name="policy-creation"></a>Richtlinienerstellung

- NAT-Standardnetzwerk: HNS erstellt Regeln zur WinNAT-Portweiterleitung und Zuordnungen mit entsprechenden Windows-Firewall Zulassungsregeln
- Alle anderen Netzwerke: HNS nutzt die VFP-Plattform für die Erstellung von Richtlinien
    - Dazu gehören: Lastenausgleich, ACLs, Kapselung usw.
    - Schauen Sie sich die [hier](https://docs.microsoft.com/en-us/windows-server/networking/technologies/hcn/hcn-top) veröffentlichten HNS-APIs und-Schemata an.

![Text](media/HNS-Management-Stack.png)

## <a name="unsupported-features-and-network-options"></a>Nicht unterstützte Features und Netzwerkoptionen

Die folgenden Netzwerkoptionen werden unter Windows derzeit **nicht** unterstützt:

- Windows-Container, die an l2bridge-, NAT-und Overlay-Netzwerke angefügt sind, unterstützen keine Kommunikation über den IPv6-Stapel.
- Verschlüsselte Container Kommunikation über IPSec.
- HTTP-Proxy Unterstützung für Container.
- Anfügen von Endpunkten an die Ausführung in der Hyper-V-Isolierung (Hot-Add).
- Netzwerk für virtualisierte Azure-Infrastruktur über den transparenten Netzwerktreiber.

| Befehl        | Nicht unterstützte Option   |
|---------------|:--------------------:|
| ``docker run``|   ``--ip6``, ``--dns-option`` |
| ``docker network create``| ``--aux-address``, ``--internal``, ``--ip-range``, ``--ipam-driver``, ``--ipam-opt``, ``--ipv6``, ``--opt encrypted`` |
