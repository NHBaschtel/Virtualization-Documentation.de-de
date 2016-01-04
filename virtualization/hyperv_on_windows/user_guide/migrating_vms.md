# Migrieren und Aktualisieren virtueller Computer

Wenn Sie virtuelle Computer auf Ihren Windows 10-Host verschieben, der ursprünglich mit Hyper-V unter Windows 8.1 oder früher erstellt wurde, können Sie die neuen Features für virtuelle Computer erst nutzen, nachdem Sie die Konfigurationsversion des virtuellen Computers manuell aktualisiert haben.

Um die Konfigurationsversion zu aktualisieren, fahren Sie den virtuellen Computer herunter und wählen dann im Hyper-V-Manager „Konfiguration des virtuellen Computers aktualisieren“. Sie können auch eine Windows PowerShell-Eingabeaufforderung mit erhöhten Rechten öffnen und Folgendes eingeben:

 ```PowerShell
Update-VmVersion <vmname> | <vmobject>
 ```

## Wie überprüfe ich die Konfigurationsversion der in Hyper-V ausgeführten virtuellen Computer?

Um die Konfigurationsversion zu bestimmen, öffnen Sie eine Windows PowerShell-Eingabeaufforderung mit erhöhten Rechten und führen den folgenden Befehl aus:

**Get-VM * | Format-Table Name, Version**

Der PowerShell-Befehl erzeugt die folgende Beispielausgabe:

```
Name        State       CPUUsage(%) MemoryAssigned(M)   Uptime              Status                  Version

Atlantis    Running         0       1024                00:04:20.5910000    Operating normally      5.0

SGC VM      Running         0       538                 00:02:44.8350000    Operating normally      6.2
```


## Was geschieht, wenn ich die Konfigurationsversion nicht aktualisiere?

Wenn Sie virtuelle Computer haben, die mit einer früheren Version von Hyper-V erstellt wurden, funktionieren einige Features auf diesen virtuellen Computern ggf. erst, nachdem Sie die VM-Version aktualisiert haben.

Mindestversion für die VM-Konfiguration für neue Hyper-V-Features:

| **Featurename**| **VM-Mindestversion**| |
| :------------------------------------- | -----------------: ||
| Speicher bei laufendem Systembetrieb hinzufügen/entfernen| 6.0| |
| Netzwerkadapter bei laufendem Systembetrieb hinzufügen/entfernen| 5.0| |
| Sicherer Start für Linux-VMs| 6.0| |
| Produktionsprüfpunkte| 6.0| |
| PowerShell Direct| 6.2| |
| Virtual Trusted Platform Module (vTPM)| 6.2| |
| VM-Gruppierung| 6.2| |



## VM-Konfigurationsversion

Beim Verschieben oder Importieren eines virtuellen Computers auf einen Host mit Hyper-V unter Windows 10 vom einem Host unter Windows 8.1 wird die Konfigurationsdatei des virtuellen Computers nicht automatisch aktualisiert. Dadurch kann die virtuelle Maschine zurück auf einem Host unter Windows 8.1 verschoben werden. Sie haben erst dann Zugriff auf die neuen Features für virtuelle Computer, nachdem Sie die Konfigurationsversion des virtuellen Computers manuell aktualisiert haben.

Die VM-Konfigurationsversion gibt an, mit welcher Version von Hyper-V die Konfiguration, der gespeicherte Zustand und die Snapshotdateien des virtuellen Computers kompatibel sind. Virtuelle Computer mit Konfigurationsversion 5 sind kompatibel mit Windows 8.1 und können unter Windows 8.1 und Windows 10 ausgeführt werden. Virtuelle Computer mit Konfigurationsversion 6 sind kompatibel mit Windows 10 und können nicht unter Windows 8.1 ausgeführt werden.

Es ist nicht notwendig, alle virtuellen Computer gleichzeitig zu aktualisieren. Sie können bei Bedarf auch bestimmte virtuelle Computer aktualisieren. Sie haben jedoch erst dann Zugriff auf die neuen Features für virtuelle Computer, nachdem Sie die Konfigurationsversion jedes virtuellen Computers manuell aktualisiert haben.


----------------

*Wichtig:*

• Nach der Aktualisierung der VM-Konfigurationsversion können Sie den virtuellen Computer nicht auf einen Host unter Windows 8.1 verschieben.

• Sie können die VM-Konfigurationsversion nicht von Version 6 auf Version 5 herabstufen.

• Zum Aktualisieren der VM-Konfiguration müssen Sie den virtuellen Computer ausschalten.

• Nach der Aktualisierung verwendet der virtuelle Computer das neue Konfigurationsdateiformat. Weitere Informationen finden Sie unter „Neues VM-Konfigurationsdateiformat“.

--------






## Was geschieht, wenn ich die Version eines virtuellen Computers aktualisiere?

Wenn Sie Konfigurationsversion eines virtuellen Computers manuell auf Version 6.x aktualisieren, ändern Sie die Dateistruktur, die zum Speichern der VM-Konfigurations- und Prüfpunktdateien verwendet wird.

Aktualisierte virtuelle Computer verwenden ein neues Konfigurationsdateiformat, das entworfen wurde, um die Effizienz beim Lesen und Schreiben von VM-Konfigurationsdaten zu steigern. Das Upgrade verringert auch das Potenzial von Datenbeschädigungen bei einem Speicherausfall.

Die folgende Tabelle enthält die Speicherorte der Binärdateien und Informationen zu den Erweiterungen eines aktualisierten virtuellen Computers.

| **VM-Konfigurations- und Prüfpunktdateien (Version 6.x)**| **Beschreibung**| |
|:---------------|:----------------||
| **VM-Konfiguration**| Konfigurationsinformationen werden in einem binären Format mit der Erweiterung VMCX gespeichert.| |
| **VM-Laufzeitzustand**| Laufzeitzustandsdaten werden in einem binären Format mit der Erweiterung VMRS gespeichert.| |
| **Virtuelle Festplatte des virtuellen Computers**| Die virtuellen Festplattendateien für den virtuellen Computermit den Dateinamenerweiterungen VHD und VHDX.| |
| **Automatische virtuelle Festplattendateien**| Die differenzierenden Datenträgerdateien für die VM-Prüfpunktemit der Dateinamenerweiterung AVHDX.| |
| **Prüfpunktdateien**| Prüfpunkte werden in mehreren Prüfpunktdateien gespeichert.Jeder Prüfpunkt erstellt eine Konfigurationsdatei und eine Datei mit dem Laufzeitzustand.Prüfpunktdateien haben die Dateinamenerweiterungen VMRS und VMCX.Diese neue Dateiformate werden auch für Produktions- und Standardprüfpunkte verwendet.| |

Nach dem Upgrade der VM-Konfigurationsversion auf Version 6.x ist ein Wechsel von Version 6.x zur Version 5 nicht mehr möglich.

Zum Aktualisieren der VM-Konfiguration muss der virtuelle Computer ausgeschaltet sein.

Die folgende Tabelle enthält die standardmäßigen Dateispeicherorte für neue und aktualisierte virtuelle Computer.

| **VM-Dateien (Version 6.x)**| **Beschreibung**| |
|:-----|:-----||
| **VM-Konfigurationsdatei**| C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines| |
| **VM-Laufzeitzustands-Datei**| C:\ProgramData\Microsoft\Windows\Hyper-V\Virtual Machines| |
| **Prüfpunktdateien (.vmrs, .vmcx)**| C:\ProgramData\Microsoft\Windows\Snapshots| |
| **Virtuelle Festplattendatei (.vhd/.vhdx)**| C:\Users\Public\Documents\Hyper-V\Virtual Hard Disks| |
| **Automatische virtuelle Festplattendateien (.avhdx)**| C:\Users\Public\Documents\Hyper-V\Virtual Hard Disks| |








