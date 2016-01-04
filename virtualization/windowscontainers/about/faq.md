# Häufig gestellte Fragen

#### Was ist ein Windows Server-Container?

Windows Server-Container stellen eine schlanke Methode zur Betriebssystemvirtualisierung dar, die zum Trennen von Anwendungen und Diensten von anderen Diensten verwendet wird, die auf demselben Containerhost ausgeführt werden. Um dies zu ermöglichen, verfügt jeder Container über eine eigene Sicht über Betriebssystem, Prozesse, Dateisystem, Registrierung und IP-Adressen.

#### Was ist ein Hyper-V-Container?

Sie können sich einen Hyper-V-Container als Windows Server-Container vorstellen, der in einer Hyper-V-Partition ausgeführt wird.

Hyper-V-Container bieten eine zusätzliche Bereitstellungsoption zwischen den überaus effizienten Windows Server-Containern mit hoher Dichte und überaus isolierten virtuellen Hyper-V-Computern mit Hardwarevirtualisierung. In Umgebungen, in denen sich Anwendungen mit verschiedenen Vertrauensstellungsgrenzen auf demselben Host befinden, ist ggf. eine weitere Isolation erforderlich. Hyper-V-Containern bieten einen höheren Isolationsgrad mittels einer optimierten Virtualisierung und des Windows Server-Betriebssystems, das Container voneinander und vom Hostbetriebssystem trennt. Beide Bereitstellungsoptionen für Container nutzen dieselben Verwaltungs-APIs, Tools und Imageformate. Zum Zeitpunkt der Bereitstellung können Kunden einfach den Bereitstellungsmodus wählen, der ihre Anforderungen am besten erfüllt.

#### Was ist der Unterschied zwischen Linux- und Windows Server-Containern?

Linux- und Windows Server-Container ähneln sich, denn beide implementieren ähnliche Technologien in ihrem Kernel und Kernbetriebssystem. Den Unterschied machen die Plattform und Workloads aus, die in den Containern ausgeführt werden.  
Wenn ein Kunde Windows Server-Container verwendet, ist eine Integration in vorhandene Windows-Technologien wie u. a. .NET, ASP.NET und PowerShell möglich.

#### Werden Hyper-V-Container auch für das Docker-Ökosystem verfügbar sein?

Ja. Hyper-V-Container bietet dieselben Integrations- und Verwaltungsmöglichkeiten mit Docker wie Windows Server-Container. Das Ziel ist eine offene, einheitliche und plattformübergreifende Erfahrung.  
Die Docker-Plattform wird außerdem das übergreifende Arbeiten mit unseren Containeroptionen stark vereinfachen und verbessern. Eine Anwendung, die mithilfe von Windows Server-Containern entwickelt wurde, kann ohne Änderung als Hyper-V-Container bereitgestellt werden.

#### Warum muss ich zur Verwaltung von Windows Server-Containern zwischen Docker und PowerShell wählen?

**Dies ist weder das gewünschte Verhalten noch unser langfristiger Plan.**  PowerShell- und Docker-Containerverwaltungstools können künftig parallel verwendet werden.

Allerdings kann das Verwenden mehrerer Verwaltungsschnittstellen zum Verwalten desselben Containers schwierig sein.

Angenommen, Sie erstellen einen Container mit PowerShell und benennen das Image mit einem Großbuchstaben. Docker unterstützt im Gegensatz zu PowerShell keine Großbuchstaben.  
Während dieses Beispiel keine größeren Probleme bereitet, ist der Umgang mit Zustandsänderungen (Racebedingungen und unterschiedlicher Erwartungen), Unterschieden bei Featuregruppen oder Versionen usw. viel schwieriger.

Unsere kurzfristige Entscheidung war, dass Verwaltungsschnittstellen (in diesem Fall Docker und PowerShell) nur die Container sehen, die mit ihnen erstellt wurden. Ein in Docker erstellter Container wird also in PowerShell nicht angezeigt und umgekehrt.

#### Muss ich als Entwickler meine App für jeden Containertyp umschreiben?

Nein. Windows-Containerimages sind für Windows Server-Container und Hyper-V-Container gleich. Die Wahl des Containertyps erfolgt zu Beginn der Containererstellung. Aus Entwicklersicht sind Windows Server- und Hyper-V-Container zwei Varianten desselben Objekts. Sie bieten dieselbe Entwicklungs-, Programmierungs- und Verwaltungserfahrung, sind offen und erweiterbar und weisen über Docker das gleichen Maß an Integration und Unterstützung auf.

Ein Entwickler kann ein Containerimage mithilfe eines Windows Server-Containers erstellen und es als Hyper-V-Container bereitstellen und umgekehrt, wobei die einzige Änderung das Angeben des entsprechenden Laufzeitkennzeichens ist.

Windows Server-Container bieten mehr Dichte und Leistung (z. B. kürzere Anlaufzeit, schnellere Laufzeitleistung im Vergleich zu geschachtelten Konfigurationen), wenn Tempo gefragt ist. Hyper-V-Container bieten einen höheren Isolationsgrad, was sicherstellt, dass Code, der in einem Container ausgeführt wird, das Hostbetriebssystem oder andere Container, die auf demselben Host ausgeführt werden, nicht gefährden oder beeinträchtigen kann. Dies ist hilfreich für mehrinstanzenfähige Szenarien (mit Anforderungen für das Hosten von nicht vertrauenswürdigem Code), einschließlich SaaS-Anwendungen und Computehosting.

#### Sind Hyper-V-/Windows Server-Container ein Add-On, oder werden sie in Windows Server integriert?

Die Containerfunktionen werden in Windows Server 2016 integriert. Je näher die allgemeine Verfügbarkeit rückt, desto mehr Informationen werden veröffentlicht. Bleiben Sie also dran.

#### Was sind die Voraussetzungen für Windows Server- und Hyper-V-Container?

Windows Server- und Hyper-V-Container erfordern Windows Server 2016. Diese Technologien funktionieren nicht mit früheren Versionen von Windows.

#### Kann ich Windows-Container auf ESXi oder einem anderen Nicht-Hyper-V-Hypervisor ausführen?

Ja, Windows-Container können auf sämtlichen TP3 Server Core-Installationen ausgeführt werden. Befolgen Sie die Anweisungen zum [direkten Aktivieren des Features „Container“](../quick_start/inplace_setup.md).

#### Nimmt Microsoft an der Open Container Initiative (OCI) teil?

Um zu gewährleisten, dass das Paketerstellungsformat universell bleibt, hat Docker vor Kurzem die Open Container Initiative (OCI) ins Leben gerufen, um sicherzustellen, dass die Paketerstellung für Container ein offenes von den Gründern geleitetes Format bleibt, wobei Microsoft eines der Gründungsmitglieder ist.

#### Sind Microsoft und Docker wirklich eine Partnerschaft eingegangen?

Ja.  
Unsere Partnerschaft mit Docker ermöglicht Entwicklern das Erstellen, Verwalten und Bereitstellen von Windows Server- und Linux-Containern, die dasselbe Docker-Toolset verwenden. Entwickler für Windows Server müssen nicht mehr eine Wahl treffen zwischen dem Verwenden der umfangreichen Palette von Windows Server-Technologien und dem Erstellen von containerisierten Anwendungen.

Docker ist zwei Dinge: die Open-Source-Gruppe von Projekten und der Name des Unternehmens. Diese Partnerschaft bezieht sich auf beides. Docker ist zum Teil aufgrund seines dynamisches Ökosystems so erfolgreich, das sich um die Docker-Containertechnologie entwickelt hat. Microsoft wirkt am Docker-Projekt mit, was Unterstützung für Windows Server- und Hyper-V-Container ermöglicht.

Weitere Informationen finden Sie im Blogbeitrag [Neue Windows Server-Container und Azure-Unterstützung für Docker](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/?WT.mc_id=Blog_ServerCloud_Announce_TTD).



