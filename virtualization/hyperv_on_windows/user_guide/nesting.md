---
title: Geschachtelte Virtualisierung
description: Geschachtelte Virtualisierung
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 06/20/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
ms.sourcegitcommit: ef18b63c454b3c12a7067d3604ba142d55403226
ms.openlocfilehash: 2d1679ffe4876ddd4eefe1b457098e8797899492

---

# Ausführen von Hyper-V auf einem virtuellen Computer mit geschachtelter Virtualisierung

Die geschachtelte Virtualisierung ist ein Feature, mit dem Sie Hyper-V auf einem virtuellen Hyper-V-Computer ausführen können. Anders ausgedrückt kann mit der geschachtelten Virtualisierung ein Hyper-V-Host selbst virtualisiert werden. Anwendungsbeispiele für die geschachtelte Virtualisierung sind beispielsweise das Ausführen eines Hyper-V-Containers auf einem virtualisierten Containerhost, das Einrichten eines Hyper-V-Labs in einer virtualisierten Umgebung oder das Testen von Szenarien mit mehreren Computern, ohne dass spezielle Hardware benötigt wird. Dieses Dokument enthält die Software- und Hardwareanforderungen, Konfigurationsschritte und Informationen zur Fehlerbehebung.

## Voraussetzungen

- Ein Hyper-V-Host mit einem Windows Insiders-Build (Windows Server 2016 oder Windows 10) mit Build 10565 oder höher.
- Auf beiden Hypervisoren (übergeordnet und untergeordnet) müssen identische Windows-Builds ausgeführt werden (10565 oder höher).
- Intel-Prozessor mit Intel-VT-X- und EPT-Technologie.

## Konfigurieren der geschachtelten Virtualisierung

Erstellen Sie zunächst einen virtuellen Computer. **Schalten Sie den virtuellen Computer nicht ein**. Weitere Informationen finden Sie unter [Erstellen eines virtuellen Computers](../quick_start/walkthrough_create_vm.md).

Sobald die virtuelle Maschine erstellt wurde, führen Sie den folgenden Befehl auf dem physischen Hyper-V-Host aus. Dadurch wird die geschachtelte Virtualisierung auf dem virtuellen Computer aktiviert.

```none
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
Beim Ausführen eines geschachtelten Hyper-V-Hosts muss der dynamische Arbeitsspeicher auf dem virtuellen Computer deaktiviert werden. Dies kann über die Einstellungen des virtuellen Computers oder mit dem folgenden PowerShell-Befehl konfiguriert werden.
```none
Set-VMMemory -VMName <VMName> -DynamicMemoryEnabled $false
```

Wenn Sie diese Schritte abgeschlossen haben, kann der virtuelle Computer gestartet und Hyper-V installiert werden. Weitere Informationen zum Installieren von Hyper-V finden Sie unter [Installieren von Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install).

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


## Bekannte Probleme

- Hosts mit aktiviertem Device Guard können keine Virtualisierungserweiterungen für Gäste verfügbar machen.
- Auf virtuellen Computern mit aktivierter virtualisierungsbasierter Sicherheit (VBS) kann nicht gleichzeitig die geschachtelte Virtualisierung aktiviert sein. Sie müssen VBS zuerst deaktivieren, um die geschachtelte Virtualisierung zu verwenden.
- Wenn die geschachtelte Virtualisierung auf einem virtuellen Computer aktiviert ist, sind die folgenden Features nicht mehr mit diesem virtuellen Computer kompatibel.  
  * Ändern der Größe des Laufzeitspeichers und dynamischer Arbeitsspeicher
  * Prüfpunkte
  * Ein virtueller Computer mit aktiviertem Hyper-V kann nicht live migriert werden.

## Häufig gestellte Fragen und Problembehandlung

Mein virtueller Computer startet nicht. Was muss ich tun?

1. Stellen Sie sicher, dass der dynamische Arbeitsspeicher deaktiviert ist.
2. Stellen Sie sicher, dass Sie über einen schnellen Intel-Prozessor verfügen.
3. Führen Sie [dieses PowerShell-Skript](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1) auf dem Hostcomputer an einer Eingabeaufforderung mit erhöhten Rechten aus.

## Feedback senden

Nutzen Sie für Rückmeldungen die Windows-Feedback-App, die [Virtualisierungsforen](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv) oder [GitHub](https://github.com/Microsoft/Virtualization-Documentation).



<!--HONumber=Jun16_HO3-->


