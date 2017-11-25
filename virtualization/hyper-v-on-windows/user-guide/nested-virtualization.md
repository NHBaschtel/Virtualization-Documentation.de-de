---
title: Geschachtelte Virtualisierung
description: Geschachtelte Virtualisierung
keywords: Windows 10, Hyper-V
author: theodthompson
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.openlocfilehash: 6f3cc3edb42a063abed33c7783e4a8bf13324cda
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/08/2017
---
# <a name="run-hyper-v-in-a-virtual-machine-with-nested-virtualization"></a>Ausführen von Hyper-V auf einem virtuellen Computer mit geschachtelter Virtualisierung

Die geschachtelte Virtualisierung ist ein Feature, mit dem Sie Hyper-V auf einem virtuellen Hyper-V-Computer ausführen können. Anders ausgedrückt kann mit der geschachtelten Virtualisierung ein Hyper-V-Host selbst virtualisiert werden. Anwendungsbeispiele für die geschachtelte Virtualisierung sind beispielsweise das Ausführen eines Hyper-V-Containers auf einem virtualisierten Containerhost, das Einrichten eines Hyper-V-Labs in einer virtualisierten Umgebung oder das Testen von Szenarien mit mehreren Computern, ohne dass spezielle Hardware benötigt wird. Dieses Dokument enthält die Software- und Hardwareanforderungen, Konfigurationsschritte und Einschränkungen. 

## <a name="prerequisites"></a>Voraussetzungen

- Hyper-V-Host unter Windows Server 2016 oder Windows 10 Anniversary Update.
- Hyper-V-VM unter Windows Server 2016 oder Windows 10 Anniversary Update.
- Hyper-V-VM mit Konfigurationsversion 8.0 oder höher.
- Intel-Prozessor mit VT-x- und EPT-Technologie.

## <a name="configure-nested-virtualization"></a>Konfigurieren der geschachtelten Virtualisierung

1. Erstellen Sie einen virtuellen Computer. Siehe die obigen erforderlichen Komponenten zum Bestimmen der erforderlichen Betriebssystem- und VM-Versionen.
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
Es gibt zwei Optionen für Netzwerke mit geschachtelten virtuellen Computern: Spoofing von MAC-Adressen und NAT-Modus.

### <a name="mac-address-spoofing"></a>Spoofing von MAC-Adressen
Damit Netzwerkpakete über zwei virtuelle Switches geleitet werden können, muss das Spoofing von MAC-Adressen auf der ersten Ebene des virtuellen Switches aktiviert sein. Dies kann mithilfe des folgenden PowerShell-Befehls erreicht werden.

```
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### <a name="network-address-translation"></a>Netzwerkadressübersetzung (Network Address Translation, NAT)
Die zweite Option basiert auf Netzwerkadressübersetzung (NAT). Dieser Ansatz eignet sich am besten für Situationen, in denen das Spoofing von MAC-Adressen nicht möglich ist, wie in beispielsweise in einer öffentlichen Cloudumgebung.

Zunächst muss ein virtueller NAT-Switch auf dem virtuellen Hostcomputer (dem "mittleren" virtuellen Computer) erstellt werden. Beachten Sie, dass die IP-Adressen nur beispielhaft sind und in unterschiedlichen Umgebungen variieren:
```
New-VMSwitch -Name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```
Als Nächstes weisen Sie dem Netzadapter eine IP-Adresse zu:
```
Get-NetAdapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```
Jedem geschachtelten virtuellen Computer muss eine IP-Adresse und ein Gateway zugewiesen sein. Beachten Sie, dass die Gateway-IP auf den NAT-Adapter aus dem vorherigen Schritt verweisen muss. Gegebenenfalls sollten Sie auch einen DNS-Server zuweisen:
```
Get-NetAdapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## <a name="3rd-party-virtualization-apps"></a>Virtualisierungs-Apps anderer Anbieter
Andere Virtualisierungsanwendungen als Hyper-V werden auf virtuellen Hyper-V-Computern nicht unterstützt und funktionieren wahrscheinlich nicht. Dies schließt jegliche Software ein, die Virtualisierungserweiterungen für Hardware benötigt.
