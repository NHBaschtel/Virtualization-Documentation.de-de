---
redirect_url: https://msdn.microsoft.com/virtualization/hyperv_on_windows/windows_welcome
---

## Hinzufügen und Entfernen von Netzwerkadaptern und Arbeitsspeicher bei laufendem Systembetrieb

Sie können nun einen Netzwerkadapter ohne Ausfallzeit bei laufendem Betrieb des virtuellen Computers hinzufügen und entfernen. Dies gilt für virtuelle Computer der Generation 2 unter den Betriebssystemen Windows und Linux.

Sie können auch die Größe des Arbeitsspeichers, der einen virtuellen Computer zugewiesen ist, bei laufendem Betrieb anpassen, auch wenn „Dynamischer Arbeitsspeicher“ nicht aktiviert ist. Dies funktioniert für virtuelle Computer der Generation 1 und 2.

## Produktionsprüfpunkte

Produktionsprüfpunkte ermöglichen das einfache Erstellen zeitpunktbezogener Images eines virtuellen Computers, die später auf eine Weise wiederhergestellt werden können, die für alle Produktionsworkloads vollständig unterstützt wird. Dies erfolgt mithilfe von Sicherungstechnologie innerhalb des Gastbetriebssystems zum Erstellen des Prüfpunkts anstatt mit Technologie zum Speichern des Zustands. Für Produktionsprüfpunkte wird der Volumemomentaufnahme-Dienst (VSS) innerhalb von virtuellen Windows-Computern verwendet. Virtuelle Linux-Computer leeren ihre Dateisystempuffer zum Erstellen eines mit dem Dateisystem konsistenten Prüfpunkts. Wenn Sie Prüfpunkte mithilfe der Technologie für den gespeicherten Zustand erstellen möchten, können Sie weiterhin Standardprüfpunkte für den virtuellen Computer erstellen.


> <g id="1" ctype="x-strong">Wichtig:</g> Standardmäßig werden für neue virtuelle Computer Produktionsprüfpunkten mit Fallback auf Standardprüfpunkte erstellt.


## Verbesserungen an Hyper-V-Manager

- <g id="1" ctype="x-strong">Unterstützung alternativer Anmeldeinformationen</g>: Sie können nun einen anderen Satz von Anmeldeinformationen im Hyper-V-Manager verwenden, wenn Sie eine Verbindung mit einem anderen Windows 10 Technical Preview-Remotehost herstellen. Sie können diese Anmeldeinformationen auch speichern, damit es später einfacher ist, sich anzumelden.

- <g id="1" ctype="x-strong">Verwaltung niedrigerer Versionen</g>: Sie können mit dem Hyper-V-Manager jetzt mehr Versionen von Hyper-V verwalten als bisher. Mit Hyper-V-Manager in Windows 10 Technical Preview können Sie Computer verwalten, die Hyper-V unter Windows Server 2012, Windows 8, Windows Server 2012 R2 und Windows 8.1 ausführen.

- <g id="1" ctype="x-strong">Verwaltungsprotokoll aktualisiert</g>: Hyper-V-Manager wurde so aktualisiert, dass die Kommunikation mit Hyper-V-Remotehosts mithilfe des Protokolls WS-MAN möglich ist, das die CredSSP-, Kerberos- oder NTLM-Authentifizierung zulässt. Wenn Sie CredSSP für das Verbinden mit einem Hyper-V-Remotehost verwenden, können Sie eine Livemigration ausführen, ohne zunächst die eingeschränkte Delegierung in Active Directory aktivieren zu müssen. Auf WS-MAN basierende Infrastruktur vereinfacht auch die Konfiguration, die zum Aktivieren eines Hosts für die Remoteverwaltung erforderlich ist. WS-MAN stellt über Port 80 eine Verbindung her, der standardmäßig geöffnet ist.


## Kompatibilität mit verbundenem Standbymodus

Wenn Hyper-V auf einem Computer aktiviert ist, der den Energiemodus „Always On/Always Connected“ verwendet, ist der Energiezustand „Verbundener Standbymodus“ nun verfügbar.

Unter Windows 8 und 8.1 verursachte Hyper-V, dass Computer, die im Energiemodus „Always On/Always Connected“ verwendet haben, nicht in den Energiesparmodus wechselten. Siehe diesen [KB-Artikel](
https://support.microsoft.com/en-us/kb/2973536), in dem Sie eine vollständige Beschreibung finden.


## Linux – Sicherer Start

Weitere Linux-Betriebssysteme auf virtuellen Computern der Generation 2 können jetzt mit aktivierter Option „Sicherer Start“ starten. Ubuntu 14.04 und höher und SUSE Linux Enterprise Server 12 sind für den sicheren Start auf Hosts aktiviert, auf denen die Technical Preview-Version ausgeführt wird. Bevor Sie den virtuellen Computer zum ersten Mal starten, müssen Sie angeben, dass der virtuelle Computer die Zertifizierungsstelle „Microsoft UEFI“ verwenden soll. Geben Sie an einer Windows PowerShell-Eingabeaufforderung mit erhöhten Rechten Folgendes ein:

    Set-VMFirmware [-VMName] <VMName> [-SecureBootTemplate] <MicrosoftUEFICertificateAuthority>

Weitere Informationen zum Ausführen virtueller Computer mit Linux in Hyper-V finden Sie unter <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Linux and FreeBSD Virtual Machines on Hyper-V</g><g id="2CapsExtId3" ctype="x-title"></g></g> (Virtuelle Linux- und FreeBSD-Computer in Hyper-V).


## VM-Konfigurationsversion

Beim Verschieben oder Importieren eines virtuellen Computers auf einen Host mit Hyper-V unter Windows 10 von einem Host unter Windows 8.1 wird die Konfigurationsdatei des virtuellen Computers nicht automatisch aktualisiert. Dadurch kann die virtuelle Maschine zurück auf einem Host unter Windows 8.1 verschoben werden. Sie haben erst dann mit Ihrem virtuellen Computer Zugriff auf die neuen Hyper-V-Features, wenn Sie die Konfigurationsversion des virtuellen Computers manuell aktualisiert haben.

Die VM-Konfigurationsversion gibt an, mit welcher Version von Hyper-V die Konfiguration, der gespeicherte Zustand und die Snapshotdateien des virtuellen Computers kompatibel sind. Virtuelle Computer mit Konfigurationsversion 5 sind kompatibel mit Windows 8.1 und können unter Windows 8.1 und Windows 10 ausgeführt werden. Virtuelle Computer mit Konfigurationsversion 6 sind kompatibel mit Windows 10 und können nicht unter Windows 8.1 ausgeführt werden.

### Überprüfen der Konfigurationsversion

Führen Sie an einer Eingabeaufforderung mit erhöhten Rechten den folgenden Befehl aus:

``` PowerShell
Get-VM * | Format-Table Name, Version
```

### Upgrade der Konfigurationsversion

Führen Sie an einer Windows PowerShell-Eingabeaufforderung mit erhöhten Rechten einen der folgenden Befehle aus:

``` 
Update-VmConfigurationVersion <VMName>
```

oder

``` 
Update-VmConfigurationVersion <VMObject>
```

> <g id="1" ctype="x-strong">Wichtig:</g>

- Nach der Aktualisierung der VM-Konfigurationsversion können Sie den virtuellen Computer nicht auf einen Host unter Windows 8.1 verschieben.
- Sie können die VM-Konfigurationsversion nicht von Version 6 auf Version 5 herabstufen.
- Zum Aktualisieren der VM-Konfiguration müssen Sie den virtuellen Computer ausschalten.
- Nach der Aktualisierung verwendet der virtuelle Computer das neue Konfigurationsdateiformat. Weitere Informationen finden Sie unter <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Konfigurationsdateiformat</g><g id="2CapsExtId3" ctype="x-title"></g></g>.


## <g id="1" ctype="x-html"></g><g id="2" ctype="x-html"></g>Konfigurationsdateiformat

Virtuelle Computer verwenden nun ein neues Konfigurationsdateiformat, das entworfen wurde, um die Effizienz beim Lesen und Schreiben von VM-Konfigurationsdaten zu steigern. Es wurde auch entwickelt, um potenzielle Datenverluste bei einem Speicherausfall zu verringern. Die neuen Konfigurationsdateien haben die Erweiterung VMCX für VM-Konfigurationsdaten und VMRS für Daten zum Laufzeitzustand.

> <g id="1" ctype="x-strong">Wichtig:</g> Die VMCX-Datei hat ein binäres Format. Ein direktes Bearbeiten der VMCX- oder VMRS-Datei wird nicht unterstützt.





<!--HONumber=May16_HO1-->


