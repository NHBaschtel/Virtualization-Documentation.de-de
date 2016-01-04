# Einführung in Hyper-V unter Windows 10

Als Entwickler von Software, IT-Experte oder Technologie-Enthusiast Sie müssen meist mehrere Betriebssysteme ausführen, gelegentlich sogar auf vielen verschiedenen Computern. Nicht alle von uns haben Zugriff auf diverse Testumgebungen zum Unterbringen aller dieser Computer, weshalb sich mithilfe der Virtualisierung Platz und Zeit sparen lässt.

## Zwecke der Virtualisierung

Dank Virtualisierung lassen sich mehrere Testumgebungen betreiben, die aus zahlreichen Betriebssystemen, Software- und Hardwarekonfigurationen bestehen. Hyper-V bietet Virtualisierung unter Windows sowie einen einfachen Mechanismus zum schnellen Umschalten zwischen diesen Umgebungen, ohne dass zusätzliche Hardwarekosten anfallen.

Hyper-V kann auf vielerlei Weise verwendet werden:
- Eine aus mehreren virtuellen Computern bestehende Testumgebung kann auf einem einzelnen Desktop- oder Laptopcomputer erstellt werden. Nach Abschluss der Tests können diese virtuellen Computer exportiert und anschließend in beliebige andere Hyper-V-Systeme importiert werden.

- Entwickler können Hyper-V auf ihrem Computer zum Testen von Software unter mehreren Betriebssystemen verwenden. Wenn Sie z. B. eine Anwendung haben, die unter Windows 8, Windows 7 und Linux getestet werden muss, können Sie auf Ihrem Entwicklungssystem mehrere virtuelle Computer erstellen, die je eines dieser Betriebssysteme enthalten.

- Mit Hyper-V können Sie in beliebigen Hyper-V-Bereitstellungen Probleme mit virtuellen Computern beheben. Sie können einen virtuellen Computer aus Ihrer Produktionsumgebung exportieren, ihn auf Ihrem Desktop-PC, auf dem Hyper-V ausgeführt wird, öffnen, die erforderliche Problembehandlung vornehmen und ihn dann zurück in die Produktionsumgebung exportieren.

- Mithilfe eines virtuellen Netzwerks können Sie für Test-, Entwicklungs- und Demonstrationszwecke eine Umgebung mit mehreren Computern einrichten, die das Produktionsnetzwerk nicht beeinträchtigt.

- Enthusiasten können Hyper-V nutzen, um mit anderen Betriebssystemen zu experimentieren. Mit Hyper-V ist es sehr einfach, verschiedene Betriebssysteme hoch- und herunterzufahren.

- Sie können Hyper-V auf einem Laptop zur Demonstration älterer Versionen von Windows oder von Nicht-Windows-Betriebssystemen verwenden.


## Systemanforderungen

Hyper-V erfordert ein 64-Bit-System mit SLAT (Second Level Address Translation). SLAT ist ein Merkmal der aktuelle Generation von 64-Bit-Prozessoren von Intel und AMD. Außerdem benötigen Sie eine 64-Bit-Version von Windows 8 oder höher und mindestens 4 GB RAM. Hyper-V unterstützt die Erstellung von 32-Bit- und 64-Bit-Betriebssystemen in den virtuellen Computern.

Der dynamische Speicher von Hyper-V ermöglicht die dynamische Zuordnung von Arbeitsspeicher zur VM bzw. Aufhebung der Zuordnung (Sie geben einen Mindest- und Höchstwert an) und die gemeinsame Nutzung von nicht belegtem Speicher zwischen VMs. Sie können 3 oder 4 VMs auf einem Computer mit 4 GB RAM ausführen. Bei mehr als 5 VMs benötigen Sie jedoch mehr RAM. Am anderen Ende des Spektrums können Sie auch je nach physischer Hardware große VMs mit 32 Prozessoren und 512 GB RAM erstellen.

## Auf einem virtuellen Computer ausführbare Betriebssysteme

Der Begriff „Gast“ bezieht sich auf einen virtuellen Computer und „Host“ auf den Computer, auf dem der virtuelle Computer ausgeführt wird. Hyper-V unter Windows unterstützt viele verschiedene Gastbetriebssysteme einschließlich verschiedener Versionen von Linux, FreeBSD und Windows. Informationen dazu, welche Betriebssysteme in Hyper-V unter Windows als Gäste unterstützt werden, finden Sie unter [Unterstützte Windows-Gastbetriebssysteme](supported_guest_os.md) und [Virtuelle Computer mit Linux und FreeBSD in Hyper-V](https://technet.microsoft.com/library/dn531030.aspx).


## Unterschiede zwischen Hyper-V unter Windows und Hyper-V unter Windows Server

Es gibt einige Features, die bei Hyper-V unter Windows und Hyper-V unter Windows Server unterschiedlich sind. Beispiele:

- Das Speicherverwaltungsmodell unterscheidet sich bei Hyper-V unter Windows. Auf einem Server wird Hyper-V-Arbeitsspeicher unter der Annahme verwaltet, dass nur die virtuellen Computer auf dem Server ausgeführt werden. Bei Hyper-V unter Windows wird Arbeitsspeicher unter der Annahme verwaltet, dass auf den meisten Clientcomputern zusätzlich zu virtuellen Computern Software ausgeführt wird. Beispielsweise kann ein Entwickler sowohl Visual Studio als auch mehrere virtuelle Computer auf dem gleichen Computer ausführen.

- SR-IOV auf einem 64-Bit-Gast funktioniert normal, 32-Bit hingegen nicht und wird nicht unterstützt.


### In Windows Hyper-V nicht verfügbare Windows Server-Features

Es gibt einige Features in Hyper-V unter Windows Server, die in Hyper-V unter Windows nicht enthalten sind. Beispiele:

- Die Remote FX-Funktion zur Virtualisierung von GPUs

- Die Livemigration von einem Host auf einen anderen

- Hyper-V-Replikat

- Virtueller Fibre Channel

- SR-IOV-Netzwerk

- Freigegebene VHDX-Datei


> **Warnung**: In Hyper-V ausgeführte virtuelle Computer verarbeiten den Wechsel von einer drahtgebundenen zu einer drahtlosen Verbindung nicht automatisch. Sie müssen die Einstellungen des Netzwerkadapters für virtuelle Computer manuell ändern.

## Einschränkungen

Die Virtualisierung hat Grenzen. Features oder Anwendungen, die von bestimmter Hardware abhängen, funktioniert auf einem virtuellen Computer nicht besonders. Spiele oder Anwendungen, die eine Verarbeitung mit GPUs erfordern (ohne Fallback auf Software anzubieten), funktionieren möglicherweise nicht gut. Auch bei Anwendungen, die mit Zeitgebern unter 10 ms arbeiten, wie z. B. latenzempfindliche Hochpräzisions-Apps für die Livemischung von Musik, können in einer VM Probleme auftreten. Das Stammbetriebssystem wird auch auf der Hyper-V-Virtualisierungsebene ausgeführt, ist aber dahingehend besonders, dass es einen direkten Zugriff auf die gesamte Hardware hat. Das ist der Grund, warum Anwendungen mit besonderen Hardwareanforderungen im Stammverzeichnis weiter stabil ausgeführt werden können. Doch bei latenzempfindlichen Hochpräzisions-Apps können bei Ausführung im Stammbetriebssystem weiter Probleme auftreten.

Zur Erinnerung: Sie müssen für alle Betriebssysteme, die Sie in den virtuellen Computern verwenden, über eine gültige Lizenz verfügen.

## Nächster Schritt:

[Exemplarische Vorgehensweise für Hyper-V unter Windows 10](..\quick_start\walkthrough.md)

Machen Sie sich mit den [Neuerungen](whats_new.md) in Hyper-V für Windows Server 10 vertraut.





