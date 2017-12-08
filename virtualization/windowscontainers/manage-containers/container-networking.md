---
title: Windows-Containernetzwerk
description: "Konfigurieren Sie das Netzwerk für Windows-Container."
keywords: Docker, Container
author: jmesser81
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 98feee128860885b4f62420cc6eb86d23579551b
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/08/2017
---
# <a name="windows-container-networking"></a>Windows-Containernetzwerk
> ***Weitere Informationen für allgemeine Docker-Netzwerkbefehle, ‑Optionen und ‑Syntax finden Sie unter [Docker-Containernetzwerk](https://docs.docker.com/engine/userguide/networking/).*** Mit Ausnahme der in diesem Dokument beschriebenen Fälle werden alle Docker-Netzwerkbefehle unter Windows mit der gleichen Syntax wie unter Linux unterstützt. Beachten Sie jedoch, dass die Netzwerkstapel sich in Windows und Linux unterscheiden und daher einige Linux-Netzwerkbefehle (z.B. ifconfig) unter Windows nicht unterstützt werden.

## <a name="basic-networking-architecture"></a>Grundlegende Netzwerkarchitektur
Dieses Thema enthält eine Übersicht daüber, wie Docker Netzwerke unter Windows erstellt und verwaltet. Windows-Container funktionieren in Bezug auf Netzwerke ähnlich wie virtuelle Computer. Jeder Container verfügt über einen virtuellen Netzwerkadapter (vNIC). der mit einem virtuellen Hyper-V-Switch (vSwitch) verbunden ist. Windows unterstützt fünf verschiedene Netzwerktreiber oder Modi, die über Docker erstellt werden können: *nat*, *overlay*, *transparent*, *l2bridge* und *l2tunnel*. Sie sollten den Netzwerktreiber auswählen, der Ihren Bedürfnissen im Hinblick auf Ihre physische Netzwerkinfrastruktur und die Netzwerkanforderungen (Netzwerk mit einem oder mehreren Hosts) am besten gerecht wird.

<figure>
  <img src="media/windowsnetworkstack-simple.png">
</figure>  

Wenn die Docker-Engine das erste Mal ausgeführt wird, wird ein standardmäßiges NAT-Netzwerk erstellt ('nat'), das einen internen vSwitch und eine Windows-Komponente mit dem Namen `WinNAT` verwendet. Falls vorhandene externe vSwitches auf dem Host existieren, die über PowerShell oder den Hyper-V-Manager erstellt wurden, stehen diese ebenfalls über den *transparenten* Netzwerktreiber für Docker zur Verfügung und werden dann angezeigt, wenn Sie den Befehl ``docker network ls`` ausführen.  

<figure>
  <img src="media/docker-network-ls.png">
</figure>

> - Ein ***interner*** vSwitch ist ein Switch, der nicht direkt mit einem Netzwerkadapter auf dem Containerhost verbunden ist 

> - Ein ***externer*** vSwitch ist ein Switch, der _direkt_ mit einem Netzwerkadapter auf dem Containerhost verbunden ist  

<figure>
  <img src="media/get-vmswitch.png">
</figure>

Das NAT-Netzwerk ist das Standardnetzwerk bei Containern, die unter Windows ausgeführt werden. Container, die unter Windows ohne Flags oder Argumente zur Implementierung bestimmter Konfigurationen ausgeführt werden, werden an das standardmäßige NAT-Netzwerk angeschlossen und einer IP-Adresse aus dem NAT-Netzwerk aus dessen internen Präfix-IP-Bereich zugewiesen. Der verwendete interne IP-Standardpräfix für NAT ist 172.16.0.0/16. 


## <a name="windows-container-network-drivers"></a>Windows-Container-Netzwerktreiber  

Zusätzlich zur Nutzung des standardmäßigen NAT-Netzwerks, das von Docker unter Windows erstellt wurde, können Benutzer benutzerdefinierte Containernetzwerke festlegen. Benutzerdefinierte Netzwerke können über den CLI Docker-Befehl [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) erstellt werden. Unter Windows stehen folgende Netzwerktreibertypen zur Verfügung:

- **nat** – einem Netzwerk hinzugefügte Container, die mit einem NAT-Treiber erstellt wurden und eine vom Benutzer angegebene (``--subnet``) IP-Präfix-IP-Adresse enthalten. Portweiterleitung/-zuordnung vom Containerhost zu Containerendpunkten wird unterstützt.
> Hinweis: Mit dem Windows 10 Creators Update werden jetzt mehrere NAT-Netzwerke unterstützt. 

- **transparent** – einem Netzwerk hinzugefügte Container, die mit einem NAT-Treiber erstellt wurden und direkt mit dem physischen Netzwerk verbunden werden. IPs aus dem physischen Netzwerk können mithilfe eines externen DHCP-Servers statisch (erfordert eine benutzerdefinierte ``--subnet``-Option) oder dynamisch zugewiesen werden. 

- **overlay** - __Neu!__  Wenn das Docker-Modul im [Schwarmmodus](./swarm-mode.md) ausgeführt wird, können Container, die mit einem Überlagerungsnetzwerks verbunden sind mit anderen, an dasselbe Netzwerk angeschlossen Containern, über mehrere Containerhosts kommunizieren. Jedes Überlagerungsnetzwerk, das für einen Schwarmcluster erstellt wird, wird mit einem eigenen IP-Subnetz erstellt, das durch ein privates IP-Präfix definiert ist. Der Überlagerungsnetzwerktreiber verwendet VXLAN Kapselung.
> Erfordert Windows Server2016 mit [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) oder Windows10 Creators Update 

- **l2bridge** - einem Netzwerk hinzugefügte Container, die mit einem 'l2bridge'-Treiber erstellt wurden und sich im gleichen IP-Subnetz wie der Containerhost befinden. Die IP-Adressen müssen statisch aus dem gleichen Präfix wie der Containerhost zugewiesen werden. Alle Containerendpunkte auf dem Host verfügen aufgrund der Layer-2-Adressübersetzung beim Eingang und -Ausgang über dieselbe MAC-Adresse (Umschreiben der MAC-Adresse).
> Erfordert Windows Server2016 oder Windows10 Creators Update

- **l2tunnel** - _Dieser Treiber sollte nur in einem Microsoft-Cloudstapel verwendet werden._

> Informationen dazu, wie Sie Containerendpunkte mithilfe des Microsoft-SDN-Stapels mit einem virtuellen Overlaynetzwerk verbinden, finden Sie im Thema [Verbinden von Containern mit einem virtuellen Netzwerk](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network).

> Mit dem Windows 10 Creators Update wurde die Plattformunterstützung eingeführt, um einem aktiven Container einen neuen Containerendpunkt hinzuzufügen (z.B. "hot-add"). Dies hebt eine End-to-End [ausstehende Docker-Pull-Anforderung](https://github.com/docker/libnetwork/pull/1661) hervor.

## <a name="network-topologies-and-ipam"></a>Netzwerktopologien und IPAM
Die folgende Tabelle zeigt, wie die Netzwerkkonnektivität für interne (Container-zu-Container) und externe Verbindungen für jeden Netzwerktreiber bereitgestellt wird.

<figure>
  <img src="media/network-modes-table.png">
</figure>

### <a name="ipam"></a>IPAM 
IP-Adressen werden von jedem Netzwerktreiber unterschiedlich belegt und diesem zugewiesen. Windows verwendet den Host-Netzwerkdienst (HNS), um dem NAT-Treiber IPAM bereitzustellen und arbeitet mit dem Docker-Schwarmmodus (interner KVS), um IPAM für die Überlagerung bereitzustellen. Alle anderen Netzwerktreiber verwenden eine externe IPAM.

<figure>
  <img src="media/ipam.png">
</figure>

# <a name="details-on-windows-container-networking"></a>Informationen über das Containernetzwerk

## <a name="isolation-namespace-with-network-compartments"></a>Isolation (Namespace) mit Netzwerkdepots
Jeder Containerendpunkt befindet sich in seinem eigenen __Netzwerkdepot__, der mit dem Netzwerknamespaces in Linux vergleichbar ist. Der Verwaltungshost vNIC und die Host-Netzwerkstapel befinden sich im Standard-Netzwerkdepot. Um eine Netzwerkisolation der Container zu erzwingen, die sich auf dem gleichen Host befinden, wird für jeden Windows Server- und Hyper-V-Container ein Netzwerkbereich erstellt, in dem der Netzwerkadapter für den Container installiert wird. Windows Server-Container verwenden eine Host-vNIC für die Verbindung mit dem virtuellen Switch. Hyper-V-Container verwenden eine synthetische VM-NIC (nicht für die Utility-VM verfügbar gemacht) für die Verbindung mit dem virtuellen Switch. 

<figure>
  <img src="media/network-compartment-visual.png">
</figure>

```powershell 
Get-NetCompartment
```

## <a name="windows-firewall-security"></a>Windows-Firewall-Sicherheit

Die Windows-Firewall wird verwendet, um die Netzwerksicherheit über Port-ACLs zu erzwingen.

> Hinweis: Standardmäßig verfügen alle Container-Endpunkte, die mit einem Überlagerungsnetzwerks verbunden sind, eine erstellte „Alle zulassen”-Regel.   

<figure>
  <img src="media/windows-firewall-containers.png">
</figure>

## <a name="container-network-management-with-host-network-service"></a>Verwaltung des Containermetzwerks mit dem Host Network Service (HNS)

Die folgende Abbildungzeigt, wie der Host Network Service (HNS) mit dem Host Compute Service (HCS) zusammenarbeitet, um Container zu erstellen und einem Netzwerk Endpunkte hinzuzufügen. 

<figure>
  <img src="media/HNS-Management-Stack.png">
</figure>

# <a name="advanced-network-options-in-windows"></a>Erweiterte Netzwerkoptionen für Windows
Es werden verschiedene Netzwerktreiberoptionen unterstützt, um die Vorteile der Windows-spezifischen Funktionen und Features zu nutzen. 

## <a name="switch-embedded-teaming-with-docker-networks"></a>Switch Embedded Teaming und Docker-Netzwerke

> Gilt für alle Netzwerktreiber 

Nutzen Sie [Switch Embedded Teaming](https://technet.microsoft.com/en-us/windows-server-docs/networking/technologies/hyper-v-virtual-switch/rdma-and-switch-embedded-teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set), wenn Sie Containerhost-Netzwerke für die Verwendung von Docker durch Angabe mehrerer Netzwerkadapter (durch Kommas getrennt) mit der `-o com.docker.network.windowsshim.interface`-Option erstellen. 

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## <a name="set-the-vlan-id-for-a-network"></a>Legen Sie die VLAN-ID für ein Netzwerk fest

> Gilt für transparente und l2bridge-Netzwerktreiber 

Wenn Sie eine VLAN-ID für ein Netzwerk festlegen möchten, verwenden Sie die Option `-o com.docker.network.windowsshim.vlanid=<VLAN ID>`für den Befehl `docker network create`. Sie können z.B. folgenden Befehl verwenden, um ein transparentes Netzwerk mit einer ID VLAN 11 zu erstellen:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
Wenn Sie die VLAN-ID für ein Netzwerk festlegen, wird damit die VLAN-Isolation für alle Containerendpunkte festgelegt, die mit dem Netzwerk verbunden sind.

> Stellen Sie sicher, dass sich der Hostnetzwerkadapter (physisch) im Trunk-Modus befindet, damit alle markierten (tagged) Datenpakte vom vSwitch mit dem vNIC-Port (Containerendpunkt) im Zugriffsmodus auf dem richtigen VLAN verarbeitet werden können.

## <a name="specify-the-name-of-a-network-to-the-hns-service"></a>Festlegen des Netzwerknamens für den HNS-Dienst

> Gilt für alle Netzwerktreiber 

Wenn Sie mit `docker network create` ein Containernetzwerk erstellen, wird der Name des Netzwerks, den Sie bereitstellen, normalerweise vom Docker-Dienst verwendet, aber nicht vom HNS-Dienst. Wenn Sie ein Netzwerk erstellen, können Sie den Namen, den es vom HNS-Dienst erhält, mit der Option `-o com.docker.network.windowsshim.networkname=<network name>` in der Anweisung `docker network create` angeben. Beispielsweise können Sie den folgenden Befehl verwenden, um ein transparentes Netzwerk mit einem Namen zu erstellen, den der HNS-Dienst verwenden soll:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## <a name="bind-a-network-to-a-specific-network-interface"></a>Binden eines Netzwerks an eine bestimmte Netzwerkschnittstelle

> Gilt für alle Netzwerktreiber mit Ausnahme von "NAT"  

Verwenden Sie zum Binden eines Netzwerks (angeschlossen über den virtuellen Hyper-V-Switch) an eine bestimmte Netzwerkschnittstelle die Option `-o com.docker.network.windowsshim.interface=<Interface>` für den Befehl `docker network create`. Sie können z.B. die folgende Anweisung verwenden, um ein transparentes Netzwerk zu erstellen, das an die Netzwerkschnittstelle „Ethernet 2“ angeschlossen ist:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> Hinweis: Der Wert für *com.docker.network.windowsshim.interface* ist der *Name* des Netzwerkadapters und kann mit folgender Anweisung gefunden werden:

>```
PS C:\> Get-NetAdapter
```
## Specify the DNS Suffix and/or the DNS Servers of a Network

> Applies to all network drivers 

Use the option, `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` to specify the DNS suffix of a network, and the option, `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` to specify the DNS servers of a network. For example, you might use the following command to set the DNS suffix of a network to "example.com" and the DNS servers of a network to 4.4.4.4 and 8.8.8.8:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## VFP

The Virtual Filtering Platform (VFP) extension is a Hyper-V virtual switch, forwarding extension used to enforce network policy and manipulate packets. For instance, VFP is used by the 'overlay' network driver to perform VXLAN encapsulation and by the 'l2bridge' driver to perform MAC re-write on ingresss and egress. The VFP extension is only present on Windows Server 2016 and Windows 10 Creators Update. To check and see if this is running correctly a user run two commands:

```powershell
Get-Service vfpext

# This should indicate the extension is Running: True 
Get-VMSwitchExtension  -VMSwitchName <vSwitch Name> -Name "Microsoft Azure VFP Switch Extension"
```

## <a name="tips--insights"></a>Tipps und Einblicke
Hier ist eine Liste mit nützlichen Tipps und Einblicken, die von den häufig von uns gehörten Fragen der Community über Windows-Container-Netzwerke inspiriert sind...

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>HNS erfordert, dass IPv6 auf den Containerhostgeräten aktiviert ist 
Als Teil des [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) ist es für HNS erforderlich, dass IPv6 auf den Windows-Containerhosts aktiviert ist. Wenn wie unten beschrieben ein Fehler auftritt, ist IPv6 wahrscheinlich auf dem Hostcomputer deaktiviert.
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
Wir arbeiten an Plattform Änderungen, um dieses Problem automatisch zu erkennen/zu vermeiden. Derzeit kann die folgende Problemumgehung verwendet werden, um sicherzustellen, dass IPv6 auf dem Hostcomputer aktiviert ist:

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-cehttpswwwdockercomcommunity-edition-instead-of-hns-internal-vswitch"></a>Moby Linux-VMs verwendet DockerNAT-Switch mit Docker für Windows (ein Produkt der [Docker CE](https://www.docker.com/community-edition)) anstelle des internen HNS vSwitches 
Docker für Windows (der Windows-Treiber für die Docker CE-Engine) auf Windows10 verwendet einen interne vSwitch mit dem Namen "DockerNAT", um Moby Linux-VMs mit dem Containerhost zu verbinden. Entwickler, die Moby Linux VMs unter Windows verwenden sollten beachten, dass ihre Hosts den DockerNAT-vSwitch anstelle des vSwitches verwenden, der vom HNS-Dienst erstellt wird (dies ist der Standardswitch für Windows-Container). 

#### <a name="to-use-dhcp-for-ip-assignment-on-a-virtual-container-host-enable-macaddressspoofing"></a>Aktivieren Sie MACAddressSpoofing, um DHCP für die IP-Adresszuweisung auf einem virtuellen Containerhost zu verwenden. 
Wenn der Containerhost virtualisiert ist und Sie DHCP für die IP-Zuweisung verwenden möchten, müssen Sie MACAddressSpoofing im Netzwerkadapter der virtuellen Computer aktivieren. Andernfalls blockiert der Hyper-V-Host Netzwerkdatenverkehr von den Containern auf dem virtuellen Computer mit mehreren MAC-Adressen. Aktivieren Sie MACAddressSpoofing mit folgendem PowerShell-Befehl:
```
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
#### <a name="creating-multiple-transparent-networks-on-a-single-container-host"></a>Erstellen mehrerer transparenter Netzwerke auf einem einzelnen Containerhost
Wenn Sie mehr als ein transparentes Netzwerk erstellen möchten, müssen Sie angeben, an welchen (virtuellen) Netzwerkadapter der Hyper-V Virtual Switch gebunden werden soll. Um die Benutzeroberfläche für ein Netzwerk anzugeben, verwenden Sie folgende Syntax:
```
# General syntax:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface=<INTERFACE NAME> <NETWORK NAME> 

# Example:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" myTransparent2
```

#### <a name="remember-to-specify---subnet-and---gateway-when-using-static-ip-assignment"></a>Denken Sie daran, bei der Verwendung einer statischen IP-Zuordnung *‑‑Subnetz* und *‑‑Gateway* anzugeben
Wenn Sie die statische IP-Adresszuweisung verwenden, müssen Sie zunächst sicherstellen, dass die Parameter *--subnet* und *--gateway* beim Erstellen des Netzwerks festgelegt sind. Subnetz- und Gateway-IP-Adresse sollten den Netzwerkeinstellungen für den Containerhost (d. h. für das physische Netzwerk) entsprechen. Hier sehen Sie, wie Sie z.B., wie Sie ein transparentes Netzwerk erstellen und dann einen Endpunkt auf diesem Netzwerk mithilfe der statischen IP-Adresszuweisung ausführen können:

```
# Example: Create a transparent network using static IP assignment
# A network create command for a transparent container network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 MyTransparentNet
# Run a container attached to MyTransparentNet
C:\> docker run -it --network=MyTransparentNet --ip=10.123.174.105 windowsservercore cmd
```

#### <a name="dhcp-ip-assignment-not-supported-with-l2bridge-networks"></a>Die DHCP-IP-Adresszuweisung wird nicht von L2Bridge-Netzwerken unterstützt
Es wird nur eine statische IP-Adresszuweisung mit Container-Netzwerken unterstützt, die mithilfe des l2bridge-Treibers erstellt wurden. Wie bereits erwähn: denken Sie daran, die *‑‑Subnetz* und *‑‑Gateway*-Parameter zu verwenden, um ein Netzwerk zu erstellen, das für statische IP-Adresszuweisungen konfiguriert ist.

#### <a name="networks-that-leverage-external-vswitch-must-each-have-their-own-network-adapter"></a>Netzwerke, die externe vSwitches nutzen müssen jeweils über eigene Netzwerkadapter verfügen.
Wenn mehrere Netzwerke, die einen externen vSwitch für die Konnektivität verwenden (z.B. Transparent, L2-Brücke, L2 Transparent), auf dem gleichen Containerhost erstellt werden, muss jedes über einen eigenen Netzwerkadapter verfügen. 

#### <a name="ip-assignment-on-stopped-vs-running-containers"></a>IP-Adresszuweisung auf Containern im angehaltenen Zustand im Vergleich zu ausgeführten Containern
Die statische IP-Zuweisung erfolgt direkt im Netzwerkadapter des Containers und darf nur dann durchgeführt werden, wenn sich der Container im Zustand BEENDET befindet. Das Hinzufügen von Containernetzwerkadaptern im laufenden Betrieb (Hot-Add) sowie Änderungen am Netzwerkstapel werden während der Containerausführung nicht unterstützt (auf Windows Server 2016).
> Hinweis: Dies wird im Windows 10 Creators Update geändert, da die Plattform jetzt "Live-Hinzufügen" unterstützt Diese Funktion wird E2E hervorheben, nachdem die [ausstehende Docker Pull-Anforderung](https://github.com/docker/libnetwork/pull/1661) zusammengeführt wird

#### <a name="existing-vswitch-not-visible-to-docker-can-block-transparent-network-creation"></a>Ein bereits vorhandener vSwitch (nicht für Docker sichtbar) kann die Erstellung eines transparenten Netzwerks blockieren
Sollten Fehler beim Erstellen eines transparenten Netzwerks auftreten, ist es möglich, dass ein externer vSwitch in Ihrem System existiert, der nicht automatisch von Docker erkannt wurde, und der verhindert, dass das transparente Netzwerk an den externen Netzwerkadapter Ihres Containerhosts gebunden wird. 

Wenn Sie ein transparentes Netzwerk erstellen, erstellt Docker einen externen vSwitch für dieses Netzwerk und versucht dann, den Switch an einen (externen) Netzwerkadapter zu binden, bei dem es sich um einen VM-Netzwerkadapter oder den physischen Netzwerkadapter handeln kann. Wenn bereits ein vSwitch auf dem Containerhost erstellt wurde, *und von Docker erkannt wird*, wird das Docker-Modul unter Windows diesen Switch verwenden, anstatt einen neuen zu erstellen. Wenn jedoch der vSwitch Out-of-Band erstellt wurde (z.B. auf dem Containerhost mithilfe des Hyper-V-Managers oder PowerShell) und noch nicht von Docker erkannt wird, versucht das Docker-Modul unter Windows einen neuen vSwitch zu erstellen, kann den neuen Switch dann jedoch nicht mit dem externen Netzwerkadapter des Containerhosts verbinden (da der Netzwerkadapter bereits mit dem Switch verbunden ist, der Out-of-Band erstellt wurde).

Das Problem taucht z.B. auf, wenn Sie zuerst einen neuen vSwitch auf Ihrem Host erstellen würden, während der Docker-Dienst ausgeführt wird, und dann versuchen würden, ein transparentes Netzwerk zu erstellen. In diesem Fall würde Docker den von Ihnen erstellen Switch nicht erkennen und würde einen neuen vSwitch für das transparente Netzwerk erstellen.

Es gibt drei Ansätze zum Lösen dieses Problems:

* Sie können natürlich den vSwitch löschen, der Out-of-Band erstellt wurde. Daraufhin kann Docker einen neuen vSwitch erstellen und ihn dann ohne Probleme mit dem Netzwerkadapter des Hosts verbinden. Bevor Sie sich für diesen Ansatz entscheiden, stellen Sie sicher dass Ihr Out-of-Band-vSwitch nicht von anderen Diensten verwendet wird (z.B. Hyper-V).
* Wenn Sie sich alternativ dafür entscheiden, einen externen Out-of-Band-vSwitch zu verwenden, starten Sie Docker und die HNS-Dienste neu, um *den Switch für Docker erkennbar zu machen.*
```
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* Eine andere Möglichkeit ist die Verwendung der Option „-o com.docker.network.windowsshim.interface“, um den externen vSwitch des transparenten Netzworks an einen bestimmten Netzwerkadapter zu binden, der noch nicht auf dem Containerhost in Gebrauch ist (z.B. ein anderer Netzwerkadapter als der, der vom vSwitch verwendet wird, der Out-of-Band erstellt wurde). Die „-o“-Option wird weiter oben im Abschnitt [Transparentes Netzwerk](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network) beschrieben.


## <a name="unsupported-features-and-network-options"></a>Nicht unterstützte Features und Netzwerkoptionen 

Die folgenden Netzwerkoptionen werden in Windows nicht unterstützt und können nicht an ``docker run`` übergeben werden:
 * Containerverknüpfung (z.B. ``--link``) – _Alternative ist von Dienstermittlung abhängig_
 * IPv6-Adressen (z.B. ``--ip6``)
 * DNS-Optionen (z.B. ``--dns-option``)
 * Mehrere Domains für die DNS-Suche (z.B. ``--dns-search``)
 
Die folgenden Netzwerkoptionen und Funktionen werden in Windows nicht unterstützt und können nicht an ``docker network create`` übergeben werden:
 * --aux-address
 * --internal
 * --ip-range
 * --ipam-driver
 * --ipam-opt
 * --ipv6 

Die folgenden Netzwerkoptionen werden auf Docker-Services nicht unterstützt
* Verschlüsselung auf der Datenschicht (z.B. ``--opt encrypted``) 


## <a name="windows-server-2016-work-arounds"></a>Windows Server2016-Problemumgehungen 

Obwohl wir auch weiterhin neue Funktionen hinzufügen und die Entwicklung vorantreiben, können einige dieser Funktionen nicht auf ältere Plattformen rückportiert werden. Der beste Aktionsplan ist das neueste Updates für Windows10 und Windows Server.  Im folgenden Abschnitt werden einige Problemumgehungen und die Vorsichtsmaßnahmen erklärt, die für ältere Versionen von Windows10 (d.h. vor dem 1704 Creators Update) und Windows Server2016 gelten.

### <a name="multiple-nat-networks-on-ws2016-container-host"></a>Mehrere NAT-Netzwerke auf WS2016-Containerhost

Die Partitionierung für neue NAT-Netzwerke muss unter dem größeren internen NAT-Netzwerkpräfix erfolgen. Das Präfix kann durch die Ausführung des folgenden PowerShell-Befehls gefunden werden und bezieht sich auf das Feld „InternalIPInterfaceAddressPrefix“.

```
PS C:\> Get-NetNAT
```

Angenommen, das interne NAT-Netzwerkpräfix des Hosts ist 172.16.0.0/16. In diesem Fall kann Docker verwendet werden, um zusätzliche NAT-Netzwerke zu erstellen *solange sie eine Teilmenge des Präfix 172.16.0.0/16 sind.* Beispielsweise können zwei NAT-Netzwerke mit den IP-Präfixen 172.16.1.0/24 (Gateway, 172.16.1.1) und 172.16.2.0/24 (Gateway, 172.16.2.1) erstellt werden.

```
C:\> docker network create -d nat --subnet=172.16.1.0/24 --gateway=172.16.1.1 CustomNat1
C:\> docker network create -d nat --subnet=172.16.2.0/24 --gateway=172.16.1.1 CustomNat2
```

Die neu erstellten Netzwerke können mithilfe folgender Optionen aufgelistet werden:
```
C:\> docker network ls
```

### <a name="docker-compose"></a>Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) kann verwendet werden, um Containernetzwerke neben den Containern/Diensten zu definieren und konfigurieren, die diese Netzwerke verwenden werden. Der Compose „Netzwerke“-Schlüssel wird als Schlüssel der obersten Ebene verwendet, um die Netzwerke zu definieren, mit denen die Container verbunden werden. Die folgende Syntax definiert beispielsweise das bereits vorhandene NAT-Netzwerk, das von Docker als „Standardnetzwerk“ für alle Container/Dienste erstellt wurde, die in einer bestimmten Compose-Datei definiert sind.

```
networks:
 default:
  external:
   name: "nat"
```

Die folgende Syntax kann auf ähnliche Weise zum Definieren eines benutzerdefinierten NAT-Netzwerks verwendet werden.

> Hinweis: Das „benutzerdefinierte NAT-Netzwerk“, das im nachfolgenden Beispiel definiert ist, ist als Partition des bereits vorhandenen NAT-internen Präfixes des Containerhosts definiert. Weiteren Kontext finden Sie im Abschnitt „Mehrere NAT-Netzwerke“ weiter oben.

```
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.16.3.0/24
```

Weitere Informationen über das Definieren/Konfigurieren von Containernetzwerken mit Docker Compose finden Sie unter [Compose File reference (Referenz zur Compose-Datei)](https://docs.docker.com/compose/compose-file/).

### <a name="service-discovery"></a>Dienstermittlung
Die Dienstermittlung wird nur für bestimmte Windows-Netzwerktreiber unterstützt.

|  | Lokale Dienstermittlung  | Globale Dienstermittlung |
| :---: | :---------------     |  :---                |
| nat | JA | Nicht verfügbar |  
| overlay | JA | JA |
| transparent | NEIN | NEIN |
| l2bridge | NEIN | NEIN |


