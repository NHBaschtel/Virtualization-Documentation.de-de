---
title: Windows-Containernetzwerk
description: "Konfigurieren Sie das Netzwerk für Windows-Container."
keywords: Docker, Container
author: jmesser81
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
translationtype: Human Translation
ms.sourcegitcommit: f489d3e6f98fd77739a2016813506be6962b34d1
ms.openlocfilehash: 499788666f306494c894b2e82f65ab68c9fc295a

---

# Containernetzwerk

Windows-Container funktionieren in Bezug auf Netzwerke ähnlich wie virtuelle Computer. Jeder Container verfügt über einen virtuellen Netzwerkadapter (vNIC), der mit einem virtuellen Switch (vSwitch) verbunden ist, über den eingehender und ausgehender Datenverkehr weitergeleitet wird. Um eine Trennung der Container zu erzwingen, die sich auf dem gleichen Host befinden, wird für jeden Windows Server- und Hyper-V-Container ein Netzwerkbereich erstellt, in dem der Netzwerkadapter für den Container installiert wird. Windows Server-Container verwenden eine Host-vNIC für die Verbindung mit dem virtuellen Switch. Hyper-V-Container verwenden eine synthetische VM-NIC (nicht für die Utility-VM verfügbar gemacht) für die Verbindung mit dem virtuellen Switch.

Windows-Container unterstützen vier verschiedene Netzwerktreiber oder -modi: *nat*, *transparent*, *l2bridge* und *l2tunnel*. Sie sollten den Netzwerkmodus auswählen, der Ihren Bedürfnissen im Hinblick auf Ihre physische Netzwerkinfrastruktur und die Netzwerkanforderungen (Netzwerk mit einem oder mehreren Hosts) am besten gerecht wird. 

Beim ersten Ausführen des dockerd-Diensts wird vom Docker-Modul standardmäßig ein NAT-Netzwerk erstellt. Das erstellte interne IP-Standardpräfix ist 172.16.0.0/12. Containerendpunkte werden automatisch an dieses Standardnetzwerk angefügt und einer IP-Adresse aus dessen internen Präfix zugewiesen.

> Hinweis: Wenn sich Ihre Containerhost-IP im selben Präfix befindet, müssen Sie das NAT-interne IP-Präfix entsprechend der folgenden Beschreibung ändern.

Auf demselben Containerhost können zusätzliche Netzwerke, die einen anderen Treiber verwenden (z. B. „transparent“ „l2bridge“), erstellt werden. Die folgende Tabelle zeigt, wie die Netzwerkkonnektivität für interne (Container-zu-Container) und externe Verbindungen für jeden Modus bereitgestellt wird.

- **Netzwerkadressübersetzung (Network Address Translation, NAT)** – jeder Container erhält eine IP-Adresse aus einem internen, privaten IP-Präfix (z. B. 172.16.0.0/12). Portweiterleitung/-zuordnung vom Containerhost zu Containerendpunkten wird unterstützt.

- **Transparent** – Jeder Containerendpunkt ist direkt mit dem physischen Netzwerk verbunden. IPs aus dem physischen Netzwerk können mithilfe eines externen DHCP-Servers statisch oder dynamisch zugewiesen werden.

- **L2-Bridge** – Alle Containerendpunkte befinden sich im gleichen IP-Subnetz wie der Containerhost. Die IP-Adressen müssen statisch aus dem gleichen Präfix wie der Containerhost zugewiesen werden. Alle Containerendpunkte auf dem Host verfügen aufgrund der Layer-2-Adressübersetzung über dieselbe MAC-Adresse.

- **L2-Tunnel** - _Dieser Modus sollte nur in einem Microsoft-Cloudstapel verwendet werden._

> Informationen dazu, wie Sie Containerendpunkte mithilfe des Microsoft-SDN-Stapels mit einem virtuellen Overlaynetzwerk verbinden, finden Sie im Thema [Verbinden von Containern mit einem virtuellen Netzwerk](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network).

## Einzelknoten

|  | Container-Container | Container-Extern |
| :---: | :---------------     |  :---                |
| nat | Überbrückte Verbindung über virtuelle Hyper-V-Switches | weitergeleitet durch WinNAT mit aktivierter Adressübersetzung | 
| transparent | Überbrückte Verbindung über virtuelle Hyper-V-Switches | direkter Zugriff auf das physische Netzwerk | 
| l2bridge | Überbrückte Verbindung über virtuelle Hyper-V-Switches|  Zugriff auf das physische Netzwerk mit MAC-Adressübersetzung|  



## Mit mehreren Knoten

|  | Container-Container | Container-Extern |
| :---: | :----       | :---------- |
| nat | muss auf IP und Port des externen Containerhosts verweisen; weitergeleitet durch WinNAT mit aktivierter Adressübersetzung | muss auf den externen Host verweisen; weitergeleitet durch WinNAT mit aktivierter Adressübersetzung | 
| transparent | muss direkt auf den Container-IP-Endpunkt verweisen | direkter Zugriff auf das physische Netzwerk | 
| l2bridge | muss direkt auf den Container-IP-Endpunkt verweisen| Zugriff auf das physische Netzwerk mit MAC-Adressübersetzung| 


## Erstellen eines Netzwerks 

### (Standard) NAT-Netzwerk

Das Docker-Modul unter Windows erstellt ein „NAT“-Standardnetzwerk (dessen Docker „nat“ heißt) mit IP-Präfix 172.16.0.0/12. Wenn ein Benutzer ein NAT-Netzwerk mit einem bestimmten IP-Präfix erstellen möchte, kann er dazu eine von zwei Optionen in der Docker-Konfigurationsdatei „daemon.json“ ändern (diese befindet sich unter „C:\ProgramData\Docker\config\daemon.json“. Ist sie noch nicht vorhanden, muss sie erstellt werden).
 1. Verwenden der Option _„fixed-cidr“: „< IP Prefix > / Mask“_. Dadurch wird das NAT-Standardnetzwerk erstellt, und IP-Präfix und Übereinstimmung werden festgelegt.
 2. Verwenden der Option _„bridge“: „none“_. Dadurch wird kein Standardnetzwerk erstellt. Ein Benutzer kann unter Verwendung des Befehls *docker network create -d <driver>* ein benutzerdefiniertes Netzwerk mit einem beliebigen Treiber erstellen.

Vor dem Ausführen einer dieser Konfigurationsoptionen muss der Docker-Dienst zuerst beendet werden, und alle bereits vorhandenen NAT-Netzwerke müssen gelöscht werden.

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

Wenn die Option „fixed-cidr“ zur Datei „daemon.json“ hinzugefügt wird, wird vom Docker-Modul ein benutzerdefiniertes NAT-Netzwerk mit dem angegebenen benutzerdefinierten IP-Präfix und der angegebenen Maske erstellt. Wenn stattdessen die Option „Bridge:none“ hinzugefügt wird, muss das Netzwerk manuell erstellt werden.

```none
# Create a user-defined NAT network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

Standardmäßig werden Containerendpunkte mit dem NAT-Standardnetzwerk verbunden. Wenn das NAT-Netzwerk nicht erstellt wurde (da in der daemon.json-Datei „bridge:none“ angegeben wurde) oder der Zugriff auf ein anderes, benutzerdefinierte Netzwerk erforderlich ist, können Benutzer den Parameter *--network* mit Ausführungsbefehl von Docker angeben.

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### Portzuordnung

Für den Zugriff auf Anwendungen, die in mit einem NAT-Netzwerk verbundenen Container ausgeführt werden, müssen zwischen dem Containerhost und dem Containerendpunkt Portzuordnungen erstellt werden. Diese Zuordnungen müssen zum Zeitpunkt der Erstellung des Containers, oder während der Container sich im Zustand ANGEHALTEN befindet, festgelegt werden.

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Creates a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

Dynamische Portzuordnungen werden ebenfalls unterstützt, und zwar entweder unter Verwendung des Parameters „-p“ oder des Befehls EXPOSE in einer Dockerfile. Wenn nichts angegeben ist, wird auf dem Containerhost per Zufallsprinzip ein kurzlebiger Port ausgewählt, der bei Ausführung von „Docker.ps“ überprüft werden kann.

```none
C:\> docker run -itd -p 80 windowsservercore cmd

# Network services running on port TCP:80 in this container can be accessed externally on port TCP:14824
C:\> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
bbf72109b1fc        windowsservercore   "cmd"               6 seconds ago       Up 2 seconds        *0.0.0.0:14824->80/tcp*   drunk_stonebraker

# Container image specified EXPOSE 80 in Dockerfile - publish this port mapping
C:\> docker network 
```
> Ab WS2016 TP5 und ab Windows Insider Builds nach 14300 wird für alle NAT-Portzuordnungen automatisch eine Firewallregel erstellt. Diese Firewallregel gilt global für den Containerhost und ist nicht auf einen bestimmten Containerendpunkt oder Netzwerkadapter begrenzt.

Die Implementierung von Windows NAT (WinNAT) weist einige Funktionslücken auf, die in diesem Blogbeitrag behandelt werden: [WinNAT capabilities and limitations](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/) 
 1. Pro Containerhost wird nur ein internes IP-Präfix für NAT unterstützt, „mehrfache“ NAT-Netzwerke müssen also durch Partitionierung des Präfixes definiert werden (Informationen hierzu finden Sie in diesem Dokument im Abschnitt „Mehrfache NAT-Netzwerke“).
 2. Containerendpunkte sind nur vom Containerhost mithilfe der internen IP-Adressen und Ports des Containers erreichbar (Informationen darüber finden Sie mithilfe „Überprüfen von Docker-Netzwerk<CONTAINER ID>“).

Zusätzliche Netzwerke können mit anderen Treibern erstellt werden. 

> Docker-Netzwerktreiber bestehen vollständig aus Kleinbuchstaben.

### Transparentes Netzwerk

Um den transparenten Netzwerkmodus zu verwenden, erstellen Sie ein Containernetzwerk mit dem Treiber namens „transparent“. 

```none
C:\> docker network create -d transparent MyTransparentNetwork
```
> Hinweis: Sollten Fehler beim Erstellen des transparenten Netzwerks auftreten, ist es möglich, dass ein externer vSwitch in Ihrem System existiert, der nicht automatisch von Docker erkannt wurde, und der verhindert, dass das transparente Netzwerk an den externen Netzwerkadapter Ihres Containerhosts gebunden wird. Weitere Informationen finden Sie im Abschnitt „Vorhandener vSwitch blockiert die Erstellung des transparenten Netzwerks“ unter „Tipps und Einschränkungen“ in diesem Thema.

Wenn der Containerhost virtualisiert ist und Sie DHCP für die IP-Zuweisung verwenden möchten, müssen Sie MACAddressSpoofing im Netzwerkadapter der virtuellen Computer aktivieren. Andernfalls blockiert der Hyper-V-Host Netzwerkdatenverkehr von den Containern auf dem virtuellen Computer mit mehreren MAC-Adressen.

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> Wenn Sie mehr als ein transparentes Netzwerk erstellen möchten, müssen Sie angeben, mit welchem (virtuellen) Netzwerkadapter der (automatisch erstellte) virtuelle Hyper-V-Switch eine Bindung herstellen soll.

Um ein (über den virtuellen Hyper-V-Switch angeschlossenes) Netzwerk an eine bestimmte Netzwerkschnittstelle zu binden, verwenden Sie die Option *-o com.docker.network.windowsshim.interface=<Interface>*

```none
# Create a transparent network which is attached to the "Ethernet 2" network interface
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

Der Wert für *com.docker.network.windowsshim.interface* ist der *Name* des Adapters aus: 
```none
Get-NetAdapter
```

IP-Adressen für Containerendpunkte, die mit einem transparenten Netzwerk verbunden sind, können entweder statisch oder dynamisch von einem externen DHCP-Server zugewiesen werden.

Wenn Sie die statische IP-Zuweisung verwenden, müssen Sie zunächst sicherstellen, dass die Parameter *--subnet* und *--gateway* beim Erstellen des Netzwerks festgelegt sind. Subnetz- und Gateway-IP-Adresse sollten den Netzwerkeinstellungen für den Containerhost (d. h. für das physische Netzwerk) entsprechen.

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
Geben Sie eine IP-Adresse unter Verwendung der Option *--Ip* im Ausführungsbefehl von Docker an.

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> Stellen Sie sicher, dass diese IP-Adresse keinem anderen Netzwerkgerät im physischen Netzwerk zugewiesen ist.

Da die Containerendpunkte über direkten Zugriff auf das physische Netzwerk verfügen, besteht keine Notwendigkeit, Portzuordnungen festzulegen.

### L2-Bridge 

Um den Netzwerkmodus mit L2-Brücke zu verwenden, erstellen Sie ein Containernetzwerk mit dem Treiber namens „l2bridge“. Subnetz und Gateway müssen angegeben werden, und zwar wieder in Übereinstimmung mit dem physischen Netzwerk.

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

Bei l2bridge-Netzwerken wird nur die statische IP-Adresszuweisung unterstützt. 

> Bei Verwendung eines l2bridge-Netzwerks für ein SDN-Fabric wird nur die dynamische IP-Zuweisung unterstützt. Weitere Informationen finden Sie im Thema [Anfügen von Containern an ein virtuelles Netzwerk](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network).

## Weitere Vorgänge und Konfigurationen

### Auflisten verfügbarer Netzwerke

```none
# list container networks
C:\> docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
0a297065f06a        nat                 nat                 local
d42516aa0250        none                null                local
```

### Entfernen eines Netzwerks

Verwenden Sie `docker network rm`, um ein Containernetzwerk zu löschen.

```none
C:\> docker network rm "<network name>"
```

Dieser Befehl bereinigt alle virtuellen Hyper-V-Switches, die vom Containernetzwerk verwendet wurden, sowie alle erstellten Objekte für die Netzwerkadressübersetzung (WinNAT – NetNat-Instanzen).

### Netzwerkprüfung 

Um anzuzeigen, welche Container mit einem bestimmten Netzwerk verbunden und welche IPs diesen Containerendpunkten zugeordnet sind, können Sie folgenden Befehl ausführen.

```none
C:\> docker network inspect <network name>
```

### Mehrere Containernetzwerke
 Aktuell wird in Windows nur ein NAT-Netzwerk unterstützt (allerdings könnte eine ausstehende [Pull-Anforderung](https://github.com/docker/docker/pull/25097) helfen, diese Einschränkung zu umgehen). 

Auf einem einzelnen Containerhost können mehrere Containernetzwerke erstellt werden. Dabei gelten folgende Einschränkungen:

* Wenn mehrere Netzwerke vorhanden sind, die einen externen vSwitch für die Verbindung verwenden (z. B. transparent, L2-Brücke, L2 transparent), muss für jedes Netzwerk ein eigener Netzwerkadapter verwendet werden.
* Derzeit ist unsere Lösung zum Erstellen von mehreren NAT-Netzwerken auf einem einzelnen Containerhost die Partitionierung des internen Präfixes des NAT-Netzwerks. Weitere Anleitungen finden Sie im nachfolgenden Abschnitt „Mehrere NAT-Netzwerke“.

### Mehrere NAT-Netzwerke
Es ist möglich, mehrere NAT-Netzwerke auf einem einzelnen Containerhost zu definieren, indem Sie das interne NAT-Netzwerkpräfix des Hosts partitionieren. 

Die Partitionierung für neue NAT-Netzwerke muss unter dem größeren internen NAT-Netzwerkpräfix erfolgen. Das Präfix kann durch die Ausführung des folgenden PowerShell-Befehls gefunden werden und bezieht sich auf das Feld „InternalIPInterfaceAddressPrefix“.

```none
PS C:\> get-netnat
```

Angenommen, das interne NAT-Netzwerkpräfix des Hosts ist 172.16.0.0/12. In diesem Fall kann Docker verwendet werden, um zusätzliche NAT-Netzwerke zu erstellen *solange sie unter das Präfix 172.16.0.0/12 fallen.* Beispielsweise können zwei NAT-Netzwerke mit den IP-Präfixen 172.16.0.0/16 (Gateway, 172.16.0.1) und 172.17.0.0/16 (Gateway, 172.17.0.1) erstellt werden. 

```none
C:\> docker network create -d nat --subnet=172.16.0.0/16 --gateway=172.16.0.1 CustomNat1
C:\> docker network create -d nat --subnet=172.17.0.0/16 --gateway=172.17.0.1 CustomNat2
```

Die neu erstellten Netzwerke können mithilfe folgender Optionen aufgelistet werden:
```none
C:\> docker network ls
```


### Netzwerkauswahl

Beim Erstellen eines Windows-Containers kann ein Netzwerk angegeben werden, mit dem der Netzwerkadapter des Containers verbunden wird. Wenn kein Netzwerk angegeben wurde, wird das NAT-Standardnetzwerk verwendet.

Um den Container mit einem nicht standardmäßigen NAT-Netzwerk zu verbinden verwenden Sie die Option „--network“ mit dem Ausführungsbefehl von Docker.

```none
C:\> docker run -it --network=MyTransparentNet windowsservercore cmd
```

### Statische IP-Adresse

```none
C:\> docker run -it --network=MyTransparentNet --ip=10.80.123.32 windowsservercore cmd
```

Die statische IP-Zuweisung erfolgt direkt im Netzwerkadapter des Containers und darf nur dann durchgeführt werden, wenn sich der Container im Zustand BEENDET befindet. Das Hinzufügen von Containernetzwerkadaptern im laufenden Betrieb (Hot-Add) sowie Änderungen am Netzwerkstapel werden während der Containerausführung nicht unterstützt.

## Docker Compose und Dienstermittlung

> Ein praktisches Beispiel, wie Docker Compose und die Dienstermittlung verwendet werden können, um horizontal skalierte Anwendungen mit mehreren Diensten zu definieren, finden Sie unter [diesem Post (in englischer Sprache)](https://blogs.technet.microsoft.com/virtualization/2016/10/18/use-docker-compose-and-service-discovery-on-windows-to-scale-out-your-multi-service-container-application/) auf unserem [Virtualization Blog](https://blogs.technet.microsoft.com/virtualization/).

### Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) kann verwendet werden, um Containernetzwerke neben den Containern/Diensten zu definieren und konfigurieren, die diese Netzwerke verwenden werden. Der Compose „Netzwerke“-Schlüssel wird als Schlüssel der obersten Ebene verwendet, um die Netzwerke zu definieren, mit denen die Container verbunden werden. Die folgende Syntax definiert beispielsweise das bereits vorhandene NAT-Netzwerk, das von Docker als „Standardnetzwerk“ für alle Container/Dienste erstellt wurde, die in einer bestimmten Compose-Datei definiert sind.

```none
networks:
 default:
  external:
   name: "nat"
```

Die folgende Syntax kann auf ähnliche Weise zum Definieren eines benutzerdefinierten NAT-Netzwerks verwendet werden.

> Hinweis: Das „benutzerdefinierte NAT-Netzwerk“, das im nachfolgenden Beispiel definiert ist, ist als Partition des bereits vorhandenen NAT-internen Präfixes des Containerhosts definiert. Weiteren Kontext finden Sie im Abschnitt „Mehrere NAT-Netzwerke“ weiter oben.

```none
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.17.0.0/16
```

Weitere Informationen über das Definieren/Konfigurieren von Containernetzwerken mit Docker Compose finden Sie unter [Compose File reference (Referenz zur Compose-Datei)](https://docs.docker.com/compose/compose-file/).

### Dienstermittlung
In Docker integriert ist die Dienstermittlung, die die Dienstregistrierung und den Namen für die IP-Zuordnung (DNS) für Container und Dienste übernimmt. Durch die Dienstermittlung ist es möglich, dass sich alle Containerendpunkte durch Namen erkennen können (entweder durch den Containernamen oder den Dienstnamen). Dies ist besonders in Szenarios mit horizontaler Skalierung wertvoll, wo mehrere Containerendpunkte verwendet werden, um einen einzelnen Dienst zu definieren. In solchen Fällen ermöglicht es die Dienstermittlung für einen Dienst, als einzelne Entität betrachtet zu werden, unabhängig davon, wie viele Container im Hintergrund ausgeführt werden. Für Dienste mit mehreren Containern wird der eingehenden Netzwerkverkehr mittels eines Roundrobinansatzes verwaltet, der den DNS-Lastenausgleich verwendet, um Verkehr einheitlich auf alle Containerinstanzen zu verteilen, die einen angegebenen Dienst implementieren.

## Tipps und Einschränkungen

### Firewall

Für den Containerhost müssen bestimmte Firewallregeln erstellt werden, um ICMP (Ping) und DHCP zu ermöglichen. ICMP und DHCP sind für Windows Server-Container erforderlich, um einen Ping zwischen zwei Containern auf dem gleichen Host auszuführen und um dynamisch zugewiesene IP-Adressen über DHCP abzurufen. In TP5 werden diese durch das Skript „Install-ContainerHost.ps1“ erstellt. In Versionen nach TP5 werden diese Regeln automatisch erstellt. Alle Firewallregeln, die für NAT-Portweiterleitungsregeln erforderlich sind, werden automatisch erstellt und nach Beendigung des Containers gelöscht.

### Vorhandener vSwitch blockiert die Erstellung des transparenten Netzwerks

Wenn Sie ein transparentes Netzwerk erstellen, erstellt Docker einen externen vSwitch für dieses und versucht dann den Switch mit einem (externen) Netzwerkadapter zu verbinden. Der Adapter kann ein VM-Netzwerkadapter oder der physische Netzwerkadapter sein. Wenn bereits ein vSwitch auf dem Containerhost erstellt wurde, *und von Docker erkannt wird*, wird das Docker-Modul unter Windows diesen Switch verwenden, anstatt einen neuen zu erstellen. Wenn jedoch der vSwitch Out-of-Band erstellt wurde (z.B. auf dem Containerhost mithilfe des Hyper-V-Managers oder PowerShell) und noch nicht von Docker erkannt wird, versucht das Docker-Modul unter Windows einen neuen vSwitch zu erstellen, kann den neuen Switch dann jedoch nicht mit dem externen Netzwerkadapter des Containerhosts verbinden (da der Netzwerkadapter bereits mit dem Switch verbunden ist, der Out-of-Band erstellt wurde).

Das Problem taucht z.B. auf, wenn Sie zuerst einen neuen vSwitch auf Ihrem Host erstellen würden, während der Docker-Dienst ausgeführt wird, und dann versuchen würden, ein transparentes Netzwerk zu erstellen. In diesem Fall würde Docker den von Ihnen erstellen Switch nicht erkennen und würde einen neuen vSwitch für das transparente Netzwerk erstellen.

Es gibt drei Ansätze zum Lösen dieses Problems:

* Sie können natürlich den vSwitch löschen, der Out-of-Band erstellt wurde. Daraufhin kann Docker einen neuen vSwitch erstellen und ihn dann ohne Probleme mit dem Netzwerkadapter des Hosts verbinden. Bevor Sie sich für diesen Ansatz entscheiden, stellen Sie sicher dass Ihr Out-of-Band-vSwitch nicht von anderen Diensten verwendet wird (z.B. Hyper-V).
* Wenn Sie sich alternativ dafür entscheiden, einen externen Out-of-Band-vSwitch zu verwenden, starten Sie Docker und die HNS-Dienste neu, um *den Switch für Docker erkennbar zu machen.*
```none
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* Eine andere Möglichkeit ist die Verwendung der Option „-o com.docker.network.windowsshim.interface“, um den externen vSwitch des transparenten Netzworks an einen bestimmten Netzwerkadapter zu binden, der noch nicht auf dem Containerhost in Gebrauch ist (z.B. ein anderer Netzwerkadapter als der, der vom vSwitch verwendet wird, der Out-of-Band erstellt wurde). Die „-o“-Option wird weiter oben im Abschnitt [Transparentes Netzwerk](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/management/container_networking#transparent-network) beschrieben.

### Nicht unterstützte Funktionen

Die folgenden Netzwerkfunktionen werden zurzeit über die Docker-Befehlszeilenschnittstelle nicht unterstützt:
 * Overlay-Standardnetzwerktreiber
 * Containerverknüpfung (z. B. „--link“)

Die folgenden Netzwerkoptionen werden in Windows Docker zurzeit nicht unterstützt:
 * --add-host
 * --dns
 * --dns-opt
 * --dns-search
 * -h, --hostname
 * --net-alias
 * --aux-address
 * --internal
 * --ip-range



<!--HONumber=Oct16_HO4-->


