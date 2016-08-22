---
title: Windows-Containernetzwerk
description: "Konfigurieren Sie das Netzwerk für Windows-Container."
keywords: Docker, Container
author: jmesser81
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
translationtype: Human Translation
ms.sourcegitcommit: c412171773e9c66569eab2252b5adfc187eedafd
ms.openlocfilehash: eb7d2c25d929cb51abfad17c26a89105f6574a48

---

# Containernetzwerk

Windows-Container funktionieren in Bezug auf Netzwerke ähnlich wie virtuelle Computer. Jeder Container verfügt über einen virtuellen Netzwerkadapter, der mit einem virtuellen Switch verbunden ist, über den eingehender und ausgehender Datenverkehr weitergeleitet wird. Um eine Trennung der Container zu erzwingen, die sich auf dem gleichen Host befinden, wird für jeden Windows Server- und Hyper-V-Container ein Netzwerkbereich erstellt, in dem der Netzwerkadapter für den Container installiert wird. Windows Server-Container verwenden eine Host-vNIC für die Verbindung mit dem virtuellen Switch. Hyper-V-Container verwenden eine synthetische VM-NIC (nicht für die Utility-VM verfügbar gemacht) für die Verbindung mit dem virtuellen Switch.

Windows-Container unterstützen vier verschiedene Netzwerktreiber oder -modi: *nat*, *transparent*, *l2bridge* und *l2tunnel*. Sie sollten den Netzwerkmodus auswählen, der Ihren Bedürfnissen im Hinblick auf Ihre physische Netzwerkinfrastruktur und die Netzwerkanforderungen (Netzwerk mit einem oder mehreren Hosts) am besten gerecht wird. 

Beim ersten Ausführen des dockerd-Diensts wird vom Docker-Modul standardmäßig ein NAT-Netzwerk erstellt. Das erstellte interne IP-Standardpräfix ist 172.16.0.0/12. 

> Hinweis: Wenn sich Ihre Containerhost-IP im selben Präfix befindet, müssen Sie das NAT-interne IP-Präfix entsprechend der folgenden Beschreibung ändern.

Containerendpunkte werden an dieses Standardnetzwerk angefügt und einer IP-Adresse aus dem internen Präfix zugewiesen. Aktuell wird in Windows nur ein NAT-Netzwerk unterstützt (allerdings könnte eine ausstehende [Pull-Anforderung](https://github.com/docker/docker/pull/25097) helfen, diese Einschränkung zu umgehen). 

Auf demselben Containerhost können zusätzliche Netzwerke, die einen anderen Treiber verwenden (z. B. „transparent“ „l2bridge“), erstellt werden. Die folgende Tabelle zeigt, wie die Netzwerkkonnektivität für interne (Container-zu-Container) und externe Verbindungen für jeden Modus bereitgestellt wird.

- **Netzwerkadressübersetzung (Network Address Translation, NAT)** – jeder Container erhält eine IP-Adresse aus einem internen, privaten IP-Präfix (z. B. 172.16.0.0/12). Portweiterleitung/-zuordnung vom Containerhost zu Containerendpunkten wird unterstützt.

- **Transparent** – Jeder Containerendpunkt ist direkt mit dem physischen Netzwerk verbunden. IPs aus dem physischen Netzwerk können mithilfe eines externen DHCP-Servers statisch oder dynamisch zugewiesen werden.

- **L2-Bridge** – Alle Containerendpunkte befinden sich im gleichen IP-Subnetz wie der Containerhost. Die IP-Adressen müssen statisch aus dem gleichen Präfix wie der Containerhost zugewiesen werden. Alle Containerendpunkte auf dem Host verfügen aufgrund der Layer-2-Adressübersetzung über dieselbe MAC-Adresse.

- **L2-Tunnel** - _Dieser Modus sollte nur in einem Microsoft-Cloudstapel verwendet werden._

> Informationen dazu, wie Sie Containerendpunkte mithilfe des Microsoft-SDN-Stapels mit einem virtuellen Overlaynetzwerk verbinden, finden Sie im Thema [Verbinden von Containern mit einem virtuellen Netzwerk](location).

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

Das Docker-Modul unter Windows erstellt ein „nat“-Standardnetzwerk mit IP-Präfix 172.16.0.0/12. Aktuell ist nur ein NAT-Netzwerk auf einem Windows-Containerhost zulässig. Wenn ein Benutzer ein NAT-Netzwerk mit einem bestimmten IP-Präfix erstellen möchte, kann er dazu eine von zwei Optionen in der Docker-Konfigurationsdatei „daemon.json“ ändern (diese befindet sich unter „C:\ProgramData\Docker\config\daemon.json“. Ist sie noch nicht vorhanden, muss sie erstellt werden).
 1. Verwenden der Option _„fixed-cidr“: „< IP Prefix > / Mask“_. Dadurch wird das NAT-Standardnetzwerk erstellt, und IP-Präfix und Übereinstimmung werden festgelegt.
 2. Verwenden der Option _„bridge“: „none“_. Dadurch wird kein Standardnetzwerk erstellt. Ein Benutzer kann unter Verwendung des Befehls *docker network create -d <driver>* ein benutzerdefiniertes Netzwerk mit einem beliebigen Treiber erstellen.

Vor dem Ausführen einer dieser Konfigurationsoptionen muss der Docker-Dienst zuerst beendet werden, und alle bereits vorhandenen NAT-Netzwerke müssen gelöscht werden.

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

Wenn der Benutzer die Option „fixed-cidr“ zur Datei „daemon.json“ hinzugefügt hat, wird vom Docker-Modul jetzt ein benutzerdefiniertes NAT-Netzwerk mit dem angegebenen benutzerdefinierten IP-Präfix und der angegebenen Maske erstellt. Wenn der Benutzer stattdessen die Option „Bridge: none“ hinzugefügt hat, muss er manuell ein Netzwerk erstellen.

```none
# Create a user-defined nat network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

Standardmäßig werden Containerendpunkte mit dem NAT-Standardnetzwerk verbunden. Wenn das NAT-Standardnetzwerk nicht erstellt wurde (da in der daemon.json-Datei „bridge:none“ angegeben wurde) oder der Zugriff auf ein anderes, benutzerdefinierte Netzwerk erforderlich ist, können Benutzer den Parameter *--network* mit Ausführungsbefehl von Docker angeben.

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### Portzuordnung

Für den Zugriff auf Anwendungen, die in mit einem NAT-Netzwerk verbundenen Container ausgeführt werden, müssen zwischen dem Containerhost und dem Containerendpunkt Portzuordnungen erstellt werden. Diese Zuordnungen müssen zum Zeitpunkt der Erstellung des Containers, oder während der Container sich im Zustand ANGEHALTEN befindet, festgelegt werden.

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Create a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

Dynamische Portzuordnungen werden ebenfalls unterstützt, und zwar entweder unter Verwendung des Parameters „-p“ oder des Befehls EXPOSE in einer Dockerfile. Auf dem Containerhost wird per Zufallsprinzip ein kurzlebiger Port ausgewählt, der bei Ausführung von „Docker.ps“ überprüft werden kann.

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
 1. Pro Containerhost wird nur ein NAT-Netzwerk (ein internes IP-Präfix) unterstützt.
 2. Containerendpunkte sind nur vom Containerhost unter Verwendung der internen IP und des internen Ports erreichbar.

Zusätzliche Netzwerke können mit anderen Treibern erstellt werden. 

> Docker-Netzwerktreiber bestehen vollständig aus Kleinbuchstaben.

### Transparentes Netzwerk

Um den transparenten Netzwerkmodus zu verwenden, erstellen Sie ein Containernetzwerk mit dem Treiber namens „transparent“. 

```none
C:\> docker network create -d transparent MyTransparentNetwork
```

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

> Bei Verwendung eines l2bridge-Netzwerks für ein SDN-Fabric wird nur die dynamische IP-Zuweisung unterstützt. Weitere Informationen finden Sie im Thema [Anfügen von Containern an ein virtuelles Netzwerk](location).

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

Auf einem einzelnen Containerhost können mehrere Containernetzwerke erstellt werden. Dabei gelten folgende Einschränkungen:
* Pro Containerhost kann nur ein NAT-Netzwerk erstellt werden.
* Wenn mehrere Netzwerke vorhanden sind, die einen externen vSwitch für die Verbindung verwenden (z. B. transparent, L2-Brücke, L2 transparent), muss für jedes Netzwerk ein eigener Netzwerkadapter verwendet werden.

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


## Tipps und Einschränkungen

### Firewall

Für den Containerhost müssen bestimmte Firewallregeln erstellt werden, um ICMP (Ping) und DHCP zu ermöglichen. ICMP und DHCP sind für Windows Server-Container erforderlich, um einen Ping zwischen zwei Containern auf dem gleichen Host auszuführen und um dynamisch zugewiesene IP-Adressen über DHCP abzurufen. In TP5 werden diese durch das Skript „Install-ContainerHost.ps1“ erstellt. In Versionen nach TP5 werden diese Regeln automatisch erstellt. Alle Firewallregeln, die für NAT-Portweiterleitungsregeln erforderlich sind, werden automatisch erstellt und nach Beendigung des Containers gelöscht.

### Nicht unterstützte Funktionen

Die folgenden Netzwerkfunktionen werden zurzeit über die Docker-Befehlszeilenschnittstelle nicht unterstützt:
 * Containerverknüpfung (z. B. „--link“)
 * Namens-/dienstbasierte IP-Auflösung für Container. _Wird bald mit einem-Wartungsupdate unterstützt_
 * Overlay-Standardnetzwerktreiber

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

 > In Windows Server 2016 Technical Preview 5 und den jüngsten Windows Insider Preview (WIP)-Builds gibt es das bekannte Problem, dass beim Upgrade auf einen neuen Build ein Containernetzwerk und vSwitch dupliziert werden. Führen Sie das folgende Skript aus, um dieses Problem zu umgehen.
```none
$KeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
$keys = get-childitem $KeyPath
foreach($key in $keys)
{
   if ($key.GetValue("FriendlyName") -eq 'nat')
   {
      $newKeyPath = $KeyPath+"\"+$key.PSChildName
      Remove-Item -Path $newKeyPath -Recurse
   }
}
remove-netnat -Confirm:$false
Get-ContainerNetwork | Remove-ContainerNetwork
Get-VmSwitch -Name nat | Remove-VmSwitch # Note: failure is expected
Stop-Service docker
Set-Service docker -StartupType Disabled
```
> Starten Sie den Host neu, und führen Sie die verbleibenden Schritte aus:
```none
Get-NetNat | Remove-NetNat -Confirm $false
Set-Service docker -StartupType automatic
Start-Service docker 
```



<!--HONumber=Aug16_HO3-->


