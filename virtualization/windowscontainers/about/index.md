---
title: Informationen zu Windows-Containern
description: "Erfahren Sie mehr über Windows-Container."
keywords: Docker, Container
author: taylorb-microsoft
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: b916b8bb2e09dfc78414785ad0d0252b5abec619
ms.sourcegitcommit: b578961db242f08261798d1b498b091b8c405924
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 10/27/2017
---
# <a name="windows-containers"></a>Windows-Container

## <a name="what-are-containers"></a>Was sind Container?

Container sind eine Möglichkeit, eine Anwendung isoliert einzupacken. Die Anwendung, die sich in einem Container befindet, weiß nichts von anderen Anwendungen oder Prozessen, die außerhalb des Containers existieren. Alles, was für die erfolgreiche Ausführung der Anwendung erforderlich ist, befindet sich ebenfalls in diesem Container.  Und wenn der Container verschoben wird, hat das keine Auswirkungen auf die Anwendung, da alles enthalten ist, damit sie ausgeführt werden kann.

Stellen Sie sich eine Küche vor. Wir verpacken alle Geräte, Möbelstücke, sämtliches Geschirr, das Spülmittel und die Geschirrtücher. Das ist unser Container.

<center style="margin: 25px">![](media/box1.png)</center>

Diesen Container können wir in jede beliebige Wohnung stellen, die Küche ist immer die gleiche. Wir brauchen lediglich einen Strom- und Wasseranschluss und können sofort etwas kochen (da alle Geräte, die wir brauchen, vorhanden sind).

<center style="margin: 25px">![](media/apartment.png)</center>

Mit Containern verhält es sich in gewisser Weise wie mit dieser Küche. Die Räume können unterschiedlich oder auch gleich sein. Wichtig ist, dass der Container alles umfasst, was benötigt wird.

Sehen Sie sich eine kurze Übersicht an: [Windows-based containers: Modern app development with enterprise-grade control](https://youtu.be/Ryx3o0rD5lY) (Windows-basierte Container: Moderne App-Entwicklung mit Steuerung auf Unternehmensniveau).

## <a name="container-fundamentals"></a>Grundlegendes zu Containern

Container sind eine isolierte, von Ressourcen gesteuerte und portierbare Laufzeitumgebung, die auf einem Hostcomputer oder virtuellen Computer ausgeführt wird. Anwendungen oder Prozesse, die in Containern ausgeführt werden, sind mit allen erforderlichen Abhängigkeiten und Konfigurationsdateien verpackt, so als ob außerhalb des Containers keine anderen Prozesse ausgeführt werden würden.

Der Host des Containers stellt einen Satz von Ressourcen für den Container bereit, und der Container verwendet nur diese Ressourcen. Für den Container existieren keine anderen Ressourcen als die angegebenen. Er kommt somit mit keinen Ressourcen in Berührung, die für einen anderen Container bereitgestellt wurden.

Die folgenden grundlegenden Konzepte können nützlich sein, wenn Sie beginnen, Windows-Container zu erstellen und damit zu arbeiten.

**Containerhost:** Physisches oder virtuelles Computersystem, das mit dem Windows-Containerfeature konfiguriert wurde. Auf dem Containerhost werden ein oder mehrere Windows-Container ausgeführt.

**Containerimage:** Wenn z.B. durch eine Softwareinstallation das Dateisystem oder die Registrierung eines Containers geändert wird, werden diese Änderungen in einem Sandkasten erfasst. In vielen Fällen ist es wünschenswert, diesen Zustand zu erfassen, damit neue Container erstellt werden können, die diese Änderungen übernehmen. Das ist, was ein Image ausmacht. Sobald der Container beendet wurde, können Sie entweder diesen Sandkasten verwerfen oder ihn in ein neues Containerimage umwandeln. Angenommen, Sie haben mithilfe des Windows Server Core-Betriebssystemimages einen Container bereitgestellt. Anschließend installieren Sie MySQL in diesem Container. Das Erstellen eines neuen Images mithilfe dieses Containers resultiert in einer bereitstellbaren Version des Containers. Dieses Image enthält nur die erfolgten Änderungen (MySQL), fungiert jedoch als Ebene über dem Containerbetriebssystem-Image.

**Sandkasten:** Nach dem Start eines Containers werden alle Schreibaktionen, z.B. Dateisystemänderungen, Registrierungsänderungen oder Softwareinstallationen, auf dieser Sandkastenebene erfasst.

**Containerbetriebssystem-Image:** Container werden mithilfe von Images bereitgestellt. Das Containerbetriebssystem-Image ist die erste Ebene von möglicherweise vielen Imageebenen, die einen Container bilden. Das Image stellt die Betriebssystemumgebung bereit. Das Betriebssystemimage eines Containers ist unveränderlich. Das heißt, es bleibt immer gleich.

**Containerrepository:** Bei jeder Erstellung eines Containerimages werden das Containerimage und dessen Abhängigkeiten in einem lokalen Repository gespeichert. Diese Images können auf dem Containerhost mehrfach wiederverwendet werden. Die Containerimages können auch in einer öffentlichen oder privaten Registrierung, wie z. B. DockerHub, gespeichert werden, damit sie in vielen verschiedenen Containerhosts verwendet werden können.

<center>![](media/containerfund.png)</center>

Wenn Sie sich mit virtuellen Computern auskennen, werden Ihnen die Ähnlichkeiten mit Containern auffallen. Ein Container, in dem ein Betriebssystem ausgeführt wird, verfügt über ein Dateisystem, und auf ihn kann über ein Netzwerk so zugegriffen werden, als handele es sich um ein physisches oder virtuelles Computersystem. Was Technologie und Konzepte anbelangt, unterscheiden sich Container jedoch sehr von virtuellen Computern.

Mark Russinovich, Experte für Microsoft Azure, erklärt diese Unterschiede in einem [großartigen Blogbeitrag](https://azure.microsoft.com/en-us/blog/containers-docker-windows-and-trends/).

## <a name="windows-container-types"></a>Windows-Containertypen

Windows-Container enthalten zwei verschiedene Containertypen bzw. Laufzeiten (Runtimes).

**Windows Server-Container**: – Bieten Anwendungsisolation mithilfe einer Technologie zum Isolieren von Prozessen und Namespaces. Ein Windows Servercontainer teilt sich einen Kernel mit dem Containerhost und allen Containern, die auf dem Host ausgeführt werden. Diese Container bieten keine schädlichen Sicherheitsgrenzen und sollten nicht zum Isolieren von nicht vertrauenswürdigen Codes verwendet werden. Aufgrund des gemeinsam genutzten Kernelspeichers benötigen diese Container die gleiche Kernelversion und -konfiguration.

**Hyper-V-Isolierung**: Erweitert die von Windows Server-Containern bereitgestellte Isolierung, indem jeder Container in einem hochgradig optimierten virtuellen Computer ausgeführt wird. Bei dieser Konfiguration wird der Kernel des Containerhosts nicht gemeinsam mit anderen Containern desselben Hosts verwendet. Diese Container wurden für schädliches mandantenfähiges Hosting mit der gleichen Sicherheitsgarantie wie der eines virtuellen Computers entwickelt. Da diese Container den Kernel nicht mit dem Host oder mit anderen Containern auf dem Host teilen, können sie Kernel mit verschiedenen Versionen und Konfigurationen (in unterstützten Versionen) ausführen – z.B. wenn alle Windows-Container unter Windows10 Hyper-V-Isolierung verwenden, um die Version und Konfiguration des Windows Server-Kernels zu nutzen.

Die Ausführung eines Containers unter Windows mit oder ohne Hyper-V-Isolierung ist eine Entscheidung während der Laufzeit. Sie können den Container auch zunächst mit Hyper-V-Isolierung erstellen und ihn später zur Laufzeit stattdessen als Windows Server-Container ausführen.

## <a name="what-is-docker"></a>Was ist Docker?

Informationen zu Containern erwähnen zwangsläufig auch Docker. Docker ist der Behälter, in dem die Containerimages verpackt und ausgeliefert werden. Dieser automatisierte Prozess erstellt Images (im Prinzip Vorlagen), die überall – lokal, in der Cloud oder auf einem persönlichen Computer – als Container ausgeführt werden können.

<center>![](media/docker.png)</center>

Ein Windows Server-Container kann wie jeder andere Container mit [Docker](https://www.docker.com) verwaltet werden.

## <a name="containers-for-developers"></a>Container für Entwickler ##

Ein Entwickler den Desktop an einen Computer testen, auf einen Satz von Produktionscomputer, ein Docker Bild erstellt werden kann, die in Sekunden genauso wie in jeder Umgebung bereitgestellt wird. Mittlerweile gibt es ein riesiges und weiter wachsendes Ökosystem von Anwendungen, die in Docker-Containern gepackt sind. In DockerHub, der von Docker verwalteten öffentlichen containerisierten Anwendungsregistrierung, werden im öffentlichen Repository der Community derzeit über 180.000Anwendungen veröffentlicht.

Wenn Sie eine App containerisieren, werden nur die App und zum Ausführen der App benötigten Komponenten zu einem „Image“ kombiniert. Container werden anschließend nach Bedarf anhand dieses Images erstellt. Sie können ein Image auch als Basis für das Erstellen eines weiteren Images nutzen, wodurch das Erstellen von Images noch schneller erfolgt. Mehrere Container können dasselbe Image nutzen, was heißt, dass Container sehr schnell starten und weniger Ressourcen nutzen. Sie können beispielsweise Container zum Einrichten von schlanken und portierbaren App-Komponenten bzw. „Microservices“ für verteilte Apps einsetzen und jeden Dienst schnell getrennt skalieren.

Da Container über alles verfügen, was zum Ausführen Ihrer Anwendung erforderlich ist, sind sie überaus portierbar und können auf allen Computern mit ausgeführtem Windows Server 2016 betrieben werden. Sie können Container lokal testen und bereitstellen und anschließend dasselbe Containerimage in der privaten Cloud Ihres Unternehmens, einer öffentlichen Cloud oder bei einem Dienstanbieter bereitstellen. Die natürliche Agilität von Containern unterstützt Muster der Entwicklung moderner Apps in riesigen, virtualisierten und Cloudumgebungen.

Mit Containern können Entwickler eine App in jeder Sprache erstellen. Diese Apps sind vollständig portierbar und können überall (Laptop, Desktop, Server, private Cloud, öffentliche Cloud oder Dienstanbieter) ohne Codeänderungen ausgeführt werden.  

Container helfen Entwicklern, Anwendungen höherer Qualität schneller zu entwickeln und auszuliefern.

## <a name="containers-for-it-professionals"></a>Container für IT-Experten ##

IT-Experten können Container verwenden, um standardisierte Umgebungen für ihre Entwicklungs-, Qualitätssicherungs- und Produktionsteams bereitzustellen. Sie müssen sich nicht mehr komplexe Installations- und Konfigurationsschritte kümmern. Mithilfe von Containern können Systemadministratoren Unterschiede bei Betriebssysteminstallationen und zugrunde liegender Infrastruktur abstrahieren.

Container können Administratoren eine Infrastruktur zu erstellen, die einfacher zu aktualisieren und zu warten ist.

## <a name="video-overview"></a>Video-Überblick

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## <a name="try-windows-server-containers"></a>Windows Server-Container testen

Möchten Sie die großartigen Möglichkeiten von Containern nutzen? Im Folgenden finden Sie Informationen zum Bereitstellen Ihres ersten Containers: <br/>
Benutzer von Windows Server: [Schnellstartanleitung für Windows Server](../quick-start/quick-start-windows-server.md) <br/>
Benutzer von Windows 10: [Schnellstartanleitung für Windows 10](../quick-start/quick-start-windows-10.md)

