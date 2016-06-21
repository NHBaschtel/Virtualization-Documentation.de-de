---
title: &2004983527 Arbeiten mit Hyper-V und Windows PowerShell
description: Arbeiten mit Hyper-V und Windows PowerShell
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &365806987 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6d1ae036-0841-4ba5-b7e0-733aad31e9a7
---

# Arbeiten mit Hyper-V und Windows PowerShell

Nachdem Sie sich mit den Grundlagen der Bereitstellung von Hyper-V sowie dem Erstellen und Verwalten virtueller Computer vertraut gemacht haben, wollen wir uns nun ansehen, wie viele dieser Aktivitäten mithilfe von PowerShell automatisiert werden können.

### Zurückgeben einer Liste von Hyper-V-Befehle

1.  Klicken Sie auf die Windows-Schaltfläche „Start“, und geben Sie <g id="2" ctype="x-strong">powershell</g> ein.
2.  Führen Sie den folgenden Befehl aus, um eine durchsuchbare Liste mit PowerShell-Befehlen anzuzeigen, die für das PowerShell-Modul für Hyper-V verfügbar sind.

 ```powershell
get-command -module hyper-v | out-gridview
 ```
  Sie erhalten eine Rückgabe wie diese:

  <g id="1" ctype="x-linkText"></g>

3. Um weitere Informationen zu einem bestimmten PowerShell-Befehl zu erhalten, verwenden Sie <g id="2" ctype="x-code">get-help</g>. Bei Ausführen des folgenden Befehls werden z. B. Informationen zum Hyper-V-Befehl <g id="2" ctype="x-code">get-vm</g> zurückgegeben.

  ```powershell
get-help get-vm
  ```
 Die Ausgabe veranschaulicht die Struktur des Befehls, die erforderlichen und optionalen Parameter sowie die Aliase, die Sie verwenden können.

 <g id="1" ctype="x-linkText"></g>


### Abrufen einer Liste virtueller Computer

Mit <g id="2" ctype="x-code">get-vm</g> können Sie eine Liste virtueller Computer zurückgeben.

1. Führen Sie in PowerShell folgenden Befehl aus:

 ```powershell
get-vm
 ```
 Die Ausgabe ist wie folgt:

 <g id="1" ctype="x-linkText"></g>

2. Um nur eingeschaltete virtuelle Computer zurückzugeben, fügen Sie dem Befehl <g id="2" ctype="x-code">get-vm</g> einen Filter hinzu. Ein Filter kann mit dem „Where-Object“-Befehl hinzugefügt werden. Weitere Informationen zum Filtern finden Sie in der Dokumentation zum <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Verwenden von „Where-Object“</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

 ```powershell
 get-vm | where {$_.State -eq ‘Running’}
 ```
3.  Listen Sie alle virtuellen Computer in einen funktionsfähigen Status, off, führen Sie den folgenden Befehl. Mit diesem Befehl wird eine Kopie des Befehls aus Schritt 2 mit dem Filter aus 'Running' auf 'Off' geändert.

 ```powershell
 get-vm | where {$_.State -eq ‘Off’}
 ```

### Starten und Herunterfahren von virtuellen Maschinen

1. Um einen bestimmten virtuellen Computer zu starten, führen Sie den folgenden Befehl mit dem Namen des virtuellen Computers:

 ```powershell
 Start-vm -Name <virtual machine name>
 ```

2. Zum Starten aller derzeit ausgeschalteten virtuellen Computer rufen Sie eine Liste dieser VMs ab, die Sie an den Befehl „start-vm“ übergeben:

  ```powershell
 get-vm | where {$_.State -eq ‘Off’} | start-vm
  ```
3. Zum Herunterfahren aller ausgeführten virtuellen Computer führen Sie diesen Befehl aus:

  ```powershell
 get-vm | where {$_.State -eq ‘Running’} | stop-vm
  ```

### Erstellen eines VM-Prüfpunkts

Wählen Sie zum Erstellen eines Prüfpunkts mithilfe von PowerShell den virtuellen Computer mit dem Befehl <g id="2" ctype="x-code">get-vm</g> aus, und übergeben Sie diesen an den Befehl <g id="4" ctype="x-code">checkpoint-vm</g>. Benennen Sie abschließend den Prüfpunkt mit dem Befehl <g id="2" ctype="x-code">-snapshotname</g>. Der vollständige Befehl sieht folgendermaßen aus:

 ```powershell
 get-vm -Name <VM Name> | checkpoint-vm -snapshotname <name for snapshot>
 ```
### Erstellen eines neuen virtuellen Computers

Im folgenden Beispiel wird das Erstellen ein neues virtuellen Computers in der PowerShell ISE (Integrated Scripting Environment) gezeigt. Dies ist ein einfaches Beispiel, das mit zusätzlichen PowerShell-Features und komplexeren VM-Bereitstellungen erweitert werden kann.

1. Geben Sie zum Öffnen der PowerShell ISE im Startmenü <g id="2" ctype="x-strong">PowerShell ISE</g> ein.
2. Führen Sie den folgenden Code zum Erstellen eines virtuellen Computers aus. In der Dokumentation zu <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">New-VM</g><g id="2CapsExtId3" ctype="x-title"></g></g> finden Sie ausführliche Informationen zum Befehl „New-VM“.

  ```powershell
 $VMName = "VMNAME"

 $VM = @{
     Name = $VMName 
     MemoryStartupBytes = 2147483648
     Generation = 2
     NewVHDPath = "C:\Virtual Machines\$VMName\$VMName.vhdx"
     NewVHDSizeBytes = 53687091200
     BootDevice = "VHD"
     Path = "C:\Virtual Machines\$VMName "
     SwitchName = (get-vmswitch).Name[0]
 }

 New-VM @VM
  ```

## Wrappen und Verweise

Dieses Dokument hat einige einfache Schritte zum Explorer das Hyper-V-PowerShell-Modul sowie einige Beispielszenarios gezeigt. Weitere Informationen zum PowerShell-Modul für Hyper-V finden Sie in der <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Referenz zu Hyper-V-Cmdlets in Windows PowerShell</g><g id="2CapsExtId3" ctype="x-title"></g></g>.






<!--HONumber=May16_HO1-->


