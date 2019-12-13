---
title: Einrichten eines NAT-Netzwerks
description: Einrichten eines NAT-Netzwerks
keywords: Windows 10, Hyper-V
author: jmesser81
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
ms.openlocfilehash: e69775c15359645f3659c9bee3562733415228d5
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909430"
---
# <a name="set-up-a-nat-network"></a>Einrichten eines NAT-Netzwerks

Windows 10 Hyper-V ermöglicht native Netzwerkadressübersetzung (Network Address Translation, NAT) für ein virtuelles Netzwerk.

Diese Anleitung begleitet Sie beim:
* Erstellen eines NAT-Netzwerks
* Herstellen einer Verbindung eines vorhandenen virtuellen Computers mit dem neuen Netzwerk
* Bestätigen, dass der virtuelle Computer ordnungsgemäß verbunden ist

Anforderungen:
* Windows 10 Anniversary Update oder höher
* Hyper-V-Rolle ist aktiviert (Anweisungen [hier](../quick-start/enable-hyper-v.md))

> **Hinweis:** Derzeit steht Ihnen nur ein NAT-Netzwerk pro Host zur Verfügung. Weitere Informationen zur Windows-NAT-Implementierung (WinNAT), zu Funktionen und Einschränkungen finden Sie im Blog [WinNAT capabilities and limitations](https://techcommunity.microsoft.com/t5/Virtualization/Windows-NAT-WinNAT-Capabilities-and-limitations/ba-p/382303) (Stärken und Schwächen von WinNAT).

## <a name="nat-overview"></a>NAT-Überblick
NAT ermöglicht einem virtuellen Computer unter Verwendung der IP-Adresse des Hostcomputers und eines Ports über einen internen virtuellen Hyper-V-Switch Zugriff auf Netzwerkressourcen.

Netzwerkadressübersetzung (NAT, Network Address Translation) ist ein Netzwerkmodus zur Konservierung von IP-Adressen durch Zuordnung einer externen IP-Adresse und eines Ports zu einem wesentlich größeren Satz interner IP-Adressen.  Im Wesentlichen nutzt ein NAT eine Flusstabelle zum Weiterleiten von Datenverkehr von einer IP-Adresse und Portnummer zur richtigen internen IP-Adresse, die einem Gerät im Netzwerk (VM, Computer, Container usw.) zugeordnet ist.

Darüber hinaus erlaubt NAT mehreren VMs das Hosten von Anwendungen, die identische (interne) Kommunikationsports erfordern, indem diese eindeutigen externen Ports zugeordnet werden.

Aus allen diesen Gründen ist NAT-Networking in der Containertechnologie sehr gebräuchlich (siehe [Containernetzwerk](https://docs.microsoft.com/virtualization/windowscontainers/container-networking/architecture)).


## <a name="create-a-nat-virtual-network"></a>Erstellen eines virtuellen NAT-Netzwerks
Betrachten wir nun die Einrichtung eines neuen NAT-Netzwerks.

1.  Öffnen Sie eine PowerShell-Konsole als Administrator.  

2. Erstellen Sie einen internen Switch.

  ``` PowerShell
  New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
  ```

3. Suchen Sie den Schnittstellenindex des virtuelles Switches, den Sie gerade erstellt haben.

    Sie können den Schnittstellen Index ermitteln, indem Sie ausführen `Get-NetAdapter`

    Die Ausgabe sollte etwa wie folgt aussehen:

    ```
    PS C:\> Get-NetAdapter

    Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
    ----                  --------------------               ------- ------       ----------           ---------
    vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
    Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
    Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps

    ```

    Die internen Switches haben Namen wie `vEthernet (SwitchName)` und eine Schnittstellenbeschreibung von `Hyper-V Virtual Ethernet Adapter`. `ifIndex` wird im nächsten Schritt verwendet.

4. Konfigurieren Sie das NAT-Gateway mit [New-NetIPAddress](https://docs.microsoft.com/powershell/module/nettcpip/New-NetIPAddress).  

  Dies ist der generische Befehl:
  ``` PowerShell
  New-NetIPAddress -IPAddress <NAT Gateway IP> -PrefixLength <NAT Subnet Prefix Length> -InterfaceIndex <ifIndex>
  ```

  Um das Gateway zu konfigurieren, benötigen Sie ein paar Informationen über Ihr Netzwerk:  
  * **IPAddress** – Die NAT-Gateway-IP gibt die zu verwendende IPv4- oder IPv6-Adresse an.  
    Die generische Form ist „a.b.c.1“ (z. B. 172.16.0.1).  Die letzte Position muss nicht 1 sein, ist es jedoch in der Regel (basierend auf der Präfixlänge).

    192.168.0.1 ist eine gängige Gateway-IP.  

  * **PrefixLength** – Die NAT-Subnetzpräfixlänge definiert die lokale Subnetzgröße (Subnetzmaske) für NAT.
    Die Subnetzpräfixlänge ist ein Ganzzahlwert zwischen 0 und 32.

    0 würde das gesamte Internet zuordnen, 32 würde nur eine zugeordnete IP zulassen.  Die gängigen Werte reichen von 24 bis 12, je nachdem, wie viele IP-Adressen der NAT zugeordnet werden müssen.

    Eine gängige PrefixLength ist 24 – Dies ist die Subnetzmaske 255.255.255.0

  * **InterfaceIndex** – ifIndex ist der Schnittstellenindex des virtuellen Switches, die Sie im vorherigen Schritt bestimmt haben.

  Führen Sie Folgendes aus, um das NAT-Gateway zu erstellen.

  ``` PowerShell
  New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
  ```

5. Konfigurieren Sie das NAT-Netzwerk mit [New-NetNat](https://docs.microsoft.com/powershell/module/netnat/New-NetNat).  

  Dies ist der generische Befehl:

  ``` PowerShell
  New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
  ```

  Um das Gateway zu konfigurieren, müssen Sie Informationen über das Netzwerk und das NAT-Gateway bereitstellen:  
  * **Name** – NATOutsideName beschreibt den Namen des NAT-Netzwerks.  Hiermit entfernen Sie das NAT-Netzwerk.

  * **InternalIPInterfaceAddressPrefix** – Das NAT-Subnetzpräfix beschreibt sowohl das NAT-Gateway-IP-Präfix von oben als auch die NAT-Subnetzpräfixlänge von oben.

    Die generische Form ist „a.b.c.0/NAT-Subnetzpräfixlänge“.

    In diesem Beispiel verwenden wir von oben „192.168.0.0/24“.

  Führen Sie für unser Beispiel Folgendes aus, um das NAT-Netzwerk einzurichten:

  ``` PowerShell
  New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
  ```

Gratulation!  Sie haben jetzt ein virtuelles NAT-Netzwerk!  Um dem NAT-Netzwerk einen virtuellen Computer hinzuzufügen, befolgen Sie bitte diese [Anweisungen](#connect-a-virtual-machine).

## <a name="connect-a-virtual-machine"></a>Herstellen einer Verbindung mit einem virtuellen Computer

Um einen virtuellen Computer mit Ihrem neuen NAT-Netzwerk zu verbinden, verbinden Sie den internen Switch, den Sie im ersten Schritt des Abschnitts [Erstellen eines virtuellen NAT-Netzwerks](#create-a-nat-virtual-network) erstellt haben, mithilfe des Menüs „VM-Einstellungen“ mit dem virtuellen Computer.

Da WinNAT nicht selbst IP-Adressen reserviert und einem Endpunkt (z. B. einer VM) zuweist, müssen Sie diese Aufgabe in der VM selbst manuell ausführen, d. h. die IP-Adresse im Bereich des NAT-internen Präfixes, die IP-Adresse des Standardgateways und die DNS-Serverinformationen festlegen. Die einzige Einschränkung liegt vor, wenn der Endpunkt an den Container angefügt ist. In diesem Fall wird der Host Compute Service (HCS) vom Host Network Service (HNS) so zugeordnet und verwendet, dass die IP-Adresse, Gateway-IP-Adresse und DNS-Informationen dem Container direkt zugewiesen werden.


## <a name="configuration-example-attaching-vms-and-containers-to-a-nat-network"></a>Konfigurationsbeispiel: Anfügen von VMs und Containern an ein NAT-Netzwerk

_Wenn Sie mehrere VMS und Container an eine einzelne NAT Anfügen müssen, müssen Sie sicherstellen, dass das interne NAT-Subnetzpräfix groß genug ist, um die IP-Adressbereiche abzudecken, die von verschiedenen Anwendungen oder Diensten zugewiesen werden (z. b. docker für Windows und Windows-Container – HNS). Hierfür ist entweder die Zuweisung von IP-Adressen auf Anwendungsebene und die Netzwerkkonfiguration oder die manuelle Konfiguration erforderlich, die von einem Administrator durchgeführt werden muss, und es wird garantiert, dass vorhandene IP-Zuweisungen auf demselben Host nicht erneut verwendet werden._

### <a name="docker-for-windows-linux-vm-and-windows-containers"></a>Docker für Windows (Linux-VM) und Windows-Container
Die nachstehend beschriebene Lösung ermöglicht sowohl Docker für Windows (in Linux-VM ausgeführten Linux-Containern) und Windows-Containern die gemeinsame Nutzung derselben WinNAT-Instanz unter Verwendung getrennter interner vSwitches. Konnektivität zwischen Linux- und Windows-Containern ist gegeben.

Der Benutzer hat VMs über den internen vSwitch „VMNAT“ mit einem NAT-Netzwerk verbunden und möchte nun das Feature „Windows-Container“ mit dem Docker-Modul installieren.
```
PS C:\> Get-NetNat “VMNAT”| Remove-NetNat (this will remove the NAT but keep the internal vSwitch).
Install Windows Container Feature
DO NOT START Docker Service (daemon)
Edit the arguments passed to the docker daemon (dockerd) by adding –fixed-cidr=<container prefix> parameter. This tells docker to create a default nat network with the IP subnet <container prefix> (e.g. 192.168.1.0/24) so that HNS can allocate IPs from this prefix.
PS C:\> Start-Service Docker; Stop-Service Docker
PS C:\> Get-NetNat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> Start-Service docker
```
Docker/HNS weist Windows-Containern IPS zu, und der Administrator weist den VMS IP-Adressen aus dem Unterschieds Satz der beiden zu.

Der Benutzer hat das Feature „Windows-Container“ mit ausgeführtem Docker-Modul installiert und möchte nun VMs mit dem NAT-Netzwerk verbinden.
```
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
PS C:\> Get-NetNat | Remove-NetNat (this will remove the NAT but keep the internal vSwitch)
Edit the arguments passed to the docker daemon (dockerd) by adding -b “none” option to the end of docker daemon (dockerd) command to tell docker not to create a default NAT network.
PS C:\> New-ContainerNetwork –name nat –Mode NAT –subnetprefix <container prefix> (create a new NAT and internal vSwitch – HNS will allocate IPs to container endpoints attached to this network from the <container prefix>)
PS C:\> Get-Netnat | Remove-NetNAT (again, this will remove the NAT but keep the internal vSwitch)
PS C:\> New-NetNat -Name SharedNAT -InternalIPInterfaceAddressPrefix <shared prefix>
PS C:\> New-VirtualSwitch -Type internal (attach VMs to this new vSwitch)
PS C:\> Start-Service docker
```
Docker/HNS weist Windows-Containern IPS zu, und der Administrator weist den VMS IP-Adressen aus dem Unterschieds Satz der beiden zu.

Am Ende verfügen Sie über zwei interne VM-Switches und einem NetNat, das von diesen gemeinsam verwendet wird.

## <a name="multiple-applications-using-the-same-nat"></a>Verwendung der gleichen NAT durch mehrere Anwendungen

In einigen Szenarien müssen mehrere Anwendungen oder Dienste die gleiche NAT verwenden. In diesem Fall muss der folgende Workflow eingehalten werden, damit mehrere Anwendungen/Dienste ein größeres NAT-internes Subnetzpräfix verwenden können.

**_In diesem Beispiel wird die Docker 4-VM mit Windows-docker Beta-Linux zusammen mit dem Windows-Container Feature auf demselben Host ausführlich erläutert. Dieser Workflow kann geändert werden._**

1. C:\> net stop docker
2. Stop Docker4Windows MobyLinux VM
3. PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
4. PS C:\> Get-NetNat | Remove-NetNat  
   *Entfernt alle zuvor vorhandenen Container Netzwerke (d. h. löscht Vswitch, löscht netnat, bereinigt)*  

5. New-ContainerNetwork -Name nat -Mode NAT –subnetprefix 10.0.76.0/24 (Dieses Subnetz wird für das Windows-Container-Feature verwendet.) *Erstellt internen vSwitch namens „nat“*  
   *Erstellt das NAT-Netzwerk mit dem Namen "NAT" mit IP-Präfix 10.0.76.0/24*  

6. Remove-NetNAT  
   *Entfernt sowohl dockernat-als auch NAT-NAT-Netzwerke (behält interne vSwitches bei)*  

7. New-NetNat -Name DockerNAT -InternalIPInterfaceAddressPrefix 10.0.0.0/17 (Dadurch entsteht ein größeres NAT-Netzwerk, das D4W und Container gemeinsam nutzen.)  
   *Erstellt das NAT-Netzwerk mit dem Namen dockernat mit größerem Präfix 10.0.0.0/17*  

8. Run Docker4Windows (MobyLinux.ps1)  
   *Erstellt einen internen Vswitch-dockernat.*  
   *Erstellt das NAT-Netzwerk mit dem Namen "dockernat" mit IP-Präfix 10.0.75.0/24.*  

9. Net start docker  
   *Docker verwendet das benutzerdefinierte NAT-Netzwerk als Standard zum Verbinden von Windows-Containern.*  

Schließlich sollten Sie über zwei interne vSwitches verfügen – einen namens „DockerNAT“ und einen anderen namens „NAT“. Bei Ausführung von Get-NetNat erhalten Sie nur die Bestätigung eines NAT-Netzwerks (10.0.0.0/17). IP-Adressen für Windows-Container werden durch den Windows Host Network Service (HNS) aus dem Subnetz 10.0.76.0/24 zugewiesen. IP-Adressen für Docker 4 Windows werden basierend auf dem vorhandenen Skript „MobyLinux.ps1“ aus dem Subnetz 10.0.75.0/24 zugewiesen.


## <a name="troubleshooting"></a>Problembehandlung

### <a name="multiple-nat-networks-are-not-supported"></a>Mehrere NAT-Netzwerke werden nicht unterstützt.  
In dieser Anleitung wird vorausgesetzt, dass keine anderen NATs auf dem Host vorhanden sind. Allerdings setzen Anwendungen und Dienste die Verwendung einer NAT voraus und erstellen diese möglicherweise beim Setup. Da Windows (WinNAT) nur ein internes NAT-Subnetzpräfix unterstützt, setzt der Versuch, mehrere NATs zu erstellen, das System in einen unbekannten Zustand.

Um herauszufinden, ob dies das Problem ist, stellen Sie sicher, dass Sie über nur eine NAT verfügen:
``` PowerShell
Get-NetNat
```

Wenn bereits eine NAT vorhanden ist, löschen Sie sie.
``` PowerShell
Get-NetNat | Remove-NetNat
```
Stellen Sie sicher, dass Sie nur über einen „internen“ VM-Switch für die Anwendung oder das Feature (z. B. Windows-Container) verfügen. Notieren Sie den Namen des vSwitches.
``` PowerShell
Get-VMSwitch
```

Prüfen Sie, ob private IP-Adressen (z. B. die NAT-Standard-Gateway-IP-Adresse – in der Regel „*.1“) der alten NAT noch einem Adapter zugewiesen sind.
``` PowerShell
Get-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)"
```

Wenn eine alte private IP-Adresse verwendet wird, löschen Sie sie.
``` PowerShell
Remove-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)" -IPAddress <IPAddress>
```

**Entfernen mehrerer NATs**  
Wir haben Berichte über die versehentliche Erstellung mehrerer NAT-Netzwerke erhalten. Die Ursache ist ein Fehler in den letzten Builds: Windows Server 2016 Technical Preview 5 und Windows 10 Insider Preview. Wenn nach Ausführung von „docker network ls“ oder „Get-ContainerNetwork“ mehrere NAT-Netzwerke vorhanden sind, führen Sie über eine PowerShell mit erhöhten Berechtigungen den folgenden Befehl aus:

```
PS> $KeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
PS> $keys = get-childitem $KeyPath
PS> foreach($key in $keys)
PS> {
PS>    if ($key.GetValue("FriendlyName") -eq 'nat')
PS>    {
PS>       $newKeyPath = $KeyPath+"\"+$key.PSChildName
PS>       Remove-Item -Path $newKeyPath -Recurse
PS>    }
PS> }
PS> remove-netnat -Confirm:$false
PS> Get-ContainerNetwork | Remove-ContainerNetwork
PS> Get-VmSwitch -Name nat | Remove-VmSwitch (_failure is expected_)
PS> Stop-Service docker
PS> Set-Service docker -StartupType Disabled
Reboot Host
PS> Get-NetNat | Remove-NetNat
PS> Set-Service docker -StartupType automaticac
PS> Start-Service docker 
```

In diesem [-Installationshandbuch für mehrere Anwendungen mit gleicher NAT](#multiple-applications-using-the-same-nat) finden Sie Informationen zum Neuerstellen Ihrer NAT-Umgebung, falls erforderlich. 

## <a name="references"></a>Verweise
Erfahren Sie mehr über [NAT-Netzwerke](https://en.wikipedia.org/wiki/Network_address_translation).
