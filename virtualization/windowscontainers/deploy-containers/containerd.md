---
title: Erstellen des Container-Stapels
description: Weitere Informationen zu neuen Container verfügbaren Bausteine in Windows.
keywords: LCOW, Linux-Container, Docker, Container, Containerd, Cri, Runhcs, runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 970de62c9a0011fa09d6741b2665479efd394313
ms.sourcegitcommit: 166aa2430ea47d7774392e65a9875501f86dd5ed
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/20/2018
ms.locfileid: "7460576"
---
# <a name="container-platform-tools-on-windows"></a>Container-Plattform-Tools unter Windows

Die Windows-Container-Plattform wird erweitert!  Docker war der erste Teil der Reise Container, nachdem wir andere Container-Plattform-Tools erstellen.

1. [Containerd/Cri](https://github.com/containerd/cri) - neu in Windows Server 2019/Windows 10 1809.
1. [Runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) - Windows-Container-Host Gegenstück zu Runc.
1. [Hcs](https://docs.microsoft.com/virtualization/api/) - der Host Compute Service + praktische Shims, die leichter zu verwenden ist.

    * [hcsshim](https://github.com/microsoft/hcsshim)
    * [Dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

In diesem Artikel wird die Plattform für Windows und Linux-Container als auch für jede Container-Plattform-Tool sprechen.

## <a name="windows-and-linux-container-platform"></a>Windows und Linux-Container-Plattform

In Umgebungen, Linux sind Container-Management-Tools wie Docker auf eine genauere Reihe von Container-Tools - [Runc](https://github.com/opencontainers/runc) und [Containerd](https://containerd.io/)integriert.

![Docker-Architektur unter Linux](media/docker-on-linux.png)

`runc` ist ein Linux-Befehlszeile-Tool zum Erstellen und Ausführen von Containern gemäß der [OCI Container-Runtime-Spezifikation](https://github.com/opencontainers/runtime-spec).

`containerd` ist ein Daemon, die Container-Lebenszyklus herunterzuladen und zu entpacken das Container-Image über Container-Ausführung und Überwachung verwaltet.

Unter Windows haben wir einen anderen Ansatz.  Wenn wir arbeiten mit Docker für Windows-Container unterstützen gestartet, integriert es direkt auf die HCS (Host Compute Service).  [In diesem Blogbeitrag](https://blogs.technet.microsoft.com/virtualization/2017/01/27/introducing-the-host-compute-service-hcs/) ist mit Informationen über warum wir die HCS integriert und warum wir diesen Ansatz mit Containern anfänglich dauerte voll.

![Anfängliche Docker-Modul-Architektur unter Windows](media/hcs.png)

Zu diesem Zeitpunkt ruft Docker weiterhin direkt in die HCS. Zukunft, jedoch Container-Management-Tools um zu enthalten Windows-Container und die Windows-Container-Host aufrufen, das in Containerd und Runhcs die Möglichkeit, die sie für Containerd und Runc unter Linux aufrufen.

## <a name="runhcs"></a>runhcs

`runhcs` ist eine Verzweigung von `runc`.  Wie `runc`, `runhcs` ein Befehlszeilen-Client für die Ausführung von Anwendungen gemäß der Open Container Initiative (OCI)-Format und eine kompatible Implementierung der Open Container Initiative-Spezifikation.

Funktionale Unterschiede zwischen Runc und Runhcs umfassen:

* `runhcs` unter Windows ausführen.  Er kommuniziert mit dem [HCS](containerd.md#hcs) zum Erstellen und Verwalten von Containern.
* `runhcs` eine Vielzahl von verschiedene Containertypen kann ausgeführt werden.

  * Windows und Linux- [Hyper-V-Container](../manage-containers/hyperv-container.md)
  * Windows verarbeitet Container (Container-Image muss den Container-Host entsprechen)

**Syntax:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` ist Ihr Name für die Container-Instanz, die Sie beginnen. Der Name muss auf dem Hostcontainer eindeutig sein.

Das bündelverzeichnis (mit `-b bundle`) ist optional.  
Wie bei Runc, werden die Container mit Bündel konfiguriert. Ein Container-Bündel ist das Verzeichnis mit der Container OCI-Spezifikationsdatei "config.json".  Der Standardwert für "Bundle" ist das aktuelle Verzeichnis.

Die Spezifikationen OCI-Datei, die "config.json", verfügt über zwei Felder für die ordnungsgemäße Ausführung haben:

1. Einen Pfad zu der Container-Speicherbereich
1. Ein Pfad zu der Container-Ebene Verzeichnis

Container-Befehle in Runhcs verfügbar sind:

* Tools zum Erstellen und Ausführen eines Containers
  * **Führen Sie** erstellt und führt einen container
  * **Erstellen Sie** erstellen einen container

* Tools zum Verwalten von Prozessen, die in einem Container ausgeführt:
  * **Starten** , wird der benutzerdefinierte Prozess in einem erstellten Container ausgeführt
  * **EXEC** wird einen neuen Prozess innerhalb des Containers ausgeführt.
  * Pause **unterbrechen** hält alle Prozesse im container
  * **Fortsetzen** setzt alle Prozesse, die zuvor angehalten wurde
  * **PS** Ps zeigt die in einem Container ausgeführten Prozesse

* Tools zum Verwalten eines Containers Zustand
  * **Zustand** ausgibt, den Zustand eines Containers
  * das angegebene Signal sendet **die Beendigung** (Standard: SIGTERM liegen) an den Container Initialisierung Prozess
  * **Löschen** löscht alle Ressourcen frei, die für den Container, die häufig mit getrennte Container verwendet

Der einzige Befehl, der mit mehreren Containern angesehen werden konnten ist **Liste**.  Es enthält ausgeführt wird (oder angehalten) Container durch Runhcs mit dem angegebenen Stamm gestartet.

### <a name="hcs"></a>HCS

Wir haben zwei Wrapper verfügbar auf GitHub, mit der HCS-Schnittstelle. Da die HCS eine C-API ist, erleichtern Wrapper aufrufen, die HCS aus höherer Ebene Sprachen.  

* [Hcsshim](https://github.com/microsoft/hcsshim) - HCSShim ist in Go geschrieben, und es ist die Grundlage für Runhcs.
Beziehen Sie die aktuelle Version aus AppVeyor, oder erstellen Sie es selbst.
* [Dotnet-Computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -Dotnet-Computevirtualization ist ein c#-Wrapper für die HCS.

Wenn Sie die HCS (entweder direkt oder über einen Wrapper) verwenden möchten, oder Sie einen Rost/Haskell/InsertYourLanguage Wrapper um die HCS machen möchten, Schreiben Sie einen Kommentar.

Eine genauere Betrachtung der HCS Überwachen [John Starks DockerCon Präsentation](https://www.youtube.com/watch?v=85nCF5S8Qok).

## <a name="containerdcri"></a>Containerd-cri

> ! Hinweis: CRI-Unterstützung ist nur in Server 2019/Windows 10 1809 und höher verfügbar.

Während OCI Spezifikationen definiert einen einzelnen Container, beschreibt [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (Container-Runtime-Schnittstelle) Container als Workload(s) in einem freigegebenen Sandkasten Umgebung einen Pod aufgerufen.  Pods können einen oder mehrere containerarbeitslasten enthalten.  Pods können Container-orchestratoren an, wie Kubernetes und Service Fabric Gitter gruppierten Workloads behandeln, die auf dem gleichen Host mit einigen freigegebenen Ressourcen wie Speicher und vNETs werden sollen.

Links zu den CRI-Spezifikation:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) - Pod-Spezifikation
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) - Workload-Spezifikation

![Containerd-basierte Container Umgebungen](media/containerd-platform.png)

Obwohl RunHCS und Containerd Systeme, auf denen Windows Server 2016 oder höher verwaltet werden können, erforderlich Pods (Gruppen von Containern) unterstützen, grundlegend geändert Container-Tools in Windows.  CRI-Unterstützung ist in Windows Server 2019/Windows 10 1809 verfügbar und höher.