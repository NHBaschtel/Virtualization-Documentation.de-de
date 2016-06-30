---
title: "Windows-Container – häufig gestellte Fragen"
description: "Windows-Container – häufig gestellte Fragen"
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
translationtype: Human Translation
ms.sourcegitcommit: cfa3c14e932f8b86edf6667200ac028ea0a16b67
ms.openlocfilehash: c3a7decaf087741c82419a8a541867ae01f0f4da

---

# Häufig gestellte Fragen

## Informationen zu Windows-Containern

**Was ist ein Windows Server-Container?**

Windows Server-Container stellen eine schlanke Methode zur Betriebssystemvirtualisierung dar, die zum Trennen von Anwendungen und Diensten von anderen Diensten verwendet wird, die auf demselben Containerhost ausgeführt werden. Um dies zu ermöglichen, verfügt jeder Container über eine eigene Sicht über Betriebssystem, Prozesse, Dateisystem, Registrierung und IP-Adressen.  

**Was ist ein Hyper-V-Container?**

Sie können sich einen Hyper-V-Container als Windows Server-Container vorstellen, der in einer Hyper-V-Partition ausgeführt wird.

Hyper-V-Container bieten eine zusätzliche Bereitstellungsoption zwischen hochgradig effizienten, HD-Windows Server-Container und der hoch isolierten virtualisierten Hardware Hyper-V virtuelle Computer. Umgebungen, in denen andere Clientanwendungen auf demselben Host Vertrauensgrenzen, möglicherweise zusätzliche Isolierung erforderlich. Hyper-V-Container bieten höheren Isolationsstufe, die mithilfe einer optimierten Virtualisierung und Windows Server-Betriebssystem, das Container voneinander und vom Host-Betriebssystem trennt. Beide Bereitstellungsoptionen für Container nutzen dieselben Verwaltungs-APIs, Tools und Imageformate. Zum Zeitpunkt der Bereitstellung können Kunden einfach den Bereitstellungsmodus wählen, der ihre Anforderungen am besten erfüllt.

**Was ist der Unterschied zwischen Linux- und Windows Server-Containern?**

Linux- und Windows Server-Container ähneln sich, denn beide implementieren ähnliche Technologien in ihrem Kernel und Kernbetriebssystem. Der Unterschied ergibt sich aus der Plattform und Arbeitslasten, die innerhalb des Containers ausgeführt.  
Wenn ein Kunde Windows Server-Container verwendet, können sie vorhandene Windows-Technologien wie .NET, ASP.NET, PowerShell und vieles mehr integrieren.

**Muss ich als Entwickler meine App für jeden Containertyp umschreiben?**

Nein. Windows-Containerimages sind für Windows Server-Container und Hyper-V-Container gleich. Die Auswahl des Containertyps erfolgt beim Starten des Containers. Aus Sicht der Entwickler sind Windows Server-Containern und Hyper-V zwei Arten von dasselbe.  Diese Entwicklung, Programmierung und Verwaltung genauso bieten, werden offene und erweiterbare und das gleiche Maß an Integration und Support über Docker enthält.

Ein Entwickler kann ein Container-Abbild, das mit einem Windows Server-Container erstellen und als Hyper-V-Container oder umgekehrt unverändert als das Festlegen der entsprechenden Common Language Runtime-Flag bereitstellen.

Windows Server-Containern bietet höhere Dichte und Leistung (z. B. niedrigere dreht sich Zeit, schnellere Common Language Runtime-Leistung im Vergleich zu geschachtelten Konfigurationen) Wenn Geschwindigkeit Schlüssel ist. Hyper-V-Container bieten mehr Isolation sicherzustellen, dass in einem Container ausgeführte Code kann nicht gefährden oder Auswirkungen auf das Hostbetriebssystem oder andere Container, die auf dem gleichen Host ausgeführt wird. Dies ist hilfreich für mehrinstanzenfähige Szenarien (mit Anforderungen für das Hosten von nicht vertrauenswürdigem Code), einschließlich SaaS-Anwendungen und Computehosting.

**Sind Hyper-V-/Windows Server-Container ein Add-On, oder werden sie in Windows Server integriert?**

Die Containerfunktionen werden in Windows Server 2016 integriert. Bleiben Sie dran, Weitere Informationen näher auf die allgemeine Verfügbarkeit.  

**Welche Beziehung besteht zwischen Windows Server-Containern und Drawbridge?**

Drawbridge war eines der vielen Forschungsprojekte, die uns wertvolle Einblicke in Container ermöglicht haben.  Ein Großteil der Containertechnologie in Windows Server 2016 basiert auf den Erfahrungen mit Drawbridge, und wir freuen uns, unseren Kunden in Windows Server 2016 erstklassige Containertechnologien bereitstellen zu können.

**Was sind die Voraussetzungen für Windows Server- und Hyper-V-Container?**

Windows Server- und Hyper-V-Container erfordern Windows Server 2016. Diese Technologien funktioniert nicht mit früheren Versionen von Windows.


## Windows-Containerverwaltung

**Werden Hyper-V-Container auch für das Docker-Ökosystem verfügbar sein?**

Ja. Hyper-V-Container bietet dieselben Integrations- und Verwaltungsmöglichkeiten mit Docker wie Windows Server-Container.  Das Ziel ist eine offene, konsistente, plattformübergreifende Erfahrung haben.  
Die Plattform Docker erheblich vereinfachen und verbessern die Erfahrung der Arbeit über unser Container-Optionen. Eine Anwendung entwickelt wurde, mithilfe von Windows Server-Container kann als Hyper-V-Container ohne Änderung bereitgestellt werden.


**Kann ich Windows-Container auf ESXi oder einem anderen Nicht-Hyper-V-Hypervisor ausführen?**

Ja, Windows-Container können auf sämtlichen TP3 Server Core-Installationen ausgeführt werden.  Befolgen Sie die Anweisungen zum [direkten Aktivieren des Containerfeatures](../quick_start/inplace_setup.md).

## Das offene Ökosystem von Microsoft

**Nimmt Microsoft an der Open Container Initiative (OCI) teil?**

Um zu gewährleisten, dass das Paketerstellungsformat universell bleibt, hat Docker vor Kurzem die Open Container Initiative (OCI) ins Leben gerufen, um sicherzustellen, dass die Paketerstellung für Container ein offenes von den Gründern geleitetes Format bleibt, wobei Microsoft eines der Gründungsmitglieder ist.

**Sind Microsoft und Docker wirklich eine Partnerschaft eingegangen?**

Ja.  
Unsere Partnerschaft mit Docker ermöglicht Entwicklern das Erstellen, verwalten und Bereitstellen von Windows Server- und Linux-Container mit denselben Docker Tool. Entwickler für Windows Server müssen nicht mehr zwischen mithilfe der großen Bereich von Windows Server-Technologien und Erstellen von Sammelartikeleinheit auszuwählen.  

Docker ist zwei Dinge, die open-Source-Gruppe von Projekten und Docker des Unternehmens. Wir betrachten diese Partnerschaft zu beiden enthalten. Docker ist teilweise erfolgreich ist, aufgrund der dynamisches Ökosystem, das rund um die Docker Container-Technologie aufgebaut wurde. Microsoft ist zum Aktivieren der Unterstützung für Windows Server-Containern und Hyper-V Docker Projekt beitragen.  

Weitere Informationen finden Sie im Blogbeitrag [New Windows Server containers and Azure support for Docker](http://azure.microsoft.com/blog/2014/10/15/new-windows-server-containers-and-azure-support-for-docker/?WT.mc_id=Blog_ServerCloud_Announce_TTD) (Neue Windows Server-Container und Azure-Unterstützung für Docker).



<!--HONumber=Jun16_HO4-->


