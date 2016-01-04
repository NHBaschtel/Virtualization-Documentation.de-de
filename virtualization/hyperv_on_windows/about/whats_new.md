# Neuerungen bei Hyper-V unter Windows 10

In diesem Thema werden die neuen und geänderten Funktionen in Hyper-V unter Windows 10® vorgestellt.

## Windows PowerShell Direct

Es gibt nun eine einfache und zuverlässige Möglichkeit vom Hostbetriebssystem aus, Windows PowerShell-Befehle in einem virtuellen Computer auszuführen. Dafür gelten keine Netzwerk- oder Firewall- bzw. speziellen Konfigurationsanforderungen. 
Die Funktionalität ist unabhängig von Ihrer Konfiguration der Remoteverwaltung sichergestellt. Voraussetzung ist das Ausführen von Windows 10 oder Windows Server Technical Preview auf dem Host und als Gastbetriebssystem des virtuellen Computers.

Führen Sie zum Einrichten einer PowerShell Direct-Sitzung einen der folgenden Befehle aus:

``` PowerShell
Enter-PSSession -VMName VMName
Invoke-Command -VMName VMName -ScriptBlock { commands }
```

Derzeit nutzen Hyper-V-Administratoren zwei Kategorien von Tools für das Verbinden mit einem virtuellen Computer auf dem Hyper-V-Host:
- Remoteverwaltungstools wie PowerShell oder Remotedesktop
- Hyper-V-Verbindung mit virtuellen Computern (VMConnect)

Beide dieser Technologien funktionieren gut, haben aber Vor- und Nachteile, sobald die Hyper-V-Bereitstellung wächst. VMConnect ist zuverlässig, kann aber schwierig zu automatisieren sein. Remote-PowerShell ist leistungsstark, kann aber schwierig einzurichten und zu verwalten sein.

Windows PowerShell Direct kombiniert eine leistungsfähige Skripterstellung und Automatisierung mit der Einfachheit von VMConnect. Da Windows PowerShell Direct zwischen Host und virtuellem Computer ausgeführt wird, ist keine Netzwerkverbindung und auch nicht das Aktivieren der Remoteverwaltung erforderlich. Sie benötigen Gastanmeldeinformationen, um sich beim virtuellen Computer anzumelden.

### Anforderungen

- Sie müssen auf einem Host mit Windows 10 oder Windows Server Technical Preview mit virtuellen Computern verbunden sein, auf denen Windows 10 oder Windows Server Technical Preview als Gastbetriebssystem ausgeführt wird.
- Sie müssen auf dem Host mit den Anmeldeinformationen eines Hyper-V-Administrators angemeldet sein.
- Sie benötigen Benutzeranmeldeinformationen für den virtuellen Computer.
- Der virtuelle Computer, mit dem Sie eine Verbindung herstellen möchten, muss gestartet sein und ausgeführt werden.


## Hinzufügen und Entfernen von Netzwerkadaptern und Arbeitsspeicher bei laufendem Systembetrieb

Sie können nun einen Netzwerkadapter ohne Ausfallzeit bei laufendem Betrieb des virtuellen Computers hinzufügen und entfernen. Dies gilt für virtuelle Computer der Generation 2 unter Windows und Linux-Betriebssystemen.

Sie können auch die Größe des Arbeitsspeichers, der einen virtuellen Computer zugewiesen ist, bei laufendem Betrieb anpassen, auch wenn „Dynamischer Arbeitsspeicher“ nicht aktiviert ist. Dies funktioniert für virtuelle Computer der Generation 1 und 2.

## Produktionsprüfpunkte

Produktionsprüfpunkte ermöglichen das einfache Erstellen zeitpunktbezogener Images eines virtuellen Computers, die später auf eine Weise wiederhergestellt werden können, die für alle Produktionsworkloads vollständig unterstützt wird. Dies erfolgt mithilfe von Sicherungstechnologie innerhalb des Gastbetriebssystems zum Erstellen des Prüfpunkts anstatt mit Technologie zum Speichern des Zustands. Für Produktionsprüfpunkte wird der Volumemomentaufnahme-Dienst (VSS) innerhalb von virtuellen Windows-Computern verwendet. Virtuelle Linux-Computer leeren ihre Dateisystempuffer zum Erstellen eines mit dem Dateisystem konsistenten Prüfpunkts. Wenn Sie Prüfpunkte mithilfe der Technologie für den gespeicherten Zustand erstellen möchten, können Sie weiterhin Standardprüfpunkte für den virtuellen Computer erstellen.


> **Wichtig:** Standardmäßig werden für neue virtuelle Computer Produktionsprüfpunkten mit Fallback auf Standardprüfpunkte erstellt.


## Verbesserungen an Hyper-V-Manager

- **Unterstützung alternativer Anmeldeinformationen**: Sie können nun einen anderen Satz von Anmeldeinformationen im Hyper-V-Manager verwenden, wenn Sie eine Verbindung mit einem anderen Windows 10 Technical Preview-Remotehost herstellen. Sie können diese Anmeldeinformationen auch speichern, damit es später einfacher ist, sich anzumelden.

- **Verwaltung niedrigerer Versionen**: Sie können mit dem Hyper-V-Manager jetzt mehr Versionen von Hyper-V verwalten als bisher. Mit Hyper-V-Manager in Windows 10 Technical Preview können Sie Computer mit ausgeführtem Hyper-V unter Windows Server 2012, Windows 8, Windows Server 2012 R2 und Windows 8.1 verwalten.

- **Verwaltungsprotokoll aktualisiert**: Hyper-V-Manager wurde so aktualisiert, dass die Kommunikation mit Hyper-V-Remotehosts mithilfe des Protokolls WS-MAN möglich ist, das die CredSSP-, Kerberos- oder NTLM-Authentifizierung zulässt. Wenn Sie CredSSP für das Verbinden mit einem Hyper-V-Remotehost verwenden, können Sie eine Livemigration ausführen, ohne zunächst die eingeschränkte Delegierung in Active Directory aktivieren zu müssen. Auf WS-MAN basierende Infrastruktur vereinfacht auch die Konfiguration, die zum Aktivieren eines Hosts für die Remoteverwaltung erforderlich ist. WS-MAN stellt über Port 80 eine Verbindung her, der standardmäßig geöffnet ist.


## Der verbundene Standbymodus funktioniert

Wenn Hyper-V auf einem Computer aktiviert ist, der den Energiemodus „Always On/Always Connected“ verwendet, ist der Energiezustand „Verbundener Standbymodus“ nun verfügbar.

Unter Windows 8 und 8.1 verursachte Hyper-V, dass Computer, die im Energiemodus „Always On/Always Connected“ verwendet haben, nicht in den Energiesparmodus wechselten. Siehe diesen [KB-Artikel](
https://support.microsoft.com/en-us/kb/2973536), in dem Sie eine vollständige Beschreibung finden.


## Linux – Sicherer Start

Weitere Linux-Betriebssysteme auf virtuellen Computern der Generation 2 können jetzt mit aktivierter Option „Sicherer Start“ starten. Ubuntu 14.04 und höher und SUSE Linux Enterprise Server 12 sind für den sicheren Start auf Hosts aktiviert, auf denen die Technical Preview-Version ausgeführt wird. Bevor Sie den virtuellen Computer zum ersten Mal starten, müssen Sie angeben, dass der virtuelle Computer die Zertifizierungsstelle „Microsoft UEFI“ verwenden soll. Geben Sie an einer Windows PowerShell-Eingabeaufforderung mit erhöhten Rechten Folgendes ein:

    Set-VMFirmware vmname -SecureBootTemplate MicrosoftUEFICertificateAuthority

Weitere Informationen zum Ausführen virtueller Computer mit Linux in Hyper-V finden Sie unter [Virtuelle Computer mit Linux und FreeBSD in Hyper-V](http://technet.microsoft.com/library/dn531030.aspx).


## VM-Konfigurationsversion

Beim Verschieben oder Importieren eines virtuellen Computers auf einen Host mit Hyper-V unter Windows 10 vom einem Host unter Windows 8.1 wird die Konfigurationsdatei des virtuellen Computers nicht automatisch aktualisiert. Dadurch kann die virtuelle Maschine zurück auf einem Host unter Windows 8.1 verschoben werden. Sie haben erst dann Zugriff auf die neuen Features für virtuelle Computer, nachdem Sie die Konfigurationsversion des virtuellen Computers manuell aktualisiert haben.

Die VM-Konfigurationsversion gibt an, mit welcher Version von Hyper-V die Konfiguration, der gespeicherte Zustand und die Snapshotdateien des virtuellen Computers kompatibel sind. Virtuelle Computer mit Konfigurationsversion 5 sind kompatibel mit Windows 8.1 und können unter Windows 8.1 und Windows 10 ausgeführt werden. Virtuelle Computer mit Konfigurationsversion 6 sind kompatibel mit Windows 10 und können nicht unter Windows 8.1 ausgeführt werden.

### Überprüfen der Konfigurationsversion

Führen Sie an einer Eingabeaufforderung mit erhöhten Rechten den folgenden Befehl aus:

``` PowerShell
Get-VM * | Format-Table Name, Version
```

### Upgrade der Konfigurationsversion

Führen Sie an einer Windows PowerShell-Eingabeaufforderung mit erhöhten Rechten einen der folgenden Befehle aus:

``` PowerShell
Update-VmConfigurationVersion <vmname>
```

Oder

``` PowerShell
Update-VmConfigurationVersion <vmobject>
```

*Wichtig:*
- Nach der Aktualisierung der VM-Konfigurationsversion können Sie den virtuellen Computer nicht auf einen Host unter Windows 8.1 verschieben.
- Sie können die VM-Konfigurationsversion nicht von Version 6 auf Version 5 herabstufen.
- Zum Aktualisieren der VM-Konfiguration müssen Sie den virtuellen Computer ausschalten.
- Nach der Aktualisierung verwendet der virtuelle Computer das neue Konfigurationsdateiformat. Weitere Informationen finden Sie unter „Neues VM-Konfigurationsdateiformat“.


## Konfigurationsdateiformat

Virtuelle Computer verwenden nun ein neues Konfigurationsdateiformat, das entworfen wurde, um die Effizienz beim Lesen und Schreiben von VM-Konfigurationsdaten zu steigern. Es wurde auch entwickelt, um potenzielle Datenverluste bei einem Speicherausfall zu verringern. Die neuen Konfigurationsdateien haben die Erweiterung VMCX für VM-Konfigurationsdaten und VMRS für Daten zum Laufzeitzustand.


> **Wichtig:** Die VMCX-Datei hat ein binäres Format. Ein direktes Bearbeiten der VMCX- oder VMRS-Datei wird nicht unterstützt.

## Integrationsdienste über Windows Update

Updates für die Integrationsdienste für Windows-Gastbetriebssysteme werden jetzt über Windows Update verteilt.

Integrationskomponenten (auch als Integrationsdienste bezeichnet) sind eine Gruppe synthetischer Treiber, die einem virtueller Computer die Kommunikation mit dem Hostbetriebssystem ermöglichen. Mit ihrer Hilfe werden Dienste wie die Zeitsynchronisierung und das Kopieren von Gastdateien gesteuert. Wir haben im vergangenen Jahr mit Kunden über die Installation von Integrationskomponenten und deren Update gesprochen und erfahren, dass diese während des Upgradeprozesses eine große Schwachstelle darstellen.


In der Vergangenheit wurden alle neuen Versionen von Hyper-V mit neuen Integrationskomponenten versehen. Bei einem Upgrade des Hyper-V-Hosts mussten die Integrationskomponenten in den virtuellen Computern ebenfalls aktualisiert werden. Die neuen Integrationskomponenten wurden dem Hyper-V-Host hinzugefügt und anschließend mithilfe von „vmguest.iso“ in den virtuellen Computern installiert. Dieser Prozess erforderte den Neustart des virtuellen Computers und konnte nicht in einem Batch mit anderen Windows-Updates zusammengefasst werden. Da der Hyper-V-Administrator „vmguest.iso“ bereitstellen und der VM-Administrator diese Datei installieren musste, benötigte der Hyper-V-Administrator für das Upgrade von Integrationskomponenten Administratorrechte für die virtuellen Computer, was nicht immer der Fall ist.
　　


Ab Windows 10 werden sämtliche Integrationskomponenten zusammen mit anderen wichtigen Updates virtuellen Computern über Windows Update zur Verfügung gestellt.


Derzeit sind Updates für virtuelle Computer mit diesen Betriebssystemen verfügbar:
*  Windows Server 2012
*  Windows Server 2008 R2
*  Windows 8
*  Windows 7

Der virtuelle Computer muss mit Windows Update oder einem WSUS-Server verbunden sein. In Zukunft werden Updates von Integrationskomponenten über eine Kategorie-ID verfügen. Für diese Version sind sie als KBs aufgeführt.

Weitere Informationen, wie die Anwendbarkeit ermittelt wird, finden Sie in diesem [Blogbeitrag](http://blogs.technet.com/b/virtualization/archive/2014/11/24/integration-components-how-we-determine-windows-update-applicability.aspx).


In diesem [Blogbeitrag](http://blogs.msdn.com/b/virtual_pc_guy/archive/2014/11/12/updating-integration-components-over-windows-update.aspx) finden Sie eine ausführliche exemplarische Vorgehensweise zur Installation von Integrationsdiensten.


> **Wichtig:** Die ISO-Imagedatei „vmguest.iso“ wird nicht mehr zum Aktualisieren von Integrationskomponenten benötigt. Sie ist nicht in Hyper-V unter Windows 10 enthalten.


## Nächster Schritt

[Exemplarische Vorgehensweise für Hyper-V unter Windows 10](..\quick_start\walkthrough.md)



