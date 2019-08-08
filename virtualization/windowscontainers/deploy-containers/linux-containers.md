---
title: Linux-Container unter Windows
description: Erfahren Sie mehr über die verschiedenen Möglichkeiten, wie Sie mit Hyper-V Linux-Container unter Windows ausführen können, als ob Sie systemeigen sind.
keywords: LCOW, Linux-Container, Docker, Container
author: scooley
ms.date: 11/02/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 0426b14c423c06a0f12ea91529ce794f7a972f47
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998477"
---
# <a name="linux-containers-on-windows"></a>Linux-Container unter Windows

Linux-Container bilden einen großen Prozentsatz des gesamten Container-Ökosystems und sind sowohl für Entwickler-als auch für Produktionsumgebungen von grundlegender Bedeutung.  Da Container einen Kernel mit dem Container-Host teilen, ist es jedoch nicht möglich[*](linux-containers.md#other-options-we-considered), Linux-Container direkt unter Windows zu verwenden.  Hier kommt die Virtualisierung zum Vorbild.

Im Moment gibt es zwei Möglichkeiten, Linux-Container mit docker für Windows und Hyper-V auszuführen:

1. Führen Sie Linux-Container in einer vollständigen Linux-VM aus – das ist das, was docker in der Regel heute macht.
1. Ausführen von Linux-Containern mit [Hyper-V-Isolierung](../manage-containers/hyperv-container.md) (LCOW) – Dies ist eine neue Option in docker für Windows.

In diesem Artikel wird erläutert, wie die einzelnen Ansätze funktionieren, bietet Anleitungen dazu, wann Sie welche Lösung auswählen und welche Arbeit in Bearbeitung ist.

## <a name="linux-containers-in-a-moby-vm"></a>Linux-Container in einer Moby-VM

Wenn Sie Linux-Container auf einem Linux-Computer ausführen möchten, folgen Sie den Anweisungen im Leitfaden für den Einstieg in das [docker-Handbuch](https://docs.docker.com/docker-for-windows/).

Andocker konnte Linux-Container auf dem Windows-Desktop ausführen, da es zunächst in 2016 (bevor Hyper-v-Isolierung oder LCOW verfügbar waren) mithilfe eines auf Hyper-v ausgeführten [LinuxKit](https://github.com/linuxkit/linuxkit) -basierten virtuellen Computers veröffentlicht wurde.

In diesem Modell wird der andocker-Client auf dem Windows-Desktop ausgeführt, ruft aber den andocker-Daemon auf der Linux-VM auf.

![Moby VM als Container-Host](media/MobyVM.png)

In diesem Modell geben alle Linux-Container einen einzigen Linux-basierten Container-Host und alle Linux-Container frei:

* Sie können einen Kernel für einander und die Moby-VM freigeben, aber nicht für den Windows-Host.
* Konsistente Speicher-und Netzwerkeigenschaften mit Linux-Containern, die unter Linux ausgeführt werden (da Sie auf einer Linux-VM ausgeführt werden).

Das bedeutet auch, dass der Linux-Container-Host (Moby VM) den docker-Daemon und alle Abhängigkeiten des Andock-Daemons ausführen muss.

Wenn Sie feststellen möchten, ob Sie mit Moby VM ausgeführt werden, überprüfen Sie den Hyper-v-Manager für Moby VM entweder mithilfe der `Get-VM` Hyper-v-Manager-Benutzeroberfläche oder durch Ausführen in einem erweiterten PowerShell-Fenster.

## <a name="linux-containers-with-hyper-v-isolation"></a>Linux-Container mit Hyper-V-Isolierung

Um LCOW zu testen, folgen Sie den Anweisungen für Linux-Container in [diesem Leitfaden für erste Schritte](../quick-start/quick-start-windows-10.md) .

Linux-Container mit Hyper-V-Isolierung führen jeden Linux-Container (LCOW) in einer optimierten Linux-VM mit nur genügend Betriebssystem zum Ausführen von Containern aus.  Im Gegensatz zum Moby VM-Ansatz verfügt jede LCOW über einen eigenen Kernel und eine eigene VM-Sandbox.  Sie werden auch direkt von Docker unter Windows verwaltet.

![Linux-Container mit Hyper-V-Isolierung (LCOW)](media/lcow-approach.png)

Wenn Sie sich genauer anschauen, wie sich die Container Verwaltung zwischen dem Moby VM-Ansatz und der LCOW unterscheidet, bleibt die Container Verwaltung im LCOW-Modell unter Windows, und jede LCOW-Verwaltung erfolgt über GRPC und Container.  Das bedeutet, dass die Linux-Distributions Container, die für LCOW verwendet werden, viel kleiner sein können.  Gerade jetzt verwenden wir LinuxKit für optimierte Distributions Container, aber auch andere Projekte wie Kata bauen ähnlich hoch abgestimmte Linux-Distributionen (Clear Linux) auf.

Hier sehen Sie sich die einzelnen LCOW genauer an:

![LCOW-Architektur](media/lcow.png)

Navigieren Sie zu, um festzustellen, ob Sie `C:\Program Files\Linux Containers`LCOW ausführen. Wenn docker für die Verwendung von LCOW konfiguriert ist, gibt es einige Dateien, die die minimale LinuxKit-Distribution enthalten, die in jedem Container ausgeführt wird, der unter Hyper-V-Isolierung ausgeführt wird.  Beachten Sie, dass die optimierten VM-Komponenten kleiner als 100 MB sind, viel kleiner als das LinuxKit-Bild in Moby VM.

### <a name="work-in-progress"></a>In Bearbeitung

LCOW befindet sich unter aktiver Entwicklung. Nachverfolgen des Fortschritts im Moby-Projekt auf [GitHub](https://github.com/moby/moby/issues/33850)

#### <a name="bind-mounts"></a>Binden von Bereitstellungen

Beim Binden der Bereitstellung von Volumes mit `docker run -v ...` werden die Dateien im Windows NTFS-Dateisystem gespeichert, sodass für POSIX-Vorgänge einige Übersetzung erforderlich ist. Einige Dateisystemvorgänge werden derzeit teilweise oder gar nicht implementiert, was bei einigen Anwendungen zu Inkompatibilitäten führen kann.

Folgende Vorgänge funktionieren momentan nicht für gebunden bereitgestellte Volumes:

* MkNod
* XAttrWalk
* XAttrCreate
* Lock
* Getlock
* Auth
* Flush
* INotify

Es gibt auch einige Vorgänge, die nicht vollständig implementiert werden:

* GetAttr – die Nlink-Zahl wird immer als 2 gemeldet
* Open – nur ReadWrite-, WriteOnly- und ReadOnly-Flags werden implementiert

Diese Anwendungen erfordern eine Datenträgerzuordnung und werden nicht gestartet oder ordnungsgemäß ausgeführt.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>Zusätzliche Informationen

[Andocker-Blog, der LCOW beschreibt](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux-Container Video](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-Kernel Plus Build-Anweisungen](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Verwendungszwecke für Moby VM vs LCOW

### <a name="when-to-use-moby-vm"></a>Verwendung von Moby VM

Im Moment empfehlen wir die Moby VM-Methode zum Ausführen von Linux-Containern für Personen, die:

- Sie benötigen eine stabile Containerumgebung.  Dies ist der andocker für Windows-Standard.
- Führen Sie Windows-oder Linux-Container, aber selten beide gleichzeitig aus.
- Komplizierte oder benutzerdefinierte Netzwerkanforderungen zwischen Linux-Containern
- Sie benötigen keine Kernel Isolierung (Hyper-V-Isolierung) zwischen Linux-Containern.

### <a name="when-to-use-lcow"></a>Verwendung von LCOW

Im Moment empfehlen wir LCOW Personen, die:

- Möchten Sie unsere neueste Technologie testen?
- Führen Sie Windows-und Linux-Container gleichzeitig aus.
- Kernel Isolierung (Hyper-V-Isolierung) zwischen Linux-Containern erforderlich.

## <a name="other-options-we-considered"></a>Andere Optionen, die wir berücksichtigt haben

Als wir nach Möglichkeiten suchten, Linux-Container unter Windows auszuführen, haben wir die WSL als WSL betrachtet. Letztendlich haben wir einen auf Virtualisierung basierenden Ansatz gewählt, damit Linux-Container unter Windows mit Linux-Containern unter Linux konsistent sind. Die Verwendung von Hyper-V macht LCOW auch sicherer. Wir können in Zukunft erneut eine Evaluierung durchführen, aber im Moment wird LCOW weiterhin Hyper-V verwenden.

Wenn Sie Gedanken haben, senden Sie uns bitte Feedback über GitHub oder UserVoice.  Wir freuen uns über Ihr Feedback zu den spezifischen Erfahrungen, die Sie sehen möchten.
