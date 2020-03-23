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
ms.openlocfilehash: 1652c3bcb32ddbc4e05e8821d0e646a76a2fd4f0
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853964"
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

> **Hinweis:**  Derzeit steht Ihnen nur ein NAT-Netzwerk pro Host zur Verfügung. Weitere Informationen zur Windows-NAT-Implementierung (WinNAT), zu Funktionen und Einschränkungen finden Sie im Blog [WinNAT capabilities and limitations](https://techcommunity.microsoft.com/t5/Virtualization/Windows-NAT-WinNAT-Capabilities-and-limitations/ba-p/382303) (Stärken und Schwächen von WinNAT).

## <a name="nat-overview"></a>NAT-Überblick
NAT ermöglicht einem virtuellen Computer unter Verwendung der IP-Adresse des Hostcomputers und eines Ports über einen internen virtuellen Hyper-V-Switch Zugriff auf Netzwerkressourcen.

Netzwerkadressübersetzung (NAT, Network Address Translation) ist ein Netzwerkmodus zur Konservierung von IP-Adressen durch Zuordnung einer externen IP-Adresse und eines Ports zu einem wesentlich größeren Satz interner IP-Adressen.  Im Wesentlichen nutzt ein NAT eine Flusstabelle zum Weiterleiten von Datenverkehr von einer IP-Adresse und Portnummer zur richtigen internen IP-Adresse, die einem Gerät im Netzwerk (VM, Computer, Container usw.) zugeordnet ist.

Darüber hinaus erlaubt NAT mehreren VMs das Hosten von Anwendungen, die identische (interne) Kommunikationsports erfordern, indem diese eindeutigen externen Ports zugeordnet werden.

Aus allen diesen Gründen ist NAT-Networking in der Containertechnologie sehr gebräuchlich (siehe [Containernetzwerk](https://docs.microsoft.com/virtualization/windowscontainers/container-networking/architecture)).


## <a name="create-a-nat-virtual-network"></a>Erstellen eines virtuellen NAT-Netzwerks
Betrachten wir nun die Einrichtung eines neuen NAT-Netzwerks.

1. Öffnen Sie eine PowerShell-Konsole als Administrator.  

2. Erstellen Sie einen internen Switch.

    ```powershell
    New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
    ```

3. Suchen Sie den Schnittstellenindex des virtuelles Switches, den Sie gerade erstellt haben.

    Sie können den Schnittstellenindex ermitteln, indem Sie `Get-NetAdapter` ausführen.

    Die Ausgabe sollte etwa wie folgt aussehen:

    ```console
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
    ```powershell
    New-NetIPAddress -IPAddress <NAT Gateway IP> -PrefixLength <NAT Subnet Prefix Length> -InterfaceIndex <ifIndex>
    ```

    Um das Gateway zu konfigurieren, benötigen Sie ein paar Informationen über Ihr Netzwerk:  
    * **IPAddress** – Die NAT-Gateway-IP gibt die zu verwendende IPv4- oder IPv6-Adresse an.  
      Die generische Form ist „a.b.c.1“ (z. B. 172.16.0.1).  Die letzte Position muss nicht 1 sein, ist es jedoch in der Regel (basierend auf der Präfixlänge).

      192.168.0.1 ist eine gängige Gateway-IP.  

    * **PrefixLength** – Die Präfixlänge des NAT-Subnetzes definiert die lokale Subnetzgröße (Subnetzmaske) für NAT.
      Die Subnetzpräfixlänge ist ein Ganzzahlwert zwischen 0 und 32.

      0 würde das gesamte Internet zuordnen, 32 würde nur eine zugeordnete IP zulassen.  Die gängigen Werte reichen von 24 bis 12, je nachdem, wie viele IP-Adressen der NAT zugeordnet werden müssen.

      Eine gängige PrefixLength ist 24 – Dies ist die Subnetzmaske 255.255.255.0

    * **InterfaceIndex** – ifIndex ist der Schnittstellenindex des virtuellen Switches, die Sie im vorherigen Schritt bestimmt haben.

    Führen Sie Folgendes aus, um das NAT-Gateway zu erstellen.

    ```powershell
    New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
    ```

5. Konfigurieren Sie das NAT-Netzwerk mit [New-NetNat](https://docs.microsoft.com/powershell/module/netnat/New-NetNat).  

    Dies ist der generische Befehl:

    ```powershell
    New-NetNat -Name <NATOutsideName> -InternalIPInterfaceAddressPrefix <NAT subnet prefix>
    ```

    Um das Gateway zu konfigurieren, müssen Sie Informationen über das Netzwerk und das NAT-Gateway bereitstellen:  
    * **Name** – NATOutsideName beschreibt den Namen des NAT-Netzwerks.  Hiermit entfernen Sie das NAT-Netzwerk.

    * **InternalIPInterfaceAddressPrefix** – Das NAT-Subnetzpräfix beschreibt sowohl das NAT-Gateway-IP-Präfix von oben als auch die NAT-Subnetzpräfixlänge von oben.

    Die generische Form ist „a.b.c.0/NAT-Subnetzpräfixlänge“.

    In diesem Beispiel verwenden wir von oben „192.168.0.0/24“.

    Führen Sie für unser Beispiel Folgendes aus, um das NAT-Netzwerk einzurichten:

    ```powershell
    New-NetNat -Name MyNATnetwork -InternalIPInterfaceAddressPrefix 192.168.0.0/24
    ```

Herzlichen Glückwunsch!  Sie haben jetzt ein virtuelles NAT-Netzwerk!  Um dem NAT-Netzwerk einen virtuellen Computer hinzuzufügen, befolgen Sie bitte diese [Anweisungen](#connect-a-virtual-machine).

## <a name="connect-a-virtual-machine"></a>Herstellen einer Verbindung mit einem virtuellen Computer

Um einen virtuellen Computer mit Ihrem neuen NAT-Netzwerk zu verbinden, verbinden Sie den internen Switch, den Sie im ersten Schritt des Abschnitts [Erstellen eines virtuellen NAT-Netzwerks](#create-a-nat-virtual-network) erstellt haben, mithilfe des Menüs „VM-Einstellungen“ mit dem virtuellen Computer.

Da WinNAT nicht selbst IP-Adressen reserviert und einem Endpunkt (z. B. einer VM) zuweist, müssen Sie diese Aufgabe in der VM selbst manuell ausführen, d. h. die IP-Adresse im Bereich des NAT-internen Präfixes, die IP-Adresse des Standardgateways und die DNS-Serverinformationen festlegen. Die einzige Einschränkung liegt vor, wenn der Endpunkt an den Container angefügt ist. In diesem Fall wird der Host Compute Service (HCS) vom Host Network Service (HNS) so zugeordnet und verwendet, dass die IP-Adresse, Gateway-IP-Adresse und DNS-Informationen dem Container direkt zugewiesen werden.


## <a name="configuration-example-attaching-vms-and-containers-to-a-nat-network"></a>Konfigurationsbeispiel: Anfügen von VMs und Containern an ein NAT-Netzwerk

_Wenn Sie mehrere VMs und Containern an ein einzelnes NAT anfügen müssen, ist sicherzustellen, dass das NAT-interne Subnetzpräfix für die IP-Bereiche groß genug ist, die verschiedenen Anwendungen oder Diensten (z. B. Docker für Windows und Windows-Container, HNS) zugewiesen sind. Dieser Vorgang erfordert entweder eine Zuweisung von IP-Adressen auf Anwendungsebene und Netzwerkkonfiguration oder eine manuelle Konfiguration, die von einem Administrator vorgenommen werden muss und sicherstellt, dass vorhandene IP-Adresszuweisungen nicht auf demselben Host wiederverwendet werden._

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
Docker/HNS weist Windows-Containern IP-Adressen zu. Der Administrator weist VMs IP-Adressen aus dem Differenzsatz der beiden zu.

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
Docker/HNS weist Windows-Containern IP-Adressen zu. Der Administrator weist VMs IP-Adressen aus dem Differenzsatz der beiden zu.

Am Ende verfügen Sie über zwei interne VM-Switches und einem NetNat, das von diesen gemeinsam verwendet wird.

## <a name="multiple-applications-using-the-same-nat"></a>Verwendung der gleichen NAT durch mehrere Anwendungen

In einigen Szenarien müssen mehrere Anwendungen oder Dienste die gleiche NAT verwenden. In diesem Fall muss der folgende Workflow eingehalten werden, damit mehrere Anwendungen/Dienste ein größeres NAT-internes Subnetzpräfix verwenden können.

**_Als Beispiel erläutern wir ausführlich, wie Docker 4 Windows – Docker Beta – Linux VM mit dem Windows-Container-Feature auf dem gleichen Host koexistieren. Dieser Workflow unterliegt Änderungen._**

1. C:\> net stop docker
2. Stop Docker4Windows MobyLinux VM
3. PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork -force
4. PS C:\> Get-NetNat | Remove-NetNat  
   *Entfernt bereits vorhandene Containernetzwerke (d. h. löscht vSwitch, löscht NetNat, bereinigt).*  

5. New-ContainerNetwork -Name nat -Mode NAT –subnetprefix 10.0.76.0/24 (Dieses Subnetz wird für das Windows-Container-Feature verwendet.) *Erstellt internen vSwitch namens „nat“*  
   *Erstellt NAT-Netzwerk mit dem Namen „nat“ mit IP-Präfix 10.0.76.0/24*  

6. Remove-NetNAT  
   *Entfernt DockerNAT- und Nat-Netzwerke (behält interne vSwitches bei)*  

7. New-NetNat -Name DockerNAT -InternalIPInterfaceAddressPrefix 10.0.0.0/17 (Dadurch entsteht ein größeres NAT-Netzwerk, das D4W und Container gemeinsam nutzen.)  
   *Erstellt ein NAT-Netzwerk mit dem Namen DockerNAT mit größeren Präfix 10.0.0.0/17*  

8. Run Docker4Windows (MobyLinux.ps1)  
   *Erstellt internes vSwitch-DockerNAT*  
   *Erstellt ein NAT-Netzwerk mit dem Namen „DockerNAT“ mit IP-Präfix 10.0.75.0/24*  

9. Net start docker  
   *Docker verwendet das benutzerdefinierte NAT-Netzwerk als Standard zur Verbindung mit Windows-Containern.*  

Schließlich sollten Sie über zwei interne vSwitches verfügen – einen namens „DockerNAT“ und einen anderen namens „NAT“. Bei Ausführung von Get-NetNat erhalten Sie nur die Bestätigung eines NAT-Netzwerks (10.0.0.0/17). IP-Adressen für Windows-Container werden durch den Windows Host Network Service (HNS) aus dem Subnetz 10.0.76.0/24 zugewiesen. IP-Adressen für Docker 4 Windows werden basierend auf dem vorhandenen Skript „MobyLinux.ps1“ aus dem Subnetz 10.0.75.0/24 zugewiesen.


## <a name="troubleshooting"></a>Problembehandlung

### <a name="multiple-nat-networks-are-not-supported"></a>Mehrere NAT-Netzwerke werden nicht unterstützt.  
In dieser Anleitung wird vorausgesetzt, dass keine anderen NATs auf dem Host vorhanden sind. Allerdings setzen Anwendungen und Dienste die Verwendung einer NAT voraus und erstellen diese möglicherweise beim Setup. Da Windows (WinNAT) nur ein internes NAT-Subnetzpräfix unterstützt, setzt der Versuch, mehrere NATs zu erstellen, das System in einen unbekannten Zustand.

Um herauszufinden, ob dies das Problem ist, stellen Sie sicher, dass Sie über nur eine NAT verfügen:
```powershell
Get-NetNat
```

Wenn bereits eine NAT vorhanden ist, löschen Sie sie.
```powershell
Get-NetNat | Remove-NetNat
```
Stellen Sie sicher, dass Sie nur über einen „internen“ VM-Switch für die Anwendung oder das Feature (z. B. Windows-Container) verfügen. Notieren Sie den Namen des vSwitches.
```powershell
Get-VMSwitch
```

Überprüfen Sie, ob private IP-Adressen (z. B. die NAT-Standard-Gateway-IP-Adresse – in der Regel _x_._y_._z_.1) des alten NAT noch einem Adapter zugewiesen sind.
```powershell
Get-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)"
```

Wenn eine alte private IP-Adresse verwendet wird, löschen Sie sie.
```powershell
Remove-NetIPAddress -InterfaceAlias "vEthernet (<name of vSwitch>)" -IPAddress <IPAddress>
```

**Entfernen mehrerer NATs**  
Wir haben Berichte über die versehentliche Erstellung mehrerer NAT-Netzwerke erhalten. Die Ursache ist ein Fehler in den letzten Builds: Windows Server 2016 Technical Preview 5 und Windows 10 Insider Preview. Wenn nach Ausführung von „docker network ls“ oder „Get-ContainerNetwork“ mehrere NAT-Netzwerke vorhanden sind, führen Sie über eine PowerShell mit erhöhten Berechtigungen den folgenden Befehl aus:

```powershell
$keys = Get-ChildItem "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
foreach($key in $keys)
{
   if ($key.GetValue("FriendlyName") -eq 'nat')
   {
      $newKeyPath = $KeyPath+"\"+$key.PSChildName
      Remove-Item -Path $newKeyPath -Recurse
   }
}
Remove-NetNat -Confirm:$false
Get-ContainerNetwork | Remove-ContainerNetwork
Get-VmSwitch -Name nat | Remove-VmSwitch # failure is expected
Stop-Service docker
Set-Service docker -StartupType Disabled
```

Starten Sie das Betriebssystem neu, bevor Sie die nachfolgenden Befehle ausführen (`Restart-Computer`).

```powershell
Get-NetNat | Remove-NetNat
Set-Service docker -StartupType Automatic
Start-Service docker 
```

In diesem [-Installationshandbuch für mehrere Anwendungen mit gleicher NAT](#multiple-applications-using-the-same-nat) finden Sie Informationen zum Neuerstellen Ihrer NAT-Umgebung, falls erforderlich. 

## <a name="references"></a>Referenzen
Erfahren Sie mehr über [NAT-Netzwerke](https://en.wikipedia.org/wiki/Network_address_translation).
