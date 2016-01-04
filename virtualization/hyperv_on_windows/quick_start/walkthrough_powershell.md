# Schritt 8: Arbeiten mit Hyper-V und Windows PowerShell

Nachdem Sie sich mit den Grundlagen der Bereitstellung von Hyper-V sowie dem Erstellen und Verwalten virtueller Computer vertraut gemacht haben, wollen wir uns nun ansehen, wie viele dieser Aktivitäten mithilfe von PowerShell automatisiert werden können.

### Abrufen einer Liste von Hyper-V-Befehlen

1.  Klicken Sie auf die Windows-Schaltfläche „Start“, und geben Sie **powershell** ein.
2.  Führen Sie den folgenden Befehl aus, um eine durchsuchbare Liste mit PowerShell-Befehlen anzuzeigen, die für das PowerShell-Modul für Hyper-V verfügbar sind.

 ```powershell
get-command –module hyper-v | out-gridview
 ```
  Sie erhalten eine Rückgabe wie diese:

  ![](media\command_grid.png)

3. Um weitere Informationen zu einem bestimmten PowerShell-Befehl zu erhalten, verwenden Sie `get-help`. Bei Ausführen des folgenden Befehls werden z. B. Informationen zum Hyper-V-Befehl `get-vm` zurückgegeben.

  ```powershell
get-help get-vm
  ```
 Die Ausgabe veranschaulicht die Struktur des Befehls, die erforderlichen und optionalen Parameter sowie die Aliase, die Sie verwenden können.

 ![](media\get_help.png)


### Abrufen einer Liste virtueller Computer

Mit `get-vm` können Sie eine Liste virtueller Computer zurückgeben.

1. Führen Sie in PowerShell folgenden Befehl aus:

 ```powershell
get-vm
 ```
 Die Ausgabe ist wie folgt:

 ![](media\get_vm.png)

2. Um nur eingeschaltete virtuelle Computer zurückzugeben, fügen Sie dem Befehl `get-vm` einen Filter hinzu. Ein Filter kann mit dem „Where-Object“-Befehl hinzugefügt werden. Weitere Informationen zum Filtern finden Sie in der Dokumentation zum [Verwenden von „Where-Object“](https://technet.microsoft.com/en-us/library/ee177028.aspx).

 ```powershell
 get-vm | where {$_.State –eq ‘Running’}
 ```
3.  Zum Auflisten aller virtuellen Computer im ausgeschalteten Zustand führen Sie den folgenden Befehl aus. Dieser Befehl ist eine Kopie des Befehls aus Schritt 2, wobei der Filter von „Running“ in „Off“ geändert wurde.

 ```powershell
 get-vm | where {$_.State –eq ‘Off’}
 ```

### Starten und Herunterfahren virtueller Computer

1. Um einen bestimmten virtuellen Computer zu starten, führen Sie den folgenden Befehl mit dem Namen des virtuellen Computers aus:

 ```powershell
 Start-vm –Name <virtual machine name>
 ```

2. Zum Starten aller derzeit ausgeschalteten virtuellen Computer rufen Sie eine Liste dieser VMs ab, die Sie an den Befehl „start-vm“ übergeben:

  ```powershell
 get-vm | where {$_.State –eq ‘Off’} | start-vm
  ```
3. Zum Herunterfahren aller ausgeführten virtuellen Computer führen Sie diesen Befehl aus:

  ```powershell
 get-vm | where {$_.State –eq ‘Running’} | stop-vm
  ```

### Erstellen eines VM-Prüfpunkts

Wählen Sie zum Erstellen eines Prüfpunkts mithilfe von PowerShell den virtuellen Computer mit dem Befehl `get-vm` aus, und übergeben Sie diesen an den Befehl `checkpoint-vm`. Benennen Sie abschließend den Prüfpunkt mit dem Befehl `-snapshotname`. Der vollständige Befehl sieht folgendermaßen aus:

 ```powershell
 get-vm -Name <VM Name> | checkpoint-vm -snapshotname <name for snapshot>
 ```
### Erstellen eines neuen virtuellen Computers

Im folgenden Beispiel wird das Erstellen ein neues virtuellen Computers in der PowerShell ISE (Integrated Scripting Environment) gezeigt. Dies ist ein einfaches Beispiel, das mit zusätzlichen PowerShell-Features und komplexeren VM-Bereitstellungen erweitert werden kann.

1. Geben Sie zum Öffnen der PowerShell ISE im Startmenü **PowerShell ISE** ein.
2. Führen Sie den folgenden Code zum Erstellen eines virtuellen Computers aus. In der Dokumentation zu [New-VM](https://technet.microsoft.com/en-us/library/hh848537.aspx) finden Sie ausführliche Informationen zum Befehl „New-VM“.

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

## Zusammenfassung und Referenzen

In diesem Dokument wurden einige einfache Schritte mit dem PowerShell-Modul für Hyper-V sowie verschiedene Beispielszenarien vorgestellt. Weitere Informationen zum PowerShell-Modul für Hyper-V finden Sie in der [Referenz zu Hyper-V-Cmdlets in Windows PowerShell](https://technet.microsoft.com/%5Clibrary/Hh848559.aspx).




