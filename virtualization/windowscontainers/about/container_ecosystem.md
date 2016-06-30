---
title: "Containerökosystem"
description: "Aufbauen eines Containerökosystems."
keywords: metadata, containers
author: scooley
manager: timlt
ms.date: 04/20/2016
ms.topic: about-article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 29fbe13a-228a-4eaa-9d4d-90ae60da5965
translationtype: Human Translation
ms.sourcegitcommit: cfa3c14e932f8b86edf6667200ac028ea0a16b67
ms.openlocfilehash: 1aad093c2c4e1c64200fcc1c10cbdee6c93b80c5

---

# Aufbauen eines Containerökosystems

Um zu verstehen, warum ein Ökosystem Container erstellen so wichtig ist, zunächst sprechen wir über Docker.

## Docker der Beschwerde

Das Konzept von Containern (Namespace Isolierung und Ressource Governance) wurde für lange Zeit schon zurückgehen BSD Jails, Solaris-Zonen und der grundlegenden UNIX Chroot (Change-Stamm)-Mechanismus.   Also Docker ist ein gemeinsamer Satz von Tools, Paketerstellungsmodell und Bereitstellungsmechanismus bereitzustellen.  Auf diese Weise vereinfacht Docker erheblich die Containerization und die Verteilung von Clientanwendungen.  Diese Anträge können dann an einer beliebigen Stelle auf einem Linux-Host eine Funktion ausführen, die wir unter Windows als auch bereitstellen.

Diese weit verbreitete Technologie vereinfacht nicht nur die Verwaltung von bietet die gleichen Management-Befehle für alle Hosts, erstellt auch eine einmalige Gelegenheit für eine nahtlose DevOps.

Ein Entwickler den Desktop an einen Computer testen, auf einen Satz von Produktionscomputer, ein Docker Bild erstellt werden kann, die in Sekunden genauso wie in jeder Umgebung bereitgestellt wird. Dieser Artikel hat eine massive erstellt und wachsenden Ökosystem von Clientanwendungen verpackt in Docker-Containern mit DockerHub, öffentliche Containern Anwendung Registrierung, die Docker verwaltet.

Docker bietet eine großartige Grundlage für die Entwicklung.

Jetzt sehen wir reden, Ökosystem Anwendungstypen und wie Sie Docker Konzepte zum Erstellen eines Workflows für Entwicklung und Bereitstellung erstellen können an Ihre Bedürfnisse geeignet ist.


## Komponenten eines Containerökosystems

Windows-Container sind eine wichtige Komponente eines großen Containerökosystems. Wir arbeiten mit der gesamten Branche zusammen, um Entwicklern Optionen auf allen Ebenen des Lösungsstapels zu bieten.

Das Container-Ökosystem bietet Methoden zum Container verwalten, Freigeben von Containern und Entwickeln von apps, die in Containern ausgeführt.

![](media/containerEcosystem.png)

Microsoft möchte Developer-Auswahl und Produktivität zu ermöglichen, während sie diese apps der nächsten Generation erstellen.  Unser Ziel ist es, Treibstoff Entwicklerproduktivität, d. ermöglichen des Anwendungszugriffs h. auf die Cloud von Microsoft ohne ändern, schreiben oder neu konfigurieren, Code als Ziel.

Microsoft ist bestrebt, die geöffnet wird und das Ökosystem Anzeigenamen.  Die Einführung von mehrere Entwickler Ökosysteme von Interesse – wie Windows und Linux – Innovationen zusammen unterstützt aktiv.

In den nächsten Monaten werden wir weitere Informationen zu zusätzliche Partnern in diesem sich entwickelnden Ökosystem veröffentlichen.



<!--HONumber=Jun16_HO4-->


