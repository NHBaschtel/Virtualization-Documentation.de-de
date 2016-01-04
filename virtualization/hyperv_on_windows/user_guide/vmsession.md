# Verwalten von Windows mit PowerShell Direct

Mit PowerShell Direct können Sie einen virtuellen Computer mit Windows 10 oder Windows Server Technical Preview auf einem Hyper-V-Host mit Windows 10 oder Windows Server Technical Preview remote verwalten. PowerShell Direct ermöglicht die PowerShell-Verwaltung innerhalb eines virtuellen Computers unabhängig von der Netzwerkkonfiguration oder den Remoteverwaltungseinstellungen des Hyper-V-Hosts oder virtuellen Computers. Dies erleichtert Hyper-V-Administratoren die Automatisierung und Erstellung von Skripts zur Verwaltung und Konfiguration virtueller Computer.

Es gibt zwei Möglichkeiten, PowerShell Direct auszuführen:
* Als interaktive Sitzung: [Wechseln Sie zu diesem Abschnitt](vmsession.md#create-and-exit-an-interactive-powershell-session), um eine PowerShell Direct-Sitzung mithilfe von „PSSession“-Cmdlets einzurichten und zu beenden.
* Ausführen mehrerer Befehle oder eines Skripts: [Wechseln Sie zu diesem Abschnitt](vmsession.md#run-a-script-or-command-with-invoke-command), um mit dem Cmdlet „Invoke-Command“ ein Skript oder einen Befehl auszuführen.


## Anforderungen

**Betriebssystemanforderungen:**
* Das Hostbetriebssystem muss Windows 10, Windows Server Technical Preview oder eine höhere Version sein.
* Das VM-Betriebssystem muss Windows 10, Windows Server Technical Preview oder eine höhere Version sein.

Wenn Sie ältere virtuelle Computer verwalten, verwenden Sie die Verbindung mit virtuellen Computern (VMConnect), oder [konfigurieren Sie ein virtuelles Netzwerk für den virtuellen Computer](http://technet.microsoft.com/library/cc816585.aspx).

So richten Sie eine PowerShell Direct-Sitzung auf einem virtuellen Computer ein
* Der virtuelle Computer muss lokal auf dem Host ausgeführt und gestartet werden.
* Sie müssen am Hostcomputer als Hyper-V-Administrator angemeldet sein.
* Sie müssen gültige Anmeldeinformationen für den virtuellen Computer angeben.

## Erstellen und Beenden einer interaktiven PowerShell-Sitzung

1. Öffnen Sie Windows PowerShell auf dem Hyper-V-Host als Administrator.

3. Führen Sie einen der folgenden Befehle aus, um eine Sitzung mit dem Namen des virtuellen Computers oder seiner GUID einzurichten:
``` PowerShell
Enter-PSSession -VMName <VMName>
Enter-PSSession -VMGUID <VMGUID>
```

4. Führen Sie alle erforderlichen Befehle aus. Diese Befehle werden auf dem virtuellen Computer ausgeführt, auf dem Sie die Sitzung erstellt haben.
5. Wenn Sie fertig sind, führen Sie den folgenden Befehl zum Schließen der Sitzung aus:
``` PowerShell
Exit-PSSession 
```

> Hinweis: Wenn für die Sitzung keine Verbindung hergestellt werden kann, vergewissern Sie sich, dass Sie Anmeldeinformationen für den virtuellen Computer, mit dem Sie sich verbinden möchten, und nicht für den Hyper-V-Host verwenden.

Weitere Informationen zu diesen Cmdlets finden Sie unter [Enter-PSSession](http://technet.microsoft.com/library/hh849707.aspx) und [Exit-PSSession](http://technet.microsoft.com/library/hh849743.aspx).

## Ausführen eines Skripts oder Befehls mit „Invoke-Command“

Sie können mithilfe des Cmdlets [Invoke-Command](http://technet.microsoft.com/library/hh849719.aspx) einen vordefinierten Satz von Befehlen auf dem virtuellen Computer ausführen. Es folgt ein Beispiel der Nutzung des Cmdlets „Invoke-Command“, wobei „PSTest“ der Name des virtuellen Computers ist und sich das auszuführende Skript (foo.ps1) im Skriptordner auf Laufwerk C:/ befindet:

 ``` PowerShell
 Invoke-Command -VMName PSTest -FilePath C:\script\foo.ps1 
 ```

Zum Ausführen eines einzelnen Befehls verwenden Sie den Parameter **-ScriptBlock**:

 ``` PowerShell
 Invoke-Command -VMName PSTest -ScriptBlock { cmdlet } 
 ```

## Problembehandlung

PowerShell Direct zeigt eine kleine Menge von Fehlermeldungen an. Es folgen die häufigsten davon, verschiedene Ursachen und Tools für die Untersuchung von Problemen.

### Fehler: Eine Remotesitzung wurde möglicherweise getrennt.

Fehlermeldung:
```
An error has occured which Windows PowerShell cannot handle.  A remote session might have ended. 
```

Mögliche Ursachen:
* Der virtuelle Computer wird nicht ausgeführt.
* Das Gastbetriebssystem unterstützt PowerShell Direct nicht (siehe die [Anforderungen](#Requirements)).
* PowerShell ist noch nicht auf dem Gast verfügbar.
    * Der Startvorgang des Betriebssystems ist noch nicht abgeschlossen.
    * Das Betriebssystem kann nicht ordnungsgemäß starten.
    * Bei einigen Ereignissen zur Startzeit sind Benutzereingaben erforderlich.
* Die Gastanmeldeinformationen konnten nicht überprüft werden.

Mit dem Cmdlet [Get-VM](http://technet.microsoft.com/library/hh848479.aspx) können Sie überprüfen, ob die von Ihnen verwendeten Anmeldeinformationen die Rolle „Hyper-V-Administrator“ enthalten und welche VMs lokal auf dem Host ausgeführt werden und gestartet wurden.

## Beispiele

Sehen Sie sich Beispiele auf [GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93) an.

Unter [PowerShell Snippets](../develop/powershell_snippets.md) finden Sie zahlreiche Beispiel zur Nutzung von PowerShell Direct in Ihrer Umgebung sowie Tipps und Tricks zum Schreiben von Hyper-V-Skripts mit PowerShell.



