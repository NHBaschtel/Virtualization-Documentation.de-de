---
title: Linux-Container unter Windows
description: Lernen Sie Möglichkeiten, die Sie Hyper-V Linux-Container unter Windows ausführen, als befänden sie systemeigene verwenden können.
keywords: LCOW, Linux-Container, Docker, Container
author: scooley
ms.date: 11/02/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 7db0135e5d5079d3b8cce815d051ecd6a7cb896b
ms.sourcegitcommit: 614e3ca3e6f4373b999a501a2829adbaa61de4c4
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/19/2018
ms.locfileid: "7277534"
---
# <a name="linux-containers-on-windows"></a>Linux-Container unter Windows

Linux-Container bis großen Prozentsatz der allgemeinen containerökosystems und sind sowohl für Entwickler Umgebungen produktionsumgebungen von grundlegender Bedeutung.  Da Container einen Kernel mit dem containerhost Teilen, jedoch ausgeführten Linux-Containern direkt auf Windows keine Option[*](linux-containers.md#other-options-we-considered).  Dies ist, in denen Virtualisierung ins Spiel kommt.

Es gibt zwei Möglichkeiten für die Ausführung von Linux-Containern mit Docker für Windows und Hyper-v: momentan

1. Ausführen von Linux-Container in einer vollständigen Linux-VM - ist dies Docker in der Regel heute Funktionsweise.
1. Ausführen von Linux-Container mit [Hyper-V-Isolierung](../manage-containers/hyperv-container.md) (LCOW) - ist dies eine neue Option in Docker für Windows.

In diesem Artikel wird beschrieben, wie jeder Ansatz funktioniert, Hinweise dazu, wann Sie auswählen, welche Lösung und teilt sich In Bearbeitung.

## <a name="linux-containers-in-a-moby-vm"></a>In einer VM Moby Linux-Container

Zum Ausführen von Linux-Container in einem Linux-VM, folgen Sie den Anweisungen im [Docker Get - Schritte](https://docs.docker.com/docker-for-windows/).

Docker hat Linux-Container unter Windows ausführen konnten desktop, da es der ersten Veröffentlichung in 2016 (vor dem Hyper-V-Isolierung oder LCOW waren verfügbar) verwenden ein [LinuxKit](https://github.com/linuxkit/linuxkit) -basierten virtuellen Computer auf Hyper-V.

In diesem Modell führt Docker-Client auf Windows-Desktop jedoch Aufrufe in Docker-Daemon auf den Linux-VM.

![Moby VM wie der containerhost](media/MobyVM.png)

In diesem Modell freigeben alle Linux-Container an einem einzelnen Linux-basierten containerhost und allen Linux-Containern:

* Teilen Sie einen Kernel sich untereinander und die Moby VM, jedoch nicht mit dem Windows-Host.
* Einheitliche Speicher-networking Eigenschaften mit Linux-Container unter Linux, (da sie auf einem Linux-VM ausgeführt werden).

Dies bedeutet auch, dass der Linux-Container-Host (Moby VM) Docker-Daemon und alle Docker-Daemon Abhängigkeiten ausgeführt werden muss.

Um festzustellen, ob Sie mit Moby VM ausführen, überprüfen Sie Hyper-V-Manager für Moby VM, mit der Hyper-V-Manager-Benutzeroberfläche oder durch Ausführen von `Get-VM` in ein PowerShell-Fenster mit erhöhten Rechten.

## <a name="linux-containers-with-hyper-v-isolation"></a>Linux-Container mit Hyper-V-Isolierung

Um LCOW zu testen, führen Sie die Linux-Container-Anweisungen in [diesem Handbuch Get started](../quick-start/quick-start-windows-10.md)

Linux-Container mit Hyper-V-Isolierung Ausführen jeder Linux-Container (LCOW) in einer optimierten Linux VM mit ausreichendem Betriebssystem-Container ausgeführt.  Im Gegensatz zu der Moby VM-Ansatz hat jede LCOW einen eigenen Kernel und eigene VM-Sandbox.  Sie können auch direkt von Docker unter Windows verwaltet.

![Linux-Container mit Hyper-V-Isolierung (LCOW)](media/lcow-approach.png)

Schalten genauer ansehen, containerverwaltung zwischen dem Moby VM-Ansatz und LCOW unterscheidet, in der LCOW Modell Container Management bleibt auf Windows und jede LCOW Verwaltung erfolgt über GRPC und Containerd.  Das bedeutet, dass, mit denen die Linux-Distribution-Container für LCOW kann eine deutlich kleiner Bestand haben.  Rechts sind jetzt LinuxKit verwenden wir für den optimierten Distribution-Container verwenden, jedoch anderen Projekten wie Kata sind ähnlich wie hoch optimiert Linux-Distributionen (unverschlüsselte Linux) sowie erstellen.

Hier sehen Sie jede LCOW näher betrachten:

![LCOW-Architektur](media/lcow.png)

Um festzustellen, ob Sie LCOW ausführen, navigieren Sie zu `C:\Program Files\Linux Containers`.  Wenn Docker für die Verwendung von LCOW konfiguriert ist, werden es einige Dateien, die hier, enthält die minimale LinuxKit-Distribution, die in jedem Hyper-V-Container ausgeführt wird.  Beachten Sie, dass die optimierten VM-Komponenten weniger als 100 MB, sehr viel kleiner als das Bild LinuxKit Moby VM sind.

### <a name="work-in-progress"></a>In Bearbeitung

LCOW wird weiterentwickelt.  Nachverfolgen von Fortschritte beim Moby-Projekt auf [GitHub](https://github.com/moby/moby/issues/33850)

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

Diese Anwendungen erfordern alle Volume-Zuordnung und nicht startet oder ordnungsgemäß ausgeführt.

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>Zusätzliche Informationen

[Beschreiben LCOW Docker-blog](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux-Container-Video](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-Kernel plus erstellungsanweisungen](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Wann Moby VM Vs LCOW verwendet wird.

### <a name="when-to-use-moby-vm"></a>Wann Moby VM verwendet werden.

Rechts empfehlen wir nun die Moby VM-Methode des Linux-Container an eine Person, die:

1. Möchten Sie eine stabile containerumgebung.  Dies ist die Standardeinstellung Docker für Windows.
1. Führen Sie Windows oder Linux-Container, jedoch nur selten beide gleichzeitig.
1. Kompliziert haben oder benutzerdefinierte networking Anforderungen zwischen Linux-Container.
1. Benötigen Sie keine Kernel Isolation (Hyper-V-Isolation) zwischen Linux-Container.

### <a name="when-to-use-lcow"></a>Wann LCOW verwendet werden.

Rechts empfehlen wir nun LCOW an eine Person, die:

1. Unsere neueste Technologie testen möchten.
1. Windows und Linux-Container zur gleichen Zeit ausführen.
1. Benötigen Sie Kernel Isolation (Hyper-V-Isolation) zwischen Linux-Container.

## <a name="other-options-we-considered"></a>Andere Optionen, die wir als betrachtet.

Wenn wir Verfahren zum Ausführen von Linux-Container unter Windows angezeigt wurden, als wir WSL betrachtet. Schließlich haben wir uns einen Ansatz virtualisierungsbasierte, sodass Linux-Container unter Windows mit Linux-Container unter Linux konsistent sind. Mit Hyper-V wird LCOW auch sicherer. Wir möglicherweise erneut in der Zukunft auswerten, aber für den Moment LCOW weiterhin Hyper-V verwenden.

Wenn Sie Gedanken haben, senden Sie Feedback über GitHub oder UserVoice.  Vielen Dank für besonders auf Feedback über die spezifischen Erfahrung, die Sie anzeigen möchten.
