# Aufbauen eines Containerökosystems

Um zu verstehen, warum das Aufbauen eines Containerökosystems so wichtig ist, lassen Sie zunächst über Docker reden.

## Der Charme von Docker

Das Konzept von Containern (Isolation von Namespaces und Ressourcenkontrolle) ist nicht wirklich neu und geht zurück auf BSD Jails, Solaris Zones und den grundlegenden UNIX-Mechanismus „chroot (change root)“. Was Docker beigetragen hat, ist ein allgemeines Toolset, ein Paketerstellungsmodell und ein Bereitstellungsmechanismus. Dadurch vereinfacht Docker die Containerisierung und Verteilung von Clientanwendungen erheblich. Diese Anwendungen können dann überall auf beliebigen Linux-Hosts ausgeführt werden. Diese Funktionalität bieten wir auch unter Windows.

Diese weit verbreitete Technologie vereinfacht nicht nur die Verwaltung durch das Anbieten derselben Verwaltungsbefehle auf beliebigen Hosts, sondern ebnet auch den Weg für reibungslose DevOps.

Sie können ein Docker-Image erstellen, das in Sekunden in beliebigen Umgebungen bereitgestellt werden kann, z. B. auf dem Desktop-PC des Entwicklers, auf einem Testcomputer oder auf Produktionscomputern. Mittlerweile gibt es eine riesiges und weiter wachsendes Ökosystem von Anwendungen, die in Docker-Containern gepackt sind, und zwar in DockerHub, der von Docker verwalteten öffentlichen containerisierten Anwendungsregistrierung.

Docker bietet eine überzeugende Grundlage für die Entwicklung.

Nun wollen wir uns mit diesem Ökosystem und damit beschäftigen, wie Sie auf Docker-Konzepten aufbauen können, um einen Entwicklungs- und Bereitstellungsworkflow entsprechend Ihren Anforderungen einzurichten.


## Komponenten eines Containerökosystems

Windows-Container sind eine wichtige Komponente eines großen Containerökosystems. Wir arbeiten mit der gesamten Branche zusammen, um Entwicklern Optionen auf allen Ebenen des Lösungsstapels zu bieten.

Das Containerökosystem bietet Methoden zum Verwalten von Containern, Freigeben von Containern und Entwickeln von in Containern ausgeführten Apps.

![](media/containerEcosystem.png)


Microsoft möchte bei der Entwicklung dieser Apps der nächsten Generation Entwicklern mehr Optionen und Produktivität bieten. Unser Ziel ist das Steigern der Entwicklerproduktivität, was bedeutet, dass Anwendungen für beliebige Microsoft-Clouds geschrieben werden können, ohne das Code geändert, neu geschrieben oder neu konfiguriert werden muss.

Microsoft setzt sich für ein offenes Ökosystem ein. Wir unterstützen aktiv das Zusammenkommen mehrerer wichtiger Ökosysteme für Entwickler, wie z. B. Windows und Linux, um Innovationen voranzutreiben.

In den nächsten Monaten werden wir weitere Informationen zu zusätzliche Partnern in diesem sich entwickelnden Ökosystem veröffentlichen.




