---
title: Containerökosystem
description: Aufbauen eines Containerökosystems.
keywords: Metadaten, Container
author: PatrickLang
ms.date: 04/20/2016
ms.topic: about-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 29fbe13a-228a-4eaa-9d4d-90ae60da5965
ms.openlocfilehash: c30b28d867a23537699bce99fafa2748d3747fa9
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576101"
---
# <a name="building-a-container-ecosystem"></a>Aufbauen eines containerökosystems

Um zu verstehen, warum ein Ökosystem Container erstellen so wichtig ist, zunächst sprechen wir über Docker.

## <a name="docker"></a>Docker

Das Konzept von Containern (Isolation von Namespaces und Ressourcenkontrolle) ist nicht wirklich neu und geht zurück auf BSD Jails, Solaris Zones und den grundlegenden UNIX-Mechanismus „chroot (change root)“.   Was Docker beigetragen hat, ist ein allgemeines Toolset, ein Paketerstellungsmodell und ein Bereitstellungsmechanismus.  Dadurch vereinfacht Docker die containerisierung und Verteilung von Clientanwendungen.  Diese Anwendungen können dann überall auf beliebigen Linux-Hosts ausgeführt werden. Diese Funktionalität bieten wir auch unter Windows.

Weit verbreitete Packaging-Modell und Bereitstellung Technologie vereinfacht die Verwaltung durch das anbieten derselben Verwaltungsbefehle auf beliebigen Hosts und ebnet Weg für reibungslose DevOps.

Ein Entwickler den Desktop an einen Computer testen, auf einen Satz von Produktionscomputer, ein Docker Bild erstellt werden kann, die in Sekunden genauso wie in jeder Umgebung bereitgestellt wird. Mittlerweile gibt es eine riesiges und weiter wachsendes Ökosystem von Anwendungen, die in Docker-Containern gepackt sind, und zwar in DockerHub, der von Docker verwalteten öffentlichen containerisierten Anwendungsregistrierung.

Docker bietet eine überzeugende Grundlage für die Entwicklung.

Nun wollen wir uns mit diesem Ökosystem und damit beschäftigen, wie Sie auf Docker-Konzepten aufbauen können, um einen Entwicklungs- und Bereitstellungsworkflow entsprechend Ihren Anforderungen einzurichten.

## <a name="components-in-a-container-ecosystem"></a>Komponenten eines Containerökosystems

Windows-Container sind eine wichtige Komponente eines großen containerökosystems. Wir arbeiten mit der gesamten Branche zusammen, um Entwicklern Optionen auf allen Ebenen des Lösungsstapels zu bieten.

Das Containerökosystem bietet Methoden zum Verwalten von Containern, Freigeben von Containern und Entwickeln von in Containern ausgeführten Apps.

![](media/containerEcosystem.png)

Microsoft möchte bei der Entwicklung dieser Apps der nächsten Generation Entwicklern mehr Optionen und Produktivität bieten.  Unser Ziel ist das Steigern der Entwicklerproduktivität, was bedeutet, dass Anwendungen für beliebige Microsoft-Clouds geschrieben werden können, ohne das Code geändert, neu geschrieben oder neu konfiguriert werden muss.

Microsoft setzt sich für ein offenes Ökosystem ein.  Wir unterstützen aktiv das Zusammenkommen mehrerer wichtiger Ökosysteme für Entwickler, wie z.B. Windows und Linux, um Innovationen voranzutreiben.

In den nächsten Monaten werden wir weitere Informationen zu zusätzliche Partnern in diesem sich entwickelnden Ökosystem veröffentlichen.
