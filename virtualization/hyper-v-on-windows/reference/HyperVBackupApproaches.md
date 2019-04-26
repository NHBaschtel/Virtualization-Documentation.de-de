# <a name="hyper-v-backup-approaches"></a>Hyper-V-Backup-Konzepte
Hyper-V können Sie eine Sicherung virtuelle Computer von Host-Betriebssystem, ohne dass benutzerdefinierte Sicherungssoftware innerhalb des virtuellen Computers ausgeführt werden müssen.  Es gibt verschiedene Methoden, die für Entwickler zur nutzen Sie je nach ihren Anforderungen verfügbar sind.
## <a name="hyper-v-vss-writer"></a>Hyper-V VSS Writer
Hyper-V implementiert einen VSS Writer auf allen Versionen von Windows Server, in denen Hyper-V unterstützt wird.  Diese VSS Writer kann Entwickler die vorhandene VSS-Infrastruktur für backup virtuelle Maschinen zu nutzen.  Es ist jedoch für kleine Skalierung backup-Vorgänge konzipiert, in denen alle virtuellen Computer auf einem Server gleichzeitig gesichert werden.

Diese Architektur – ein besseres Verständnis finden Sie in dieser Präsentation:https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-V-WMI-basierte Sicherung
Ab Windows Server 2016, gestartet Hyper-V-Backup über die Hyper-V-WMI-API unterstützt.  Dieser Ansatz weiterhin VSS innerhalb des virtuellen Computers um zu Sicherungszwecken nutzt, aber nicht mehr VSS in der Host-Betriebssystem verwendet.  Stattdessen, eine Kombination von Referenzpunkten und robustes Änderungsprotokoll (RCT) verwendet, um Entwicklern den Zugriff auf Informationen zu ermöglichen, virtuelle Computer auf effiziente Weise gesichert.  Dieser Ansatz ist skalierbarer als die Verwendung von VSS auf dem Host, jedoch nur auf Windows Server 2016 verfügbar und höher.

Diese Architektur – ein besseres Verständnis finden Sie in dieser Präsentation:https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

Es ist auch ein Beispiel zur Verwendung dieser APIs verfügbar hier:https://msconfiggallery.cloudapp.net/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Methoden zum Lesen von Sicherungen von WMI-basiertes Backup
Beim virtuellen Computer Sicherungen mit Hyper-V-WMI zu erstellen, gibt es drei Methoden zum Lesen von der tatsächlichen Daten aus der Sicherung.  Jede Komponente eindeutige vor- und Nachteile.
### <a name="wmi-export"></a>WMI-Export
Entwickler können die backup-Daten über die Hyper-V-WMI-Schnittstellen (wie im obigen Beispiel verwendet) exportieren.  Hyper-V wird die Änderungen in eine virtuelle Festplatte zu kompilieren und kopieren Sie die Datei auf den angeforderten Pfad.  Diese Methode ist einfach zu verwenden, funktioniert für alle Szenarien und Remote verwendbare ist.  Allerdings wird die virtuelle Festplatte, die häufig generiert eine große Menge an Daten für die Übertragung über das Netzwerk erstellt.
### <a name="win32-apis"></a>Win32-APIs
Entwickler können die SetVirtualDiskInformation GetVirtualDiskInformation und QueryChangesVirtualDisk-APIs auf der virtuellen Festplatte Win32-API wie dokumentiert hier festgelegten: https://docs.microsoft.com/en-us/windows/desktop/api/_vhd/ Beachten Sie, dass zur Verwendung dieser APIs Hyper-V-WMI noch werden, um den Verweis erstellen verwendet muss verweist auf zugehörigen virtuellen Computern.  Diese Win32-APIs ermöglichen dann Zugriff auf die Daten des gesicherten virtuellen Computers.  Die Win32 APIs haben einige Einschränkungen:
*   Sie können nur lokal zugegriffen werden
*   Die nicht lesen von Daten aus Unterstützung freigegebenen virtuellen Festplattendateien
*   Diese zurückgeben Datenadressen, die relativ zum die interne Struktur der virtuellen Festplatte

### <a name="remote-shared-virtual-disk-protocol"></a>Freigegebene Remote-Protokoll für virtuelle Datenträger
Wenn Entwickler effizient backup-Daten aus einer Datei freigegebenen virtuellen Festplatte – den Zugriff auf Informationen müssen sie abschließend das Remote freigegebenen virtuellen Festplatte Protokoll verwenden.  Dieses Protokoll wird hier beschrieben:https://msdn.microsoft.com/en-us/library/dn393384.aspx
