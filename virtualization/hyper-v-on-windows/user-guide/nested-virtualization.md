---
title: Geschachtelte Virtualisierung
description: Geschachtelte Virtualisierung
keywords: Windows 10, Hyper-V
author: johncslack
ms.date: 12/18/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.openlocfilehash: f819ac04773188525af202d370ba271a2d93e259
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439347"
---
# <a name="run-hyper-v-in-a-virtual-machine-with-nested-virtualization"></a>Ausführen von Hyper-V auf einem virtuellen Computer mit geschachtelter Virtualisierung

Die geschachtelte Virtualisierung ist ein Feature, mit dem Sie Hyper-V auf einem virtuellen Hyper-V-Computer (VM) ausführen können. Dies ist bei der Ausführung eines Smartphone-Emulator in Visual Studio auf einem virtuellen Computer hilfreich, oder beim Testen von Konfigurationen, die normalerweise mehrere Hosts erfordern.

![](./media/HyperVNesting.png)

## <a name="prerequisites"></a>Erforderliche Komponenten

* Sowohl der Hyper-V-Host als auch der Gast müssen Windows Server 2016/Windows 10 Anniversary Update oder höher ausführen.
* VM-Konfigurationsversion 8.0 oder höher.
* Ein Intel-Prozessor mit VT-x- und EPT-Technologie – die Schachtelung ist derzeit **nur Intel**.
* Es gibt einige Unterschiede bei virtuellen Netzwerken für sekundäre virtuelle Computer. Weitere Informationen finden Sie unter "Geschachtelte VM-Netzwerke".


## <a name="configure-nested-virtualization"></a>Konfigurieren der geschachtelten Virtualisierung

1. Erstellen einer virtuellen Maschine Siehe die obigen erforderlichen Komponenten zum Bestimmen der erforderlichen Betriebssystem- und VM-Versionen.
2. Während der virtuelle Computer den Status AUS hat, führen Sie den folgenden Befehl auf dem physischen Hyper-V-Host aus. Dadurch wird die geschachtelte Virtualisierung auf dem virtuellen Computer aktiviert.

```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. Starten Sie einen virtuellen Computer.
4. Installieren Sie Hyper-V auf dem virtuellen Computer ebenso wie auf einem physischen Server. Weitere Informationen zum Installieren von Hyper-V finden Sie unter [Installieren von Hyper-V](../quick-start/enable-hyper-v.md).

## <a name="disable-nested-virtualization"></a>Deaktivieren der geschachtelten Virtualisierung
Sie können die geschachtelte Virtualisierung für einen beendeten virtuellen Computer mit dem folgenden PowerShell-Befehl deaktivieren:
```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## <a name="dynamic-memory-and-runtime-memory-resize"></a>Ändern der Größe des dynamischen Arbeitsspeichers und Laufzeitspeichers
Wenn Hyper-V auf einem virtuellen Computer ausgeführt wird, muss der virtuelle Computer ausgeschaltet werden, um seinen Arbeitsspeicher anpassen. Dies bedeutet, dass auch bei aktiviertem dynamischen Arbeitsspeicher die Arbeitsspeichermenge nicht schwankt. Bei virtuellen Computern ohne aktivierten dynamischen Arbeitsspeicher misslingt jeder Versuch, die Arbeitsspeichermenge bei laufendem Betrieb anzupassen. 

Beachten Sie, dass das bloße Aktivieren der geschachtelten Virtualisierung keine Auswirkung auf die Änderung der Größe des dynamischen Arbeitsspeichers oder Laufzeitspeichers hat. Die Inkompatibilität tritt nur auf, solange Hyper-V auf dem virtuellen Computer ausgeführt wird.

## <a name="networking-options"></a>Netzwerkoptionen

Es gibt zwei Optionen für das Verwenden von Netzwerke mit geschachtelten virtuellen Computern: 

1. Spoofing von MAC-Adressen
2. NAT-Networking

### <a name="mac-address-spoofing"></a>Spoofing von MAC-Adressen
Damit Netzwerkpakete über zwei virtuelle Switches geleitet werden können, muss das Spoofing von MAC-Adressen auf der ersten Ebene (L1) des virtuellen Switches aktiviert sein. Dies kann mithilfe des folgenden PowerShell-Befehls erreicht werden.

``` PowerShell
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="network-address-translation-nat"></a>Netzwerkadressübersetzung (NAT)
Die zweite Option basiert auf Netzwerkadressübersetzung (NAT). Dieser Ansatz eignet sich am besten für Situationen, in denen das Spoofing von MAC-Adressen nicht möglich ist, wie in beispielsweise in einer öffentlichen Cloudumgebung.

Zunächst muss ein virtueller NAT-Switch auf dem virtuellen Hostcomputer (dem "mittleren" virtuellen Computer) erstellt werden. Beachten Sie, dass die IP-Adressen nur beispielhaft sind und in unterschiedlichen Umgebungen variieren:

``` PowerShell
New-VMSwitch -Name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```

Als Nächstes weisen Sie dem Netzadapter eine IP-Adresse zu:

``` PowerShell
Get-NetAdapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```

Jedem geschachtelten virtuellen Computer muss eine IP-Adresse und ein Gateway zugewiesen sein. Beachten Sie, dass die Gateway-IP auf den NAT-Adapter aus dem vorherigen Schritt verweisen muss. Gegebenenfalls sollten Sie auch einen DNS-Server zuweisen:

``` PowerShell
Get-NetAdapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## <a name="how-nested-virtualization-works"></a>So funktioniert die geschachtelte Virtualisierung

Moderne Prozessoren enthalten Hardwarefeatures, die die Virtualisierung schneller und sicherer machen. Hyper-V erfordert die Unterstützung der Prozessorerweiterungen (z. B. Intel VT-X und AMD-V), um virtuelle Computer ausführen zu können. Sobald Hyper-V startet, kann keine andere Software mehr auf die Funktionen dieser Prozessoren zugreifen.  Dadurch wird verhindert, dass Gastcomputer Hyper-V ausführen.

Eine geschachtelte Virtualisierung macht diese Hardwareunterstützung für virtuelle Gastcomputer verfügbar.

Das folgende Diagramm zeigt Hyper-V ohne Schachtelung.  Der Hyper-V-Hypervisor hat die vollständige Kontrolle über die Hardwarevirtualisierungsfunktionen (orangefarbener Pfeil) und macht diese nicht für das Gastbetriebssystem verfügbar.

![](./media/HVNoNesting.PNG)

Im Gegensatz dazu zeigt das folgende Diagramm Hyper-V mit aktivierter geschachtelter Virtualisierung. In diesem Fall stellt Hyper-V die Hardwarevirtualisierungserweiterungen seinen virtuellen Computern zur Verfügung. Bei aktivierter Schachtelung können Gast-VMs ihren eigenen Hypervisor installieren und eigene Gast-VMs ausführen.

![](./media/HVNesting.png)

## <a name="3rd-party-virtualization-apps"></a>Virtualisierungs-Apps anderer Anbieter

Andere Virtualisierungsanwendungen als Hyper-V werden auf virtuellen Hyper-V-Computern nicht unterstützt und funktionieren wahrscheinlich nicht. Dies schließt jegliche Software ein, die Virtualisierungserweiterungen für Hardware benötigt.
