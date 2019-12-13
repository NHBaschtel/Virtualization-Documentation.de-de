# <a name="hyper-v-backup-approaches"></a>Hyper-V-Sicherungs Ansätze
Mit Hyper-V können Sie virtuelle Computer vom Host Betriebssystem aus sichern, ohne dass innerhalb des virtuellen Computers eine benutzerdefinierte Sicherungssoftware ausgeführt werden muss.  Es gibt verschiedene Ansätze, die Entwickler abhängig von Ihren Anforderungen verwenden können.
## <a name="hyper-v-vss-writer"></a>Hyper-V-VSS-Writer
Hyper-v implementiert eine VSS Writer für alle Versionen von Windows Server, auf denen Hyper-v unterstützt wird.  Diese VSS Writer ermöglicht Entwicklern die Verwendung der vorhandenen VSS-Infrastruktur zum Sichern virtueller Computer.  Es ist jedoch für kleine Sicherungs Vorgänge konzipiert, bei denen alle virtuellen Computer auf einem Server gleichzeitig gesichert werden.

Um diese Architektur besser zu verstehen – lesen Sie diese Präsentation: https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-V-WMI-basierte Sicherung
Ab Windows Server 2016 begann Hyper-v, die Sicherung über die Hyper-v-WMI-API zu unterstützen.  Bei diesem Ansatz wird weiterhin VSS innerhalb des virtuellen Computers für Sicherungszwecke verwendet, aber VSS wird nicht mehr im Host Betriebssystem verwendet.  Stattdessen wird eine Kombination aus Referenzpunkten und robuster Änderungs Nachverfolgung (RCT) verwendet, damit Entwickler auf effiziente Weise auf die Informationen zu gesicherten virtuellen Computern zugreifen können.  Diese Vorgehensweise ist skalierbarer als die Verwendung von VSS auf dem Host, Sie ist jedoch nur unter Windows Server 2016 und höher verfügbar.

Um diese Architektur besser zu verstehen – lesen Sie diese Präsentation: https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

Außerdem ist ein Beispiel für die Verwendung dieser APIs verfügbar: https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>Methoden zum Lesen von Sicherungen aus einer WMI-basierten Sicherung
Beim Erstellen von Sicherungen virtueller Computer mit Hyper-V-WMI gibt es drei Methoden zum Lesen der tatsächlichen Daten aus der Sicherung.  Jede verfügt über eindeutige vor-und Nachteile.
### <a name="wmi-export"></a>WMI-Export
Entwickler können die Sicherungsdaten über die Hyper-V-WMI-Schnittstellen exportieren (wie im obigen Beispiel verwendet).  Die Änderungen werden von Hyper-V in eine virtuelle Festplatte kompiliert und an den angeforderten Speicherort kopiert.  Diese Methode ist einfach zu verwenden, funktioniert für alle Szenarien und ist Remote fähig.  Die generierte virtuelle Festplatte erstellt jedoch häufig eine große Menge an Daten, die über das Netzwerk übertragen werden.
### <a name="win32-apis"></a>Win32-APIs
Entwickler können die setvirtualdiskinformation-, getvirtualdiskinformation-und querychangesvirtualdisk-APIs auf der Win32-API-Gruppe der virtuellen Festplatte wie hier beschrieben verwenden: https://docs.microsoft.com/windows/desktop/api/_vhd/ beachten Sie, dass zum Erstellen dieser APIs auch Hyper-V-WMI verwendet werden muss, um Referenzpunkte auf zugeordneten virtuellen Maschinen zu erstellen.  Diese Win32-APIs ermöglichen einen effizienten Zugriff auf die Daten des gesicherten virtuellen Computers.  Die Win32-APIs haben mehrere Einschränkungen:
* Auf Sie kann nur lokal zugegriffen werden.
* Das Lesen von Daten aus freigegebenen virtuellen Festplatten Dateien wird von nicht unterstützt.
* Sie geben Daten Adressen zurück, die relativ zur internen Struktur der virtuellen Festplatte sind.

### <a name="remote-shared-virtual-disk-protocol"></a>Protokoll für freigegebene Remote Datenträger
Wenn ein Entwickler zum Schluss effizient auf Sicherungsdaten Informationen aus einer freigegebenen virtuellen Festplatten Datei zugreifen muss – muss er das freigegebene Remote Protokoll für virtuelle Festplatten verwenden.  Dieses Protokoll ist [hier](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)dokumentiert.
