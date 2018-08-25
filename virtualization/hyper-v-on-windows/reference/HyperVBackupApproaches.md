# <a name="hyper-v-backup-approaches"></a>Hyper-V-Backup-Konzepte
Hyper-V können Sie eine Sicherung virtuellen Computern von das Hostbetriebssystem, ohne dass benutzerdefiniertes backup-Software auf dem virtuellen Computer ausgeführt werden müssen.  Es gibt verschiedene Methoden, die für Entwickler, je nach ihren Anforderungen Nutzung zur Verfügung stehen.
## <a name="hyper-v-vss-writer"></a>Hyper-V-VSS Writer
Hyper-V implementiert einen VSS Writer in allen Versionen von Windows Server, auf dem Hyper-V unterstützt wird.  Dieser VSS Writer ermöglicht Entwicklern die vorhandene VSS-Infrastruktur backup virtuellen Computern zu nutzen.  Es ist jedoch für kleinen Maßstab Sicherungsvorgänge entwickelt, in dem alle virtuellen Computer auf einem Server gleichzeitig gesichert werden.

Diese Architektur – ein besseres Verständnis finden Sie in dieser Präsentation:https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-V-WMI-basierte Sicherung
Starten in Windows Server 2016, gestartet Hyper-V-Sicherung über die Hyper-V-WMI-API unterstützen.  Dieser Ansatz verwendet weiterhin VSS innerhalb der virtuellen Maschine um zu Sicherungszwecken, aber nicht mehr in das Hostbetriebssystem verwendet VSS.  Stattdessen wird eine Kombination von Verweis Punkt und ausfallsichere Versionsvergleich (RCT) verwendet, um können Entwickler die Informationen zum Zugriff auf virtuellen Computern effizient gesichert.  Dieser Ansatz ist skalierbarer als die Verwendung von VSS in den Host, jedoch nur und höher auf Windows Server 2016 verfügbar ist.

Diese Architektur – ein besseres Verständnis finden Sie in dieser Präsentation:https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

Es gibt auch ein Beispiel zur Verwendung dieser APIs verfügbar hier:https://msconfiggallery.cloudapp.net/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Methoden zum Lesen von Sicherungen von WMI-basierten Sicherung
Beim Erstellen von virtuellen Computer Sicherungen Verwenden von WMI für Hyper-V, gibt es drei Methoden zum Lesen von der tatsächlichen Daten aus der Sicherung.  Jeder weist einzigartige Vorteile und Nachteile.
### <a name="wmi-export"></a>WMI-Export
Entwickler können die Sicherungsdaten über die Hyper-V-WMI-Schnittstellen (wie im obigen Beispiel verwendet) exportieren.  Hyper-V werden die Änderungen in einer virtuellen Festplatte kompilieren und kopieren Sie die Datei am angeforderten Speicherort.  Diese Methode ist einfach zu verwenden, funktioniert für alle Szenarien und ist remotefähig.  Die virtuelle Festplatte häufig generiert erstellt jedoch eine große Menge von Daten über das Netzwerk übertragen.
### <a name="win32-apis"></a>Win32-APIs
Entwickler können die SetVirtualDiskInformation GetVirtualDiskInformation und QueryChangesVirtualDisk-APIs in der virtuellen Festplatte Win32-API legen Sie wie dokumentiert hier: https://docs.microsoft.com/en-us/windows/desktop/api/_vhd/ Beachten Sie, dass für die Verwendung dieser APIs Hyper-V WMI noch werden, um den Verweis zu erstellen verwendet muss Punkte auf zugeordneten virtuellen Computern.  Diese Win32-APIs ermöglichen Sie dann für den effizienten Zugriff auf die Daten des gesicherten virtuellen Computers.  Win32-APIs müssen verschiedene Einschränkungen:
*   Sie können nur lokal zugegriffen werden
*   Die nicht Support Lesen von Daten aus freigegebenen virtuelle Festplatte-Dateien
*   Diese zurückgeben Datenadressen, die sich die interne Struktur der virtuellen Festplatte beziehen

### <a name="remote-shared-virtual-disk-protocol"></a>Freigegebene Remote-Protokoll von virtuellen Datenträgern
Wenn ein Entwickler effizient Sicherungsdaten Zugriff auf Informationen aus einer Datei freigegebenen virtuelle Festplatte muss – müssen sie schließlich Remote freigegebenen virtuellen Datenträger Protokoll verwendet werden.  Dieses Protokoll wird hier beschrieben:https://msdn.microsoft.com/en-us/library/dn393384.aspx
