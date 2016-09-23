---
title: Geschachtelte Virtualisierung
description: Geschachtelte Virtualisierung
keywords: Windows 10, Hyper-V
author: theodthompson
manager: timlt
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
translationtype: Human Translation
ms.sourcegitcommit: 4eb3e733990ca86f28620e23bf408ff71b879173
ms.openlocfilehash: c94b7f2c9c90eaf2834a1d0c54fb1f9b2a8f9669

---

# Ausführen von Hyper-V auf einem virtuellen Computer mit geschachtelter Virtualisierung

Die geschachtelte Virtualisierung ist ein Feature, mit dem Sie Hyper-V auf einem virtuellen Hyper-V-Computer ausführen können. Anders ausgedrückt kann mit der geschachtelten Virtualisierung ein Hyper-V-Host selbst virtualisiert werden. Anwendungsbeispiele für die geschachtelte Virtualisierung sind beispielsweise das Ausführen eines Hyper-V-Containers auf einem virtualisierten Containerhost, das Einrichten eines Hyper-V-Labs in einer virtualisierten Umgebung oder das Testen von Szenarien mit mehreren Computern, ohne dass spezielle Hardware benötigt wird. Dieses Dokument enthält die Software- und Hardwareanforderungen, Konfigurationsschritte und Einschränkungen. 

## Voraussetzungen

- Hyper-V-Host unter Windows Server 2016 oder Windows 10 Anniversary Update.
- Hyper-V-VM unter Windows Server 2016 oder Windows 10 Anniversary Update.
- Hyper-V-VM mit Konfigurationsversion 8.0 oder höher.
- Intel-Prozessor mit VT-x- und EPT-Technologie.

## Konfigurieren der geschachtelten Virtualisierung

1. Erstellen Sie einen virtuellen Computer. Siehe die obigen erforderlichen Komponenten zum Bestimmen der erforderlichen Betriebssystem- und VM-Versionen.
2. Während der virtuelle Computer den Status AUS hat, führen Sie den folgenden Befehl auf dem physischen Hyper-V-Host aus. Dadurch wird die geschachtelte Virtualisierung auf dem virtuellen Computer aktiviert.
```none
    Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. Starten Sie einen virtuellen Computer.
4. Installieren Sie Hyper-V auf dem virtuellen Computer ebenso wie auf einem physischen Server. Weitere Informationen zum Installieren von Hyper-V finden Sie unter [Installieren von Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install).

## Deaktivieren der geschachtelten Virtualisierung
Sie können die geschachtelte Virtualisierung für einen beendeten virtuellen Computer mit dem folgenden PowerShell-Befehl deaktivieren:
```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

## Ändern der Größe des dynamischen Arbeitsspeichers und Laufzeitspeichers
Wenn Hyper-V auf einem virtuellen Computer ausgeführt wird, muss der virtuelle Computer ausgeschaltet werden, um seinen Arbeitsspeicher anpassen. Dies bedeutet, dass auch bei aktiviertem dynamischen Arbeitsspeicher die Arbeitsspeichermenge nicht schwankt. Bei virtuellen Computern ohne aktivierten dynamischen Arbeitsspeicher misslingt jeder Versuch, die Arbeitsspeichermenge bei laufendem Betrieb anzupassen. 

Beachten Sie, dass das bloße Aktivieren der geschachtelten Virtualisierung keine Auswirkung auf die Änderung der Größe des dynamischen Arbeitsspeichers oder Laufzeitspeichers hat. Die Inkompatibilität tritt nur auf, solange Hyper-V auf dem virtuellen Computer ausgeführt wird.

## Netzwerkoptionen
Es gibt zwei Optionen für Netzwerke mit geschachtelten virtuellen Computern: Spoofing von MAC-Adressen und NAT-Modus.

### Spoofing von MAC-Adressen
Damit Netzwerkpakete über zwei virtuelle Switches geleitet werden können, muss das Spoofing von MAC-Adressen auf der ersten Ebene des virtuellen Switches aktiviert sein. Dies kann mithilfe des folgenden PowerShell-Befehls erreicht werden.

```none
Get-VMNetworkAdapter -VMName <VMName> | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### Netzwerkadressübersetzung (Network Address Translation, NAT)
Die zweite Option basiert auf Netzwerkadressübersetzung (NAT). Dieser Ansatz eignet sich am besten für Situationen, in denen das Spoofing von MAC-Adressen nicht möglich ist, wie in beispielsweise in einer öffentlichen Cloudumgebung.

Zunächst muss ein virtueller NAT-Switch auf dem virtuellen Hostcomputer (dem "mittleren" virtuellen Computer) erstellt werden. Beachten Sie, dass die IP-Adressen nur beispielhaft sind und in unterschiedlichen Umgebungen variieren:
```none
new-vmswitch -name VmNAT -SwitchType Internal
New-NetNat –Name LocalNAT –InternalIPInterfaceAddressPrefix “192.168.100.0/24”
```
Als Nächstes weisen Sie dem Netzadapter eine IP-Adresse zu:
```none
get-netadapter "vEthernet (VmNat)" | New-NetIPAddress -IPAddress 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
```
Jedem geschachtelten virtuellen Computer muss eine IP-Adresse und ein Gateway zugewiesen sein. Beachten Sie, dass die Gateway-IP auf den NAT-Adapter aus dem vorherigen Schritt verweisen muss. Gegebenenfalls sollten Sie auch einen DNS-Server zuweisen:
```none
get-netadapter "Ethernet" | New-NetIPAddress -IPAddress 192.168.100.2 -DefaultGateway 192.168.100.1 -AddressFamily IPv4 -PrefixLength 24
Netsh interface ip add dnsserver “Ethernet” address=<my DNS server>
```

## Virtualisierungs-Apps anderer Anbieter
Andere Virtualisierungsanwendungen als Hyper-V werden auf virtuellen Hyper-V-Computern nicht unterstützt und funktionieren wahrscheinlich nicht. Dies schließt jegliche Software ein, die Virtualisierungserweiterungen für Hardware benötigt.



<!--HONumber=Sep16_HO3-->


