---
title: Windows-Container-Plattform
description: Weitere Informationen zu neuen Container verfügbaren Bausteine in Windows.
keywords: LCOW, Linux-Container, Docker, Container, Containerd, Cri, Runhcs, runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 74e22702aa4be30055b3f4f48c7fac926d793095
ms.sourcegitcommit: 03e9203e9769997d8be3f66dc7935a3e5c0a83e1
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621608"
---
# <a name="container-platform-tools-on-windows"></a>Container-Plattform-Tools unter Windows

Die Windows-Container-Plattform wird erweitert! Docker war der erste Teil der Reise Container, nachdem wir andere Container-Plattform-Tools erstellen.

* [Containerd/Cri](https://github.com/containerd/cri) - neu in Windows Server 2019/Windows 10 1809.
* [Runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) - Windows-Container-Host Gegenstück zu Runc.
* [Hcs](https://docs.microsoft.com/virtualization/api/) - der Host Compute Service + praktische Shims, um es zu vereinfachen.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [Dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

In diesem Artikel wird die Plattform für Windows und Linux-Container als auch für jede Container-Plattform-Tool sprechen.

## <a name="windows-and-linux-container-platform"></a>Windows und Linux-Container-Plattform

In Umgebungen, Linux, basieren die Container-Management-Tools wie Docker auf eine genauere Satz von Container-Tools: [Runc](https://github.com/opencontainers/runc) und [Containerd](https://containerd.io/).

![Docker-Architektur unter Linux](media/docker-on-linux.png)

`runc` ist ein Linux-Befehlszeilentool zum Erstellen und Ausführen von Containern gemäß der [OCI Container-Runtime-Spezifikation](https://github.com/opencontainers/runtime-spec).

`containerd` ist ein Daemon, die Container-Lebenszyklus herunterzuladen und zu entpacken containerimages, Container Ausführung und Überwachung verwaltet.

Unter Windows haben wir einen anderen Ansatz.  Wenn wir arbeiten mit Docker für Windows-Container unterstützen gestartet, integriert wir direkt auf die HCS (Host Compute Service).  [In diesem Blogbeitrag](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) enthält viele Informationen warum wir die HCS integriert und warum wir diesen Ansatz mit Containern anfänglich haben.

![Anfängliche Docker-Modul-Architektur unter Windows](media/hcs.png)

Zu diesem Zeitpunkt ruft Docker weiterhin direkt in der HCS. Zukunft, jedoch Container-Management-Tools um zu enthalten Windows-Container und die Windows-Container-Host aufrufen, das in Containerd und Runhcs die Möglichkeit, die sie für Containerd und Runc unter Linux aufrufen.

## <a name="runhcs"></a>runhcs

`runhcs` ist eine Verzweigung von `runc`.  Wie `runc`, `runhcs` ist ein Befehlszeilen-Client für die Ausführung von Anwendungen, die gemäß der Open Container Initiative (OCI) Format verpackt und ist eine kompatible Implementierung der Open Container Initiative-Spezifikation.

Funktionale Unterschiede zwischen Runc und Runhcs umfassen:

* `runhcs` unter Windows ausführen.  Er kommuniziert mit der [HCS](containerd.md#hcs) zum Erstellen und Verwalten von Containern.
* `runhcs` eine Vielzahl von verschiedene Containertypen kann ausgeführt werden.

  * Windows und Linux- [Hyper-V-Isolierung](../manage-containers/hyperv-container.md)
  * Windows verarbeiten Container (Container-Image muss die Container-Host entsprechen)

**Syntax:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` ist der Name für die Container-Instanz, die Sie starten. Der Name muss auf dem Hostcontainer eindeutig sein.

Das bündelverzeichnis (mit `-b bundle`) ist optional.  
Wie bei Runc, werden die Container mit Bündel konfiguriert. Ein Container-Bündel ist das Verzeichnis mit der Container OCI-Spezifikationsdatei "config.json".  Der Standardwert für "Bundle" ist das aktuelle Verzeichnis.

Die Spezifikationen OCI-Datei, die "config.json", verfügt über zwei Felder für die ordnungsgemäße Ausführung haben:

* Einen Pfad zu der Container-Speicherbereich
* Ein Pfad zu der Container-Layer-Verzeichnis

Container-Befehle in Runhcs verfügbar sind:

* Tools zum Erstellen und Ausführen eines Containers
  * **Führen Sie** erstellt und ausgeführt wird einen container
  * **Erstellen Sie** erstellen einen container

* Tools zum Verwalten von Prozessen, die in einem Container ausgeführt:
  * **Starten** , wird der benutzerdefinierte Prozess in einem erstellten Container ausgeführt
  * **EXEC** wird einen neuen Prozess innerhalb des Containers ausgeführt.
  * **Anhalten** Pause Hält alle Prozesse im container
  * **Fortsetzen** setzt alle Prozesse, die zuvor angehalten wurde
  * **PS** Ps zeigt die in einem Container ausgeführten Prozesse

* Tools zum Verwalten eines Containers Zustand
  * **Zustand** ausgibt, den Zustand eines Containers
  * das angegebene Signal sendet **die Beendigung** (Standard: SIGTERM liegen) des Containers Initialisierung Prozess
  * **Löschen** löscht alle Ressourcen frei, die für den Container, die häufig mit getrennte Container verwendet

Der einzige Befehl, der mit mehreren Containern berücksichtigt werden konnte ist **Liste**.  Es enthält ausgeführt wird oder angehalten Container durch Runhcs mit dem angegebenen Stamm gestartet.

### <a name="hcs"></a>HCS

Wir haben zwei Wrapper verfügbar auf GitHub, mit der HCS-Schnittstelle. Da die HCS eine C-API ist, erleichtern Wrapper aufrufen, die HCS aus höherer Ebene Sprachen.  

* [Hcsshim](https://github.com/microsoft/hcsshim) - HCSShim wird in Gehe geschrieben, und es ist die Grundlage für Runhcs.
Nehmen Sie die aktuelle Version aus AppVeyor, oder erstellen Sie es selbst.
* [Dotnet-Computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -Dotnet-Computevirtualization ist ein c#-Wrapper für die HCS.

Wenn Sie die HCS (entweder direkt oder über einen Wrapper) verwenden möchten oder Sie einen Rost/Haskell/InsertYourLanguage Wrapper um die HCS machen möchten, Schreiben Sie einen Kommentar.

Eine genauere Betrachtung der HCS Überwachen [John Starks DockerCon Präsentation](https://www.youtube.com/watch?v=85nCF5S8Qok).

## <a name="containerdcri"></a>Containerd-cri

> [!IMPORTANT]
> CRI-Unterstützung ist nur in Server 2019/Windows 10 1809 und höher verfügbar.  Wir entwickeln für Windows auch weiterhin aktiv Containerd.
> Test-/nur.

Während OCI Spezifikationen definiert einen einzelnen Container, beschreibt [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (Container-Runtime-Schnittstelle) Container als Workload(s) in einem freigegebenen Sandkasten Umgebung einen Pod aufgerufen.  Pods können einen oder mehrere containerarbeitslasten enthalten.  Pods können Container-orchestratoren an, wie Kubernetes und Service Fabric Gitter gruppierte Workloads behandeln, die auf dem gleichen Host mit einigen freigegebenen Ressourcen wie Speicher und vNETs werden sollen.

Containerd-Cri ermöglicht die folgenden Kompatibilitätsmatrix Pods:

| Host-Betriebssystem | Betriebssystemversion des Containers | Isolation | Pod-Unterstützung? |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | Ja – unterstützt "true", mit mehreren Containern Pods. |
|  | Windows Server 2019/1809 | `process`* oder `hyperv` | Ja – unterstützt "true" mit mehreren Containern Pods aus, wenn jeder Container Workload Betriebssystem das Hilfsprogramm VM-Betriebssystem übereinstimmt. |
|  | WindowsServer 2016</br>WindowsServer 1709</br>WindowsServer 1803 | `hyperv` | Partielle – unterstützt pod-Sandboxes ausgeführt, die einen einzelnen Prozess-isolierten Container pro Dienstprogramm VM unterstützen können, wenn das Betriebssystem Container Dienstprogramms VM-Betriebssystem übereinstimmt. |

\*Windows 10 Hosts unterstützen nur Hyper-V-Isolierung

Links zu den CRI-Spezifikation:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) - Pod-Spezifikation
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) - Workload-Spezifikation

![Containerd-basierte Container Umgebungen](media/containerd-platform.png)

Obwohl RunHCS und Containerd Systeme, auf denen Windows Server 2016 oder höher verwaltet werden können, erforderlich Pods (Gruppen von Containern) unterstützen, grundlegend geändert Container-Tools in Windows.  CRI-Unterstützung ist verfügbar in Windows Server 2019/Windows 10 1809 und höher.
