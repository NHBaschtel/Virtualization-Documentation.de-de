# <a name="hyper-v-backup-approaches"></a>Hyper-V-Sicherungs Ansätze
Hyper-V ermöglicht es Ihnen, virtuelle Computer vom Hostbetriebssystem zu sichern, ohne benutzerdefinierte Sicherungssoftware innerhalb des virtuellen Computers ausführen zu müssen.  Es gibt verschiedene Ansätze, die Entwicklern je nach Ihren Anforderungen zur Verfügung stehen.
## <a name="hyper-v-vss-writer"></a>Hyper-V-VSS-Writer
Hyper-v implementiert einen VSS-Writer auf allen Versionen von Windows Server, auf denen Hyper-v unterstützt wird.  Dieser VSS-Writer ermöglicht es Entwicklern, die vorhandene VSS-Infrastruktur für die Sicherung virtueller Computer zu verwenden.  Sie ist jedoch für kleine Backup-Vorgänge vorgesehen, bei denen alle virtuellen Computer auf einem Server gleichzeitig gesichert werden.

Um diese Architektur besser verstehen zu können, lesen Sie diese Präsentation:https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-V-WMI-basiertes Backup
Ab Windows Server 2016 unterstützt Hyper-v die Sicherung über die Hyper-v-WMI-API.  Dieser Ansatz nutzt weiterhin VSS innerhalb des virtuellen Computers zu Sicherungszwecken, verwendet aber nicht mehr VSS im Hostbetriebssystem.  Stattdessen wird eine Kombination aus Referenzpunkten und einer widerstandsfähigen Änderungsnachverfolgung (RCT) verwendet, um Entwicklern den Zugriff auf die Informationen zu gesicherten virtuellen Maschinen auf effiziente Weise zu ermöglichen.  Dieser Ansatz ist skalierbarer als die Verwendung von VSS im Host, ist jedoch nur unter Windows Server 2016 und höher verfügbar.

Um diese Architektur besser verstehen zu können, lesen Sie diese Präsentation:https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

Ein Beispiel für die Verwendung dieser APIs finden Sie hier:https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Methoden zum Lesen von Sicherungen aus WMI-basierter Sicherung
Beim Erstellen von Sicherungen virtueller Computer mit Hyper-V WMI gibt es drei Methoden zum Lesen der tatsächlichen Daten aus der Sicherung.  Jede hat einzigartige vor-und Nachteile.
### <a name="wmi-export"></a>WMI-Export
Entwickler können die Sicherungsdaten über die Hyper-V-WMI-Schnittstellen exportieren (wie im obigen Beispiel verwendet).  Hyper-V kompiliert die Änderungen auf einer virtuellen Festplatte und kopiert die Datei an den angeforderten Speicherort.  Diese Methode ist einfach zu verwenden, funktioniert in allen Szenarien und ist remotefähig.  Mit der generierten virtuellen Festplatte wird jedoch häufig eine große Menge an Daten erstellt, die über das Netzwerk übertragen werden.
### <a name="win32-apis"></a>Win32-APIs
Entwickler können die SetVirtualDiskInformation-, GetVirtualDiskInformation-und QueryChangesVirtualDisk-APIs auf dem virtuellen Festplatten-Win32-API- https://docs.microsoft.com/windows/desktop/api/_vhd/ Satz verwenden, wie hier dokumentiert: Beachten Sie, dass für die Verwendung dieser APIs weiterhin Hyper-V WMI zum Erstellen von Verweisen verwendet werden muss. Punkte auf zugeordneten virtuellen Computern.  Diese Win32-APIs ermöglichen dann einen effizienten Zugriff auf die Daten der gesicherten virtuellen Maschine.  Die Win32-APIs haben mehrere Einschränkungen:
* Auf Sie kann nur lokal zugegriffen werden
* Das Lesen von Daten aus freigegebenen virtuellen Festplatten-Dateien wird nicht unterstützt
* Sie geben Daten Adressen zurück, die relativ zur internen Struktur der virtuellen Festplatte sind.

### <a name="remote-shared-virtual-disk-protocol"></a>Remote Shared Virtual Disk Protocol
Und schließlich: Wenn ein Entwickler effizient auf Sicherungsdaten aus einer freigegebenen virtuellen Festplattendatei zugreifen muss, müssen Sie das freigegebene Virtual Disk Protocol verwenden.  Dieses Protokoll ist [hier](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)dokumentiert.
