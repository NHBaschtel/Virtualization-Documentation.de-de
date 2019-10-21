# <a name="microsoft-software-supplemental-license-for-windows-container-base-image"></a>Microsoft-Software-Ergänzung-Lizenz für Windows-Container Basis Bild

Diese zusätzliche Lizenz ist für das Windows-Container-Basis Bild ("Container-Image"). Wenn Sie die Bedingungen dieser zusätzlichen Lizenz erfüllen, können Sie das Container Bild wie nachstehend beschrieben verwenden.

## <a name="container-os-image"></a>CONTAINERBETRIEBSSYSTEM-IMAGE
Das Container Bild darf nur mit einer gültig lizenzierten Kopie von verwendet werden:

Windows Server-Standard-oder Windows Server Datacenter-Software (zusammengefasst "Server-Host-Software") oder Software für Microsoft Windows-Betriebs System (Version 10) ("Client-Host-Software") oder Windows 10-viel Enterprise-und Windows 10-Core (gemeinsam "viel") Host Software ").
Die Server-Host-Software, die Client-Host-Software und die viel-Host-Software werden gemeinsam als "Host-Software" bezeichnet, und eine Lizenz für die Host-Software ist eine "Host-Lizenz".

Sie dürfen das Container-Image nicht verwenden, wenn Sie nicht über eine entsprechende Version und Edition der Host-Lizenz verfügen. Es gelten bestimmte Einschränkungen und zusätzliche Bestimmungen, die hierin beschrieben sind. Wenn die Lizenzbedingungen in Konflikt mit der Host-Lizenz stehen, wird diese zusätzliche Lizenz in Bezug auf das Container-Image geregelt. indem Sie diese zusätzliche Lizenz akzeptieren oder das Container-Bild verwenden, erklären Sie sich mit den folgenden Bedingungen einverstanden. Wenn Sie diese Bestimmungen nicht akzeptieren und befolgen, dürfen Sie das Container Bild nicht verwenden.

## <a name="definitions"></a>Definitionen
Der **Windows Server-Container** (ohne Hyper-V-Isolierung) ist eine Funktion der Microsoft Windows Server-Software.

**Windows Server-Container mit Hyper-V-Isolierung** Abschnitt 2 (k) der Microsoft Windows Server-Lizenzbestimmungen wird hiermit vollständig gelöscht und durch die überarbeiteten Begriffe ersetzt, wie unter "aktualisiert" unten dargestellt.

Aktualisiert: der Windows Server-Container mit Hyper-v-Isolierung (vormals als Hyper-v-Container bezeichnet) ist eine Container Technologie in Windows Server, die eine virtuelle Betriebssystemumgebung verwendet, um einen oder mehrere Windows Server-Container (n) zu hosten. Jede Hyper-V-Isolierungs Instanz, die zum Hosten von einem oder mehreren Windows Server-Containern verwendet wird, gilt als eine virtuelle Betriebssystemumgebung.

## <a name="license-terms"></a>Lizenzbestimmungen
**Host-Lizenz.** Die Host-Lizenzbedingungen gelten für ihre Verwendung des Container-Images und aller Windows-Container, die mit dem Container-Bild erstellt wurden, die eindeutig und von einem virtuellen Computer getrennt sind.

**Nutzungsrechte.** Das Container Bild kann verwendet werden, um eine isolierte virtualisierte Windows-Betriebssystemumgebung zu erstellen, die mindestens eine Anwendung enthält, die primäre und signifikante Funktionen hinzufügt. Sie können das Container Bild nur verwenden, um Windows-Container (s) auf der Host Software zu erstellen, zu erstellen und auszuführen. Updates der Host Software aktualisieren das Container Bild möglicherweise nicht, sodass Sie möglicherweise alle Windows-Container auf Grundlage eines aktualisierten Container Bilds neu erstellen können.

**Einschränkungen.** Sie können diese zusätzliche Lizenzdokument Datei nicht aus dem Container Bild entfernen. Sie können den Remotezugriff auf die Anwendung (en), die Sie in Ihrem Container ausführen, nicht aktivieren, um entsprechende Lizenzgebühren zu vermeiden. Sie sind nicht berechtigt, das Container Bild zu Reverse Engineering, zu dekompilieren oder zu disassemblieren oder zu versuchen, es sei denn, dies ist nur in dem Umfang erforderlich, der durch die Lizenzbedingungen von Drittanbietern für die Verwendung bestimmter Open-Source-Komponenten, die in der Software enthalten sein können, erforderlich ist. Es gelten möglicherweise zusätzliche Einschränkungen in der Host-Lizenz.

## <a name="additional-terms"></a>zusätzliche Begriffe
**Client-Host Software.** Wenn Sie ein Container Abbild auf der Client-Host Software ausführen, können Sie eine beliebige Anzahl des Container Bilds ausführen, das als Windows-Container für Test-oder Entwicklungszwecke instanziiert wurde. Sie sind nicht berechtigt, diese Windows-Container in einer Produktionsumgebung auf Client-Host-Software zu verwenden.

**Viele Host-Software.** Wenn Sie ein Container-Image auf vielen Host-Software ausführen, können Sie eine beliebige Anzahl von Container-Bildern ausführen, die nur zu Test-oder Entwicklungszwecken als Windows-Container instanziiert wurden. Sie dürfen das Container Bild nur in einer Produktionsumgebung verwenden, wenn Sie den Microsoft Commercial-Nutzungsbedingungen für Windows 10 Core-Lauf Zeit Bilder oder die Windows 10-Enterprise-Gerätelizenz ("Windows-Vertrag für kommerzielle Nutzung") zugestimmt haben. Zusätzliche Bestimmungen und Einschränkungen in den Windows-kommerziellen Vereinbarungen gelten für ihre Verwendung des Container Bilds in einer Produktionsumgebung.

**Software von Drittanbietern.** Das Container Bild kann Anwendungen von Drittanbietern enthalten, die Ihnen unter dieser zusätzlichen Lizenz oder unter ihren eigenen Bedingungen lizenziert sind. Lizenzbestimmungen, Hinweise und Bestätigungen, falls vorhanden, für die Anwendungen von Drittanbietern sind möglicherweise Online in http://aka.ms/thirdpartynotices oder in einer begleitenden Benachrichtigungsdatei abrufbar. Auch wenn derartige Anwendungen anderen Vereinbarungen unterliegen, gelten der Haftungsausschluss, die Beschränkungen und Ausschlüsse von Schäden in der Host-Lizenz auch in dem durch das anwendbare Recht gestatteten Umfang.

**Öffnen Sie Quellkomponenten.** Das Container-Image kann urheberrechtlich geschützte Software Dritter enthalten, die unter Open-Source-Lizenzen mit Quellcode-Verfügbarkeits Verpflichtungen lizenziert ist. Kopien dieser Lizenzen sind in der ThirdPartyNotices-Datei oder in einer anderen begleitenden Benachrichtigungsdatei enthalten. Sie können den vollständigen entsprechenden Quellcode von Microsoft anfordern, wenn und wie dies unter der entsprechenden Open-Source-Lizenz erforderlich ist, indem Sie eine Zahlungsanweisung senden oder auf $5,00 zu: Quellcode Compliance-Team, Microsoft Corporation, 1 Microsoft Way, Redmond, WA 98052, USA. Geben Sie den Namen "Microsoft-Software-Ergänzung-Lizenz für Windows-Container Basis", den Namen der Open Source-Komponente und die Versionsnummer in der Zeile "Memo" Ihrer Zahlung an. Sie können auch eine Kopie der Quelle unter http://aka.ms/getsourcefinden.
