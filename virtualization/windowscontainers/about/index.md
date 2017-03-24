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
translationtype: Human Translation
ms.sourcegitcommit: 59621ca2db190d5c13034752a08c291e3dc19daa
ms.openlocfilehash: d68be61b1d462b70986df5cfd6df052b388cec1d
ms.lasthandoff: 02/08/2017

---

# Windows-Container

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

## Was sind Container?

Sehen Sie sich eine kurze Übersicht an: [Windows-based containers: Modern app development with enterprise-grade control](https://youtu.be/Ryx3o0rD5lY) (Windows-basierte Container: Moderne App-Entwicklung mit Steuerung auf Unternehmensniveau).

Ein Container ist eine isolierte, von Ressourcen gesteuerte und portierbare Betriebsumgebung.

Im Grunde ist ein Container ein isolierter Ort, an dem eine Anwendung ohne Beeinflussung der Ressourcen (Speicher, Festplatte, Netzwerk usw.) anderer Container oder des Hosts ausgeführt werden kann.

Ein Container verhält sich wie ein neu installierter physischer oder virtueller Computer. Ein Windows-Servercontainer kann mit [Docker](https://www.docker.com/) wie jeder andere Container verwaltet werden.

## Windows-Containertypen

Windows-Container enthalten zwei verschiedene Containertypen bzw. Laufzeiten (Runtimes).

**Windows Server-Container**: – Bieten Anwendungsisolation mithilfe einer Technologie zum Isolieren von Prozessen und Namespaces. Ein Windows Servercontainer teilt sich einen Kernel mit dem Containerhost und allen Containern, die auf dem Host ausgeführt werden.

**Hyper-V-Container**: – Erweitern die von Windows Server-Containern bereitgestellte Isolation, indem jeder Container in einem hochgradig optimierten virtuellen Computer ausgeführt wird. Bei dieser Konfiguration wird der Kernel des Containerhosts nicht gemeinsam mit den Hyper-V-Containern verwendet.


## Grundlegendes zu Containern

Wenn Sie die Arbeit mit Containern aufnehmen, werden Sie viele Ähnlichkeiten zwischen einem Container und einem virtuellen Computer feststellen. Ein Container, in dem ein Betriebssystem ausgeführt wird, verfügt über ein Dateisystem, und auf ihn kann über ein Netzwerk so zugegriffen werden, als handele es sich um ein physisches oder virtuelles Computersystem. Allerdings unterscheiden sich die Technologie und Konzepte hinter Containern ganz wesentlich von denen virtueller Computer.  

[In diesem Blogbeitrag](http://azure.microsoft.com/blog/2015/08/17/containers-docker-windows-and-trends/) von Mark Russinovich werden Container aufschlussreich erläutert.

Die folgenden grundlegenden Konzepte können nützlich sein, wenn Sie beginnen, Windows-Container zu erstellen und damit zu arbeiten. 

**Containerhost:** Physisches oder virtuelles Computersystem, das mit dem Windows-Containerfeature konfiguriert wurde. Auf dem Containerhost werden ein oder mehrere Windows-Container ausgeführt.

**Containerimage:** Wenn z. B. durch eine Softwareinstallation das Dateisystem oder die Registrierung eines Containers geändert werden, werden diese Änderungen in einem Sandkasten erfasst.  In vielen Fällen ist es wünschenswert, diesen Zustand zu erfassen, damit neue Container erstellt werden können, die diese Änderungen übernehmen. Das ist, was ein Image ausmacht. Sobald der Container beendet wurde, können Sie entweder diesen Sandkasten verwerfen oder ihn in ein neues Containerimage umwandeln. Angenommen, Sie haben mithilfe des Windows Server Core-Betriebssystemimages einen Container bereitgestellt. Anschließend installieren Sie MySQL in diesem Container. Das Erstellen eines neuen Images mithilfe dieses Containers resultiert in einer bereitstellbaren Version des Containers. Dieses Image enthält nur die erfolgten Änderungen (MySQL), fungiert jedoch als Ebene über dem Containerbetriebssystem-Image.

**Sandkasten:** Nach dem Start eines Containers werden alle Schreibaktionen, z. B. Dateisystemänderungen, Registrierungsänderungen oder Softwareinstallationen, auf dieser Sandkastenebene erfasst.  
 
**Containerbetriebssystem-Image:** Container werden mithilfe von Images bereitgestellt. Das Containerbetriebssystem-Image ist die erste Ebene von möglicherweise vielen Imageebenen, die einen Container bilden. Das Image stellt die Betriebssystemumgebung bereit. Ein Containerbetriebssystem-Image ist unveränderlich.

**Containerrepository:** Bei jeder Erstellung eines Containerimages werden das Containerimage und dessen Abhängigkeiten in einem lokalen Repository gespeichert. Diese Images können auf dem Containerhost mehrfach wiederverwendet werden. Die Containerimages können auch in einer öffentlichen oder privaten Registrierung, wie z. B. DockerHub, gespeichert werden, damit sie in vielen verschiedenen Containerhosts verwendet werden können.

<center>![](media/containerfund.png)</center>

## Container für Entwickler

Ein Entwickler den Desktop an einen Computer testen, auf einen Satz von Produktionscomputer, ein Docker Bild erstellt werden kann, die in Sekunden genauso wie in jeder Umgebung bereitgestellt wird. Mittlerweile gibt es ein riesiges und weiter wachsendes Ökosystem von Anwendungen, die in Docker-Containern gepackt sind. In DockerHub, der von Docker verwalteten öffentlichen containerisierten Anwendungsregistrierung, werden im öffentlichen Repository der Community derzeit über 180.000 Anwendungen veröffentlicht.  

Wenn Sie eine App containerisieren, werden nur die App und zum Ausführen der App benötigten Komponenten zu einem „Image“ kombiniert. Container werden anschließend nach Bedarf anhand dieses Images erstellt. Sie können ein Image auch als Basis für das Erstellen eines weiteren Images nutzen, wodurch das Erstellen von Images noch schneller erfolgt.  Mehrere Container können dasselbe Image nutzen, was heißt, dass Container sehr schnell starten und weniger Ressourcen nutzen. Sie können beispielsweise Container zum Einrichten von schlanken und portierbaren App-Komponenten bzw. „Microservices“ für verteilte Apps einsetzen und jeden Dienst schnell getrennt skalieren.

Da Container über alles verfügen, was zum Ausführen Ihrer Anwendung erforderlich ist, sind sie überaus portierbar und können auf allen Computern mit ausgeführtem Windows Server 2016 betrieben werden. Sie können Container lokal testen und bereitstellen und anschließend dasselbe Containerimage in der privaten Cloud Ihres Unternehmens, einer öffentlichen Cloud oder bei einem Dienstanbieter bereitstellen. Die natürliche Agilität von Containern unterstützt Muster der Entwicklung moderner Apps in riesigen, virtualisierten und Cloudumgebungen.

Mit Containern können Entwickler eine App in jeder Sprache erstellen. Diese Apps sind vollständig portierbar und können überall (Laptop, Desktop, Server, private Cloud, öffentliche Cloud oder Dienstanbieter) ohne Codeänderungen ausgeführt werden.  

Container helfen Entwicklern, Anwendungen höherer Qualität schneller zu entwickeln und auszuliefern.

## Container für IT-Experten ##

IT-Experten können Container verwenden, um standardisierte Umgebungen für ihre Entwicklungs-, Qualitätssicherungs- und Produktionsteams bereitzustellen. Sie müssen sich nicht mehr komplexe Installations- und Konfigurationsschritte kümmern. Mithilfe von Containern können Systemadministratoren Unterschiede bei Betriebssysteminstallationen und zugrunde liegender Infrastruktur abstrahieren.

Container können Administratoren eine Infrastruktur zu erstellen, die einfacher zu aktualisieren und zu warten ist.

## Video-Überblick

<iframe 
src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>


## Wiederholen Sie den Windows Server-Container

[Schnellstarteinführung für Container](../quick_start/quick_start.md)


