# <a name="microsoft-software-supplemental-license-for-windows-container-base-image"></a>Microsoft-Software-Zusatzlizenz für Windows-Container-Basis Image

Diese ergänzende Lizenz gilt für das Windows-Containerbasis Image ("Container Image"). Wenn Sie die Bedingungen dieser ergänzenden Lizenz einhalten, können Sie das Container Image verwenden, wie unten beschrieben.

## <a name="container-os-image"></a>CONTAINERBETRIEBSSYSTEM-IMAGE
Das Container Image darf nur mit einer ordnungsgemäß lizenzierten Kopie von verwendet werden:

Windows Server Standard oder Windows Server Datacenter Software (kollektiv "Server Host Software") oder Microsoft Windows-Betriebs System Software ("Client Host Software") oder Windows 10 IOT Enterprise und Windows 10 IOT Core (kollektiv "IOT" Host Software ").
Die Server Host Software, die Client Host Software und die IOT-Host Software werden zusammen als "Host Software" bezeichnet, und eine Lizenz für die Host Software ist eine "Host Lizenz".

Sie dürfen das Container Image nicht verwenden, wenn Sie nicht über eine entsprechende Version und Edition der Host Lizenz verfügen. Bestimmte Einschränkungen und zusätzliche Bedingungen können angewendet werden, die hier beschrieben werden. Wenn die Lizenzbedingungen in Konflikt mit der Host Lizenz liegen, wird diese ergänzende Lizenz in Bezug auf das Container Image geregelt. Wenn Sie diese ergänzende Lizenz akzeptieren oder das Container Image verwenden, stimmen Sie diesen Bedingungen zu. Wenn Sie diese Bedingungen nicht akzeptieren und einhalten, dürfen Sie das Container Image nicht verwenden.

## <a name="definitions"></a>DEFINITIONEN
Der **Windows Server-Container** (ohne Hyper-V-Isolation) ist ein Feature von Microsoft Windows Server-Software.

**Windows Server-Container mit Hyper-V-Isolation.** Abschnitt 2 (k) der Microsoft Windows Server-Lizenzbedingungen werden in seiner Gesamtheit gelöscht und durch die geänderten Begriffe ersetzt, wie unten im Abschnitt "aktualisiert" gezeigt.

Aktualisiert: Windows Server-Container mit Hyper-v-Isolation (früher als Hyper-v-Container bezeichnet) ist eine Container Technologie in Windows Server, die eine virtuelle Betriebssystemumgebung zum Hosten von mindestens einem Windows Server-Container verwendet. Jede Hyper-V-Isolations Instanz, die zum Hosten von mindestens einem Windows Server-Container verwendet wird, wird als eine virtuelle Betriebssystemumgebung angesehen.

## <a name="license-terms"></a>Lizenzbedingungen
**Host Lizenz.** Die Lizenzbedingungen für den Host gelten für die Verwendung des Container Images und aller Windows-Container, die mit dem Container Image erstellt wurden und von einem virtuellen Computer getrennt und getrennt sind.

**Rechte verwenden.** Das Container Image kann zum Erstellen einer isolierten virtualisierten Windows-Betriebssystemumgebung verwendet werden, in der mindestens eine Anwendung enthalten ist, die primäre und bedeutende Funktionalität hinzufügt. Sie dürfen das Container Image nur zum Erstellen, erstellen und Ausführen von Windows-Containern auf Host Software verwenden. Updates der Host Software aktualisieren das Container Image möglicherweise nicht, sodass Sie Windows-Container auf der Grundlage eines aktualisierten Container Images neu erstellen können.

**Begrenzungen.** Sie dürfen diese ergänzende Lizenz Dokumentdatei nicht aus dem Container Image entfernen. Sie können den Remote Zugriff auf die Anwendung (en), die Sie in Ihrem Container ausführen, nicht aktivieren, um die anwendbaren Lizenzgebühren zu vermeiden. Sie dürfen das Container Image nicht umkehren, dekompilieren oder disassemblieren oder versuchen, dies zu tun, außer und nur in dem Umfang, der für Lizenzbedingungen von Drittanbietern erforderlich ist, die die Verwendung bestimmter Open Source-Komponenten steuern, die möglicherweise in der Software enthalten sind. Zusätzliche Einschränkungen in der Host Lizenz können zutreffen.

## <a name="additional-terms"></a>Weitere Bedingungen
**Client Host Software.** Wenn Sie ein Container Image auf Client Host Software ausführen, können Sie eine beliebige Anzahl von Container Images ausführen, die als Windows-Container für Test-oder Entwicklungszwecke instanziiert werden. Diese Windows-Container dürfen nicht in einer Produktionsumgebung auf Client Host Software verwendet werden.

**IOT-Host Software.** Wenn Sie ein Container Image auf der IOT-Host Software ausführen, können Sie eine beliebige Anzahl von Container Images ausführen, die als Windows-Container für Test-oder Entwicklungszwecke instanziiert werden. Sie dürfen das Container Image nur in einer Produktionsumgebung verwenden, wenn Sie die kommerziellen Nutzungsbedingungen von Microsoft für Windows 10 Core-Lauf Zeit Images oder die Windows 10 IOT Enterprise-Geräte Lizenz ("Windows IOT-Handelsvereinbarung") zugestimmt haben. Zusätzliche Nutzungsbedingungen in den Windows IOT-Handelsvereinbarungen gelten für die Verwendung des Container Images in einer Produktionsumgebung.

**Third Party Software.** Das Container Image kann Anwendungen von Drittanbietern enthalten, die Ihnen gemäß dieser ergänzenden Lizenz oder eigenen Bedingungen lizenziert sind. Lizenzbedingungen, Hinweise und Bestätigungen (sofern vorhanden) für die Anwendungen von Drittanbietern sind möglicherweise online unter http://aka.ms/thirdpartynotices oder in einer begleitenden Benachrichtigungs Datei verfügbar. Auch wenn solche Anwendungen von anderen Vereinbarungen unterstützt werden, gelten der Haftungsausschluss, die Einschränkungen und die Ausschlüsse von Schäden in der Host Lizenz auch im durch das anwendbare Recht zugelassenen Umfang.

**Open-Source-Komponenten.** Das Container Image kann geschützte Software von Drittanbietern enthalten, die unter Open Source-Lizenzen mit Quell Code Verfügbarkeit lizenziert ist. Kopien dieser Lizenzen sind in der thirdpartynotizen-Datei oder in einer anderen begleitenden Benachrichtigungs Datei enthalten. Sie erhalten ggf. den gesamten entsprechenden Quellcode von Microsoft, wenn dies unter der relevanten Open-Source-Lizenz erforderlich ist, indem Sie eine Money-Bestellung senden oder nach $5,00 für das Quell Code Konformitäts Team, Microsoft Corporation, 1 Microsoft Way, Redmond, WA 98052 und USA suchen. Geben Sie den Namen "Microsoft-Software ergänzende Lizenz für Windows-Container-Basis Image", den Open Source-Komponentennamen und die Versionsnummer in der Memo Zeile Ihrer Zahlung an. Möglicherweise finden Sie auch eine Kopie der Quelle unter http://aka.ms/getsource.
