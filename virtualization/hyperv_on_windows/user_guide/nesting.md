---
title: Geschachtelte Virtualisierung
description: Geschachtelte Virtualisierung
keywords: windows 10, hyper-v
author: theodthompson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 68c65445-ce13-40c9-b516-57ded76c1b15
---

# Geschachtelte Virtualisierung

Die geschachtelte Virtualisierung bietet die Möglichkeit zum Ausführen von Hyper-V-Hosts in einer virtualisierten Umgebung. Anders ausgedrückt kann mit der geschachtelten Virtualisierung ein Hyper-V-Host selbst virtualisiert werden. Anwendungsbeispiele für die geschachtelte Virtualisierung sind das Ausführen eines Hyper-V-Labs in einer virtualisierten Umgebung und das Bereitstellen von Virtualisierungsdiensten für andere, ohne spezielle Hardware zu benötigen. Außerdem basiert die Windows-Containertechnologie auf der geschachtelten Virtualisierung, wenn Hyper-V-Container auf einem virtualisierten Containerhost ausgeführt werden. Dieses Dokument enthält die Software- und Hardwareanforderungen, Konfigurationsschritte und Informationen zur Fehlerbehebung.

> Bei der geschachtelten Virtualisierung handelt es sich um eine Vorschauversion, die nicht in der Produktion verwendet werden sollte.

## Voraussetzungen

- Windows Insiders-Build (Windows Server 2016, Nano Server oder Windows 10) mit Build 10565 oder höher.
- Auf beiden Hypervisoren (übergeordnet und untergeordnet) müssen identische Windows-Builds ausgeführt werden (10565 oder höher).
- Mindestens 4 GB verfügbarer Arbeitsspeicher (RAM).
- Intel-Prozessor mit Intel-VT-X-Technologie.

## Konfigurieren der geschachtelten Virtualisierung

Erstellen Sie zunächst einen virtuellen Computer, auf dem derselbe Build wie auf dem Host ausgeführt wird. **Schalten Sie den virtuellen Computer nicht ein**. Weitere Informationen finden Sie unter [Erstellen eines virtuellen Computers](../quick_start/walkthrough_create_vm.md).

Sobald der virtuelle Computer erstellt wurde, führen Sie den folgenden Befehl auf dem übergeordneten Hypervisor aus. Dies ermöglicht die geschachtelte Virtualisierung auf dem virtuellen Computer.

```none
Set-VMProcessor -VMName <virtual machine> -ExposeVirtualizationExtensions $true -Count 2
```

Beim Ausführen eines geschachtelten Hyper-V-Hosts muss der dynamische Arbeitsspeicher auf dem virtuellen Computer deaktiviert werden. Dies kann über die Einstellungen des virtuellen Computers oder mit dem folgenden PowerShell-Befehl konfiguriert werden.

```none
Set-VMMemory <virtual machine> -DynamicMemoryEnabled $false
```

Damit geschachtelte virtuelle Computer IP-Adressen empfangen können, muss das Spoofing von MAC-Adressen aktiviert sein. Dies kann mithilfe des folgenden PowerShell-Befehls erreicht werden.

```none
Get-VMNetworkAdapter -VMName <virtual machine> | Set-VMNetworkAdapter -MacAddressSpoofing On
```

Wenn Sie diese Schritte abgeschlossen haben, kann der virtuelle Computer gestartet und Hyper-V installiert werden. Weitere Informationen zum Installieren von Hyper-V finden Sie unter [Installieren von Hyper-V]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install).

## VPN-Gerät herunterladen

Optional kann die geschachtelte Virtualisierung mit dem folgenden Skript aktiviert und konfiguriert werden. Mit den folgenden Befehlen wird das Skript heruntergeladen und ausgeführt.
  
```none
# download script
Invoke-WebRequest https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Enable-NestedVm.ps1 -OutFile .\Enable-NestedVm.ps1 

# run script
.\Enable-NestedVm.ps1 -VmName "DemoVM"
```

## Bekannte Probleme

- Hosts mit aktiviertem Device Guard können keine Virtualisierungserweiterungen für Gäste verfügbar machen.
- Hosts mit aktivierter virtualisierungsbasierter Sicherheit (VBS) können keine Virtualisierungserweiterungen für Gäste verfügbar machen. Sie müssen VBS zuerst deaktivieren, um die geschachtelte Virtualisierung zu verwenden.
- Die Verbindung mit dem virtuellen Computer wird möglicherweise unterbrochen, wenn ein leeres Kennwort verwendet wird. Ändern Sie Ihr Kennwort. Hierdurch sollte das Problem behoben werden.
- Wenn die geschachtelte Virtualisierung auf einem virtuellen Computer aktiviert ist, sind die folgenden Features nicht mehr mit diesem virtuellen Computer kompatibel.  
  * Ändern der Größe des Laufzeitspeichers.
  * Anwenden eines Prüfpunkts auf einen ausgeführten virtuellen Computer.
  * Die Live-Migration für einen virtuellen Computer, der andere virtuelle Computer hostet, ist nicht möglich.
  * Speichern und Wiederherstellen funktionieren nicht.

## Häufig gestellte Fragen und Problembehandlung

Mein virtueller Computer startet nicht. Was muss ich tun?

1. Stellen Sie sicher, dass der dynamische Arbeitsspeicher deaktiviert ist.
2. Führen Sie [dieses PowerShell-Skript](https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/master/hyperv-tools/Nested/Get-NestedVirtStatus.ps1) auf dem Hostcomputer an einer Eingabeaufforderung mit erhöhten Rechten aus.
  
## Feedback senden

Nutzen Sie für Rückmeldungen die Windows-Feedback-App, die [Virtualisierungsforen](https://social.technet.microsoft.com/Forums/windowsserver/En-us/home?forum=winserverhyperv) oder [GitHub](https://github.com/Microsoft/Virtualization-Documentation).



<!--HONumber=Jun16_HO2-->


