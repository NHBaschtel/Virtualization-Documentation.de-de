---
title: Einrichten eines NAT-Netzwerks
description: Einrichten eines NAT-Netzwerks
keywords: windows 10, hyper-v
author: jmesser81
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1f8a691c-ca75-42da-8ad8-a35611ad70ec
---

# Einrichten eines NAT-Netzwerks

Windows 10 Hyper-V ermöglicht native Netzwerkadressübersetzung (Network Address Translation, NAT) für ein virtuelles Netzwerk.

Diese Anleitung begleitet Sie beim:
* Erstellen eines NAT-Netzwerks
* Herstellen einer Verbindung eines vorhandenen virtuellen Computers mit dem neuen Netzwerk
* Bestätigen, dass der virtuelle Computer ordnungsgemäß verbunden ist

Anforderungen:
* Windows-Build 14295 oder höher
* Hyper-V-Rolle ist aktiviert (Anweisungen [hier](../quick_start/walkthrough_create_vm.md))

> **Hinweis:**  Derzeit lässt Hyper-V nur das Erstellen eines einzigen NAT-Netzwerks zu.

## NAT-Überblick
NAT ermöglicht einem virtuellen Computer unter Verwendung der IP-Adresse des Hostcomputers und eines Ports Zugriff auf Netzwerkressourcen.

Netzwerkadressübersetzung (NAT, Network Address Translation) ist ein Netzwerkmodus zur Konservierung von IP-Adressen durch Zuordnung einer externen IP-Adresse und eines Ports zu einem wesentlich größeren Satz interner IP-Adressen.  Im Grunde leitet ein NAT Switch den Netzwerkverkehr mittels einer NAT-Zuordnungstabelle von einer IP-Adresse und Portnummer an die richtige interne IP-Adresse, die einem Gerät im Netzwerk (VM, Computer, Container etc.) zugeordnet ist.

Darüber hinaus erlaubt NAT mehreren VMs das Hosten von Anwendungen, die identische (interne) Kommunikationsports erfordern, indem diese eindeutigen externen Ports zugeordnet werden.

Aus allen diesen Gründen ist NAT-Networking in der Containertechnologie sehr gebräuchlich (siehe [Containernetzwerk](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/management/container_networking)).


## Erstellen eines virtuellen NAT-Netzwerks
Betrachten wir nun die Einrichtung eines neuen NAT-Netzwerks.

1.  Öffnen Sie eine PowerShell-Konsole als Administrator.  

2. Erstellen Sie einen internen Switch.  

  ``` PowerShell
  New-VMSwitch -SwitchName "SwitchName" -SwitchType Internal
  ```

3. Konfigurieren Sie das NAT-Gateway mit [New-NetIPAddress](https://technet.microsoft.com/en-us/library/hh826150.aspx).  

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

  * **InterfaceIndex** – Dies ist der Schnittstellenindex des oben erstellten virtuellen Switches.

    Sie können den Schnittstellenindex ermitteln, indem Sie Folgendes ausführen: `Get-NetAdapter`

    Die Ausgabe sollte etwa wie folgt aussehen:

    ```
    PS C:\> Get-NetAdapter

    Name                  InterfaceDescription               ifIndex Status       MacAddress           LinkSpeed
    ----                  --------------------               ------- ------       ----------           ---------
    vEthernet (intSwitch) Hyper-V Virtual Ethernet Adapter        24 Up           00-15-5D-00-6A-01      10 Gbps
    Wi-Fi                 Marvell AVASTAR Wireless-AC Net...      18 Up           98-5F-D3-34-0C-D3     300 Mbps
    Bluetooth Network ... Bluetooth Device (Personal Area...      21 Disconnected 98-5F-D3-34-0C-D4       3 Mbps

    ```

    Die internen Switches haben Namen wie `vEthernet (SwitchName)` und eine Schnittstellenbeschreibung von `Hyper-V Virtual Ethernet Adapter`.

  Führen Sie Folgendes aus, um das NAT-Gateway zu erstellen.

  ``` PowerShell
  New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex 24
  ```

4. Konfigurieren Sie das NAT-Netzwerk mit [New-NetNat](https://technet.microsoft.com/en-us/library/dn283361(v=wps.630).aspx).  

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

Gratulation!  Sie haben jetzt ein virtuelles NAT-Netzwerk!  Um dem NAT-Netzwerk einen virtuellen Computer hinzuzufügen, befolgen Sie bitte diese [Anweisungen](setup_nat_network.md#connect-a-virtual-machine).

## Herstellen einer Verbindung mit einem virtuellen Computer

Um einen virtuellen Computer mit Ihrem neuen NAT-Netzwerk zu verbinden, verbinden Sie den internen Switch, den Sie im ersten Schritt des Abschnitts [Erstellen eines virtuellen NAT-Netzwerks](setup_nat_network.md#create-a-nat-virtual-network) erstellt haben, mithilfe des Menüs „VM-Einstellungen“ mit dem virtuellen Computer.


## Problembehandlung

Dieser Workflow setzt voraus, dass keine anderen NATs auf dem Host vorhanden sind. Jedoch erfordern manchmal mehrere Anwendungen oder Dienste eine NAT-Nutzung. Da Windows (WinNAT) nur ein internes NAT-Subnetzpräfix unterstützt, setzt der Versuch, mehrere NATs zu erstellen, das System in einen unbekannten Zustand.

### Schritte zur Fehlerbehebung
1. Stellen Sie sicher, dass Sie nur eine NAT haben.

  ``` PowerShell
  Get-NetNat
  ```
2. Wenn bereits eine NAT vorhanden ist, löschen Sie sie.

  ``` PowerShell
  Get-NetNat | Remove-NetNat
  ```

3. Stellen Sie sicher, dass Sie nur einen „internen“ VMSwitch für die NAT haben. Notieren Sie den Namen des vSwitches für Schritt 4.

  ``` PowerShell
  Get-VMSwitch
  ```

4. Überprüfen Sie, ob private IP-Adressen (z. B. die NAT-Standard-Gateway-IP-Adresse – in der Regel „*.1“) des alten NAT noch einem Adapter zugewiesen sind.

  ``` PowerShell
  Get-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)"
  ```

5. Wenn eine alte private IP-Adresse verwendet wird, löschen Sie sie.  
   ``` PowerShell
  Remove-NetIPAddress -InterfaceAlias "vEthernet(<name of vSwitch>)" -IPAddress <IPAddress>
  ```

## Verwendung der gleichen NAT durch mehrere Anwendungen

In einigen Szenarien müssen mehrere Anwendungen oder Dienste die gleiche NAT verwenden. In diesem Fall muss der folgende Workflow eingehalten werden, damit mehrere Anwendungen/Dienste ein größeres NAT-internes Subnetzpräfix verwenden können.

**_Als Beispiel erläutern wir ausführlich, wie Docker 4 Windows – Docker Beta – Linux VM mit dem Windows-Container-Feature auf dem gleichen Host koexistieren. Dieser Workflow unterliegt Änderungen._**

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


## Verweise
Erfahren Sie mehr über [NAT-Netzwerke](https://en.wikipedia.org/wiki/Network_address_translation).


<!--HONumber=May16_HO5-->


