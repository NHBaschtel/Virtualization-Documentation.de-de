---
title: Linux-Container unter Windows
description: Erfahren Sie mehr über verschiedene Möglichkeiten, wie Sie Hyper-V zum Ausführen von Linux-Containern unter Windows verwenden können, als ob Sie System eigen sind.
keywords: Lkuh, Linux-Container, Docker, Container
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 14445f3e9d292dbdab28986e772d0c045fca1586
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910570"
---
# <a name="linux-containers-on-windows"></a>Linux-Container unter Windows

Linux-Container bilden einen großen Anteil des gesamten containerökosystems und sind grundlegend für Entwickler-und Produktionsumgebungen.  Da Container einen Kernel mit dem Container Host gemeinsam nutzen, ist das direkte Ausführen von Linux-Containern unter Windows keine Option[*](linux-containers.md#other-options-we-considered).  An dieser Stelle kommt die Virtualisierung ins Spiel.

Derzeit gibt es zwei Möglichkeiten zum Ausführen von Linux-Containern mit docker für Windows und Hyper-V:

- Ausführen von Linux-Containern in einer vollständigen Linux-VM: Dies erfolgt in der Regel von Docker.
- Ausführen von Linux-Containern mit [Hyper-V-Isolation](../manage-containers/hyperv-container.md) (lkuh): Dies ist eine neue Option in docker für Windows.

In diesem Artikel wird die Funktionsweise der einzelnen Ansätze beschrieben. Außerdem erhalten Sie Anleitungen dazu, wann welche Lösung ausgewählt und welche Aufgaben in Bearbeitung sind.

## <a name="linux-containers-in-a-moby-vm"></a>Linux-Container in einer VM mit dem virtuellen Computer

Um Linux-Container auf einer Linux-VM auszuführen, befolgen Sie die Anweisungen im [Leitfaden "Get-Started" von Docker](https://docs.docker.com/docker-for-windows/).

Docker ist in der Lage, Linux-Container auf Windows-Desktop auszuführen, seit es erstmals in 2016 (vor der Hyper-V-Isolation oder Linux-Containern unter Windows) mit einem [linuxkit](https://github.com/linuxkit/linuxkit) -basierten virtuellen Computer, der unter Hyper-v ausgeführt wird, veröffentlicht wurde.

In diesem Modell wird der Docker-Client auf Windows-Desktop ausgeführt, ruft jedoch den docker-Daemon auf dem virtuellen Linux-Computer auf.

![VM Moby als Container Host](media/MobyVM.png)

In diesem Modell nutzen alle Linux-Container einen einzelnen Linux-basierten Container Host und alle Linux-Container:

* Verwenden Sie einen Kernel gemeinsam mit dem virtuellen Computer mit dem Namen "Moby", aber nicht mit dem Windows-Host.
* Sie verfügen über konsistente Speicher-und Netzwerk Eigenschaften für Linux-Container, die unter Linux ausgeführt werden (da Sie auf einer Linux-VM ausgeführt werden).

Dies bedeutet auch, dass der Linux-Container Host ("Moby VM") den docker-Daemon und alle Abhängigkeiten von Docker-Daemon ausführen muss.

Um festzustellen, ob Sie mit dem virtuellen Computer "Moby" arbeiten, überprüfen Sie die Hyper-v-Manager-Benutzeroberfläche, oder führen Sie `Get-VM` in einem PowerShell-Fenster mit erhöhten Rechten aus.

## <a name="linux-containers-with-hyper-v-isolation"></a>Linux-Container mit Hyper-V-Isolation

Wenn Sie Linux-Container unter Windows (lkuh) testen möchten, befolgen Sie die Anweisungen für Linux-Container unter Linux-Container [unter Windows 10](../quick-start/quick-start-windows-10-linux.md).

Linux-Container mit Hyper-V-Isolation führen jeden Linux-Container in einer optimierten Linux-VM mit nur ausreichend Betriebssystem aus, um Container auszuführen. Im Gegensatz zum "Moby VM"-Ansatz hat jeder Linux-Container seinen eigenen Kernel und seinen eigenen VM-Sandkasten. Sie werden auch direkt von Docker unter Windows verwaltet.

![Linux-Container mit Hyper-V-Isolation (lkuh)](media/lcow-approach.png)

Wenn Sie sich genauer ansehen, wie sich die Container Verwaltung zwischen dem "Moby VM"-Ansatz und dem lcow unterscheidet, wird die Container Verwaltung im lkuh-Modell unter Windows ausgeführt, und jede lkuh-Verwaltung erfolgt über GrpC und containerd.  Dies bedeutet, dass die Linux-Distribution-Container, die für lkuh verwendet werden, eine viel geringere Inventur aufweisen können  Zurzeit verwenden wir das linuxkit für die optimierten Distribution-Container, aber andere Projekte wie z. b. Kata bauen ähnlich hochoptimierte Linux-Distributionen auf (Clear Linux).

Im folgenden finden Sie eine genauere Betrachtung der einzelnen lkuh:

![Lkuh-Architektur](media/lcow.png)

Navigieren Sie zu `C:\Program Files\Linux Containers`, um zu sehen, ob Sie lkuh ausführen. Wenn docker für die Verwendung von lkuh konfiguriert ist, gibt es hier einige Dateien, die die minimale linuxkit-Distribution enthalten, die in jedem Container ausgeführt wird, der unter Hyper-V-Isolation ausgeführt wird.  Beachten Sie, dass es sich bei den optimierten VM-Komponenten um weniger als 100 MB handelt, die wesentlich kleiner als das linuxkit-Image im virtuellen

### <a name="work-in-progress"></a>In Bearbeitung

Lkuh befindet sich in der aktiven Entwicklung. Verfolgen Sie den fortlaufenden Status im Projekt "Moby" auf [GitHub](https://github.com/moby/moby/issues/33850) .

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

Diese Anwendungen erfordern eine volumezuordnung und werden nicht ordnungsgemäß gestartet oder ausgeführt.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>Zusätzliche Informationen

[Docker-Blog zum Beschreiben von lkuh](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux-Container-Video](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[Linuxkit lkuh-Kernel Plus Build-Anweisungen](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Verwendung von "Moby VM vs lkuh"

### <a name="when-to-use-moby-vm"></a>Verwendung von "Moby VM"

Zurzeit wird die Methode "Moby VM" zum Ausführen von Linux-Containern für Personen empfohlen, die Folgendes ausführen:

- Sie möchten eine stabile Container Umgebung.  Dies ist der Docker für Windows Standard.
- Führen Sie Windows-oder Linux-Container aus, aber nur selten gleichzeitig.
- Sie verfügen über komplizierte oder benutzerdefinierte Netzwerk Anforderungen zwischen Linux-Containern.
- Sie benötigen keine Kernel Isolation (Hyper-V-Isolation) zwischen Linux-Containern.

### <a name="when-to-use-lcow"></a>Verwendung von lkuh

Zurzeit wird die lkuh für Personen empfohlen, die Folgendes:

- Wir möchten unsere neueste Technologie testen.
- Führen Sie Windows-und Linux-Container gleichzeitig aus.
- Sie benötigen eine Kernel Isolation (Hyper-V-Isolation) zwischen Linux-Containern.

## <a name="other-options-we-considered"></a>Andere von uns berücksichtigte Optionen

Als wir uns mit Möglichkeiten zum Ausführen von Linux-Containern unter Windows aussuchten, haben wir WSL in Erwägung gezogen. Letztendlich haben wir uns für einen virtualisierungsbasierten Ansatz entschieden, damit Linux-Container unter Windows mit Linux-Containern unter Linux konsistent sind. Durch die Verwendung von Hyper-V wird lkuh auch sicherer. Wir werden uns möglicherweise in Zukunft neu auswerten, aber vorerst verwendet lkuh weiterhin Hyper-V.

Wenn Sie Gedanken haben, senden Sie uns Feedback über GitHub oder UserVoice.  Wir freuen uns besonders über Feedback zu den spezifischen Funktionen, die Sie sehen möchten.
