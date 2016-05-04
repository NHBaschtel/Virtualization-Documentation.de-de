



# Windows-Container

Anwendungen sind der Motor der Innovation im Zeitalter von Cloud- und Mobiltechnologien. Container und das Ökosystem, das gezogen, entwickelt werden Softwareentwickler Applications-Erlebnis die nächste Generation erstellen können.

Video mit Kurzübersicht: [Windows-based containers: Modern app development with enterprise-grade control](https://youtu.be/Ryx3o0rD5lY).

## Was sind Container?

Bei Containern handelt es sich um eine isolierte, von Ressourcen gesteuerte und portierbare Betriebsumgebung.

Im Grunde ist ein Container einer isolierten Stelle, wo eine Anwendung ohne den Rest des Systems und ohne das System, das sich auf die Anwendung ausgeführt werden kann. Container sind der nächste Entwicklungsschritt bei der Virtualisierung.

Wenn Sie sich in einem Container befänden, sähe es dort so aus, als befänden Sie sich in einem frisch installierten physischen Container bzw. virtuellen Computer. Und mit [Docker](https://www.docker.com/) kann ein Windows Server-Container auf die gleiche Weise wie jeder andere Container verwaltet werden.

## Windows-Container

Windows-Container enthalten zwei verschiedene Containertypen bzw. Laufzeiten.

**Windows Server-Container**: Bieten Anwendungsisolation mittels einer Isolationstechnologie für Prozesse und Namespaces. Ein Windows Server-Container teilt sich einen Kernel mit dem Containerhost und allen Container, die auf dem Host ausgeführt werden.

**Hyper-V-Container**: Erweitert die von Windows Server-Containern gebotene Isolation, indem jeder Container in einem stark optimierten virtuellen Computer ausgeführt wird. Bei dieser Konfiguration wird der Kernel des Containerhosts nicht gemeinsam mit den Hyper-V-Containern verwendet.


## Grundlegendes zu Containern

Beim Arbeiten mit Containern bemerken Sie viele ähnlichkeiten zwischen einem Container und einer virtuellen Maschine. Ein Container ein Betriebssystem ausgeführt wird, verfügt über ein Dateisystem und kann über ein Netzwerk zugegriffen werden, als ob es sich um eine physische oder virtuelle Computersystem war. Dies bedeutet, dass die Technologie und die Konzepte hinter Container sehr von dem der virtuelle Computer unterscheiden.

[In diesem Blogbeitrag](http://azure.microsoft.com/blog/2015/08/17/containers-docker-windows-and-trends/) von Mark Russinovich werden Container aufschlussreich erläutert.

Die folgenden grundlegenden Konzepte können nützlich sein, wenn Sie beginnen, Windows-Container zu erstellen und damit zu arbeiten.

**Containerhost:** Physisches oder virtuelles Computersystem, das mit dem Feature „Windows-Container“ konfiguriert wurde. Auf dem Containerhost werden ein oder mehrere Windows-Container ausgeführt.

**Containerimage:** Wenn am Dateisystem oder der Registrierung eines Containers Änderungen erfolgen, z. B. durch eine Softwareinstallation, werden diese in einem Sandkasten erfasst. In vielen Fällen ist es wünschenswert, diesen Zustand zu erfassen, damit neue Container erstellt werden können, die diese Änderungen übernehmen. Was ist ein Image ist – nach der Container, die Sie beendet wurde, kann entweder diese Sandbox verwerfen oder können Sie es in ein neues Container-Bild konvertieren. Nehmen wir z. B., dass Sie einen Container aus dem Windows Server Core-Betriebssystem-Image bereitgestellt haben. Anschließend installieren Sie MySQL in diesem Container. Erstellen ein neues Bild aus diesem Container würde als bereitstellbar Version des Containers fungieren. Dieses Bild enthält nur die Änderungen (MySQL), jedoch als eine Schicht über das Betriebssystemabbild Container arbeiten würden.

**Sandkasten:** Nach dem Start eines Containers werden alle Schreibaktionen, z. B. Dateisystemänderungen, Registrierungsänderungen oder Software-Installationen, auf dieser Sandkastenebene erfasst.

**Containerbetriebssystem-Image:** Container werden mithilfe von Images bereitgestellt. Das Betriebssystemabbild Container ist möglicherweise viele Bildebenen, aus denen ein Container der ersten Ebene. Grafik für das Betriebssystem. Ein Container-Betriebssystemabbild ist unveränderlich, kann nicht geändert werden.

**Containerrepository:** Bei jeder Erstellung eines Containerimages werden das Containerimage und dessen Abhängigkeiten in einem lokalen Repository gespeichert. Diese Images können auf dem Containerhost mehrfach wiederverwendet werden. Die Containerimages können auch in einer öffentlichen oder privaten Registrierung, wie z. B. DockerHub, gespeichert werden, damit sie in vielen verschiedenen Containerhosts verwendet werden können.

**Containerverwaltungstechnologie:** Windows-Container können mithilfe von PowerShell und Docker verwaltet werden. Mit einem dieser Tools können Sie neue Container und Containerimages erstellen sowie den Containerlebenszyklus verwalten.

<center>![](media/containerfund.png)</center>

## Container für Entwickler

Ein Entwickler den Desktop an einen Computer testen, auf einen Satz von Produktionscomputer, ein Docker Bild erstellt werden kann, die in Sekunden genauso wie in jeder Umgebung bereitgestellt wird. Dieser Artikel hat eine massive erstellt und wachsenden Ökosystem Anwendungstypen in Docker-Containern mit DockerHub, öffentliche Containern Anwendung Registrierung, die Docker derzeit mehr als 180.000 Applications im öffentlichen Community-Repository veröffentlichen verwaltet, verpackt.

Wenn Sie eine app containerize, werden nur die app und zum Ausführen der Anwendung erforderlichen Komponenten "Abbild" zusammengefasst. Container werden aus diesem Bild nach Bedarf erstellt. Sie können auch ein Bild als Basislinie verwenden, um ein anderes Bild zu erstellen, wodurch schnellere Erstellung von Abbildern. Mehrere Container können dasselbe Image, freigeben, d. h. Container sehr schnell starten und weniger Ressourcen. Beispielsweise können Sie Container ein leicht Hochfahren und portable app Komponenten – oder "Micro-Dienste" – für apps verteilt und schnell skalieren, jeden Dienst einzeln.

Da dem Container alles zum Ausführen der Anwendung benötigt ist, sie sind sehr leicht portieren und können auf jedem Computer, auf Windows Server 2016 ausführen. Sie können erstellen Testcontainer lokal, und, dasselbe Abbild für den Container für private Clouds, öffentliche Cloud oder Dienstanbieter Ihres Unternehmens bereitstellen. Die natürliche Flexibilität von Containern moderne app Entwicklungsmustern in großem Maßstab unterstützt, virtualisiert und cloud-Umgebungen.

Mit Containern können Entwickler eine Anwendung in jeder Sprache erstellen. Diese apps sind vollständig portabel und können überall - Laptop, Desktop, Server, private Clouds, öffentliche Cloud oder Dienstanbieter - Code unverändert ausgeführt werden.

Container können Entwickler erstellen und höherer Qualität Applications schneller anzubieten.

## Container für IT-Experten

Container können IT-Experten standardisierte Umgebung für ihre Entwicklung, QA und Produktions-Teams bereitstellen. Sie müssen nicht mehr über komplexe Installations- und Konfigurationsschritte erforderlich machen. Mithilfe von Containern abstrahieren Systemadministratoren abwesend Unterschiede in Betriebssysteminstallationen versorgen und zugrunde liegende Infrastruktur.

Container können Administratoren eine Infrastruktur zu erstellen, die einfacher zu aktualisieren und zu warten ist.

## Video-Überblick

<iframe 
src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player#ccLang=de" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>


## Wiederholen Sie den Windows Server-Container

[Erste Schritte mit Windows-Containern in Azure](../quick_start/azure_setup.md)  
[Erste Schritte mit Windows-Containern in lokalen Umgebungen](../quick_start/container_setup.md)






<!--HONumber=Feb16_HO3-->


