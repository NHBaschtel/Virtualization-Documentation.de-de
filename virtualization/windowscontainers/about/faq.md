



# Häufig gestellte Fragen

## Informationen zu Windows-Containern

**Was ist ein Windows Server-Container?**

Windows Server-Container sind eine einfache Betriebssystem-Virtualisierung-Methode verwendet, um Programme oder Dienste von anderen Diensten, die auf dem gleichen Container Host getrennt. Um dies zu ermöglichen, hat jeder Container ihre eigene Ansicht von dem Betriebssystem, Prozesse, Dateisystem, Registrierung und IP-Adressen.

**Was ist ein Hyper-V-Container?**

Sie können einen Hyper-V-Container als ein Windows Server-Container in einer Hyper-V-Partition vorstellen.

Hyper-V-Container bieten eine zusätzliche Bereitstellungsoption zwischen hochgradig effizienten, HD-Windows Server-Container und der hoch isolierten virtualisierten Hardware Hyper-V virtuelle Computer. Umgebungen, in denen andere Clientanwendungen auf demselben Host Vertrauensgrenzen, möglicherweise zusätzliche Isolierung erforderlich. Hyper-V-Container bieten höheren Isolationsstufe, die mithilfe einer optimierten Virtualisierung und Windows Server-Betriebssystem, das Container voneinander und vom Host-Betriebssystem trennt. Beide Container-Bereitstellungsoptionen verwenden dieselben Management-APIs, Tools und Image-Formate zum Zeitpunkt der Bereitstellung, Kunden können einfach die beste Bereitstellungsmodus ihren Anforderungen erfüllt.

**Was ist der Unterschied zwischen Linux- und Windows Server-Containern?**

Linux und Windows Server-Containern ähneln – beide ähnliche Technologien in ihrem Betriebssystem-Kernel und Core implementieren. Der Unterschied ergibt sich aus der Plattform und Arbeitslasten, die innerhalb des Containers ausgeführt.  
Wenn ein Kunde Windows Server-Container verwendet, können sie vorhandene Windows-Technologien wie .NET, ASP.NET, PowerShell und vieles mehr integrieren.

**Als Entwickler habe ich meine app für jeden Container neu schreiben?**

Nein, gelten Container Windows-Abbilder über Windows Server-Container und Hyper-V-Container. Die Auswahl des Containertyps erfolgt beim Starten des Containers. Aus Sicht der Entwickler sind Windows Server-Containern und Hyper-V zwei Arten von dasselbe. Diese Entwicklung, Programmierung und Verwaltung genauso bieten, werden offene und erweiterbare und das gleiche Maß an Integration und Support über Docker enthält.

Ein Entwickler kann ein Container-Abbild, das mit einem Windows Server-Container erstellen und als Hyper-V-Container oder umgekehrt unverändert als das Festlegen der entsprechenden Common Language Runtime-Flag bereitstellen.

Windows Server-Containern bietet höhere Dichte und Leistung (z. B. niedrigere dreht sich Zeit, schnellere Common Language Runtime-Leistung im Vergleich zu geschachtelten Konfigurationen) Wenn Geschwindigkeit Schlüssel ist. Hyper-V-Container bieten mehr Isolation sicherzustellen, dass in einem Container ausgeführte Code kann nicht gefährden oder Auswirkungen auf das Hostbetriebssystem oder andere Container, die auf dem gleichen Host ausgeführt wird. Dies ist hilfreich für mehrinstanzenfähige Szenarien (mit Anforderungen zum Hosten von nicht vertrauenswürdigen Codes) einschließlich SaaS-Anwendung und Compute-hosting.

**Sind Container für Windows/Hyper-V-Server ein Add-on, oder werden sie in Windows Server integriert?**

Die Container-Funktionen werden in Windows Server 2016 integriert. Bleiben Sie dran, Weitere Informationen näher auf die allgemeine Verfügbarkeit.

**Welche Beziehung besteht zwischen Windows Server-Containern und Drawbridge?**

Drawbridge war eines der vielen Forschungsprojekte, die uns wertvolle Einblicke in Container ermöglicht haben. Ein Großteil der Containertechnologie in Windows Server 2016 basiert auf den Erfahrungen mit Drawbridge, und wir freuen uns, unseren Kunden in Windows Server 2016 erstklassige Containertechnologien bereitstellen zu können.

**Was sind die erforderlichen Komponenten für Windows Server-Containern und Hyper-V?**

Sowohl im Fenster Server-Containern und Hyper-V erfordern Windows Server 2016. Diese Technologien funktioniert nicht mit früheren Versionen von Windows.


## Windows-Containerverwaltung

**Sind Hyper-V-Container auch auf das Ökosystem Docker verfügbar?**

Ja – bietet Hyper-V-Container das gleiche Maß an Integration und Verwaltung mit Docker als Container für Windows Server. Das Ziel ist eine offene, konsistente, plattformübergreifende Erfahrung haben.  
Die Plattform Docker erheblich vereinfachen und verbessern die Erfahrung der Arbeit über unser Container-Optionen. Eine Anwendung entwickelt wurde, mithilfe von Windows Server-Container kann als Hyper-V-Container ohne Änderung bereitgestellt werden.

**Warum muss ich zwischen Docker und Windows Server PowerShell-Container Management auswählen?**

_Dies ist weder das gewünschte Verhalten noch unser langfristiger Plan._ PowerShell- und Docker-Containerverwaltungstools können künftig parallel verwendet werden.

Allerdings kann das Verwenden mehrerer Verwaltungsschnittstellen zum Verwalten desselben Containers schwierig sein.

Nehmen Sie z. B. das Erstellen eines Containers mit PowerShell, und benennen das Abbild mit einem Großbuchstaben. Docker unterstützt keine Caps, PowerShell tut.  
Obwohl dieses Beispiel sehr leicht zu verwaltendes ist, welche ruft Behandlung viel schwieriger werden Zustandsänderungen (Racebedingungen und andere Erwartungen), Featureliste Unterschiede oder Versionen...

Unsere Entscheidung kurzfristig wurden (in diesem Fall Docker und PowerShell) Verwaltungsschnittstellen, nur den von die Ihnen erstellten – Siehe-Container Sie einen Container mit Docker erstellen und PowerShell nicht wird angezeigt, mit PowerShell erstellen und Docker nicht wird angezeigt.

**Kann ich Windows-Container auf ESXi oder einem anderen Nicht-Hyper-V-Hypervisor ausführen?**

Ja, Windows-Container können auf sämtlichen TP3 Server Core-Installationen ausgeführt werden. Befolgen Sie die Anweisungen zum [direkten Aktivieren des Features „Container“](../quick_start/inplace_setup.md).


## Das offene Ökosystem von Microsoft

**Teilnehmen Microsoft, die in der geöffneten Container Initiative (OCI)?**

Um sicherzustellen, dass das verpackungsformat universal bleibt organisiert Docker vor kurzem die öffnen Container Initiative (OCI), um sicherzustellen, dass der Container Verpackung eine offene und Foundation-led-Format mit Microsoft als einer der Gründer Member bleibt abzielen.

**Arbeitet Microsoft mit Docker wirklich zusammen?**

Ja.  
Unsere Partnerschaft mit Docker ermöglicht Entwicklern das Erstellen, verwalten und Bereitstellen von Windows Server- und Linux-Container mit denselben Docker Tool. Entwickler für Windows Server müssen nicht mehr zwischen mithilfe der großen Bereich von Windows Server-Technologien und Erstellen von Sammelartikeleinheit auszuwählen.

Docker ist zwei Dinge, die open-Source-Gruppe von Projekten und Docker des Unternehmens. Wir betrachten diese Partnerschaft zu beiden enthalten. Docker ist teilweise erfolgreich ist, aufgrund der dynamisches Ökosystem, das rund um die Docker Container-Technologie aufgebaut wurde. Microsoft ist zum Aktivieren der Unterstützung für Windows Server-Containern und Hyper-V Docker Projekt beitragen.

Weitere Informationen finden Sie im Blogbeitrag [Neue Windows Server-Container und Azure-Unterstützung für Docker](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/?WT.mc_id=Blog_ServerCloud_Announce_TTD).




<!--HONumber=Feb16_HO4-->
