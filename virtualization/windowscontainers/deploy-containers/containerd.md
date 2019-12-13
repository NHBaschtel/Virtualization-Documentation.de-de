---
title: Windows-Container Plattform
description: Weitere Informationen zu neuen Container Bausteinen, die in Windows verfügbar sind.
keywords: Lkuh, Linux-Container, Docker, Container, containerd, CRI, runhcs, Runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 74e22702aa4be30055b3f4f48c7fac926d793095
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909920"
---
# <a name="container-platform-tools-on-windows"></a>Container Platt Form Tools unter Windows

Die Windows-Container Plattform wird erweitert! Docker war der erste Teil der Container Journey, jetzt werden andere Container Platt Form Tools aufgebaut.

* [containerd/CRI](https://github.com/containerd/cri) : neu in Windows Server 2019/Windows 10 1809.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) : ein Windows-Container Host gegen Runc.
* [HCS](https://docs.microsoft.com/virtualization/api/) : der hostcomputedienst + praktische Shims, um die Verwendung zu vereinfachen.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [DotNet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

In diesem Artikel wird die Windows-und Linux-Container Plattform sowie die einzelnen Container Platt Form Tools erläutert.

## <a name="windows-and-linux-container-platform"></a>Windows-und Linux-Container Plattform

In Linux-Umgebungen basieren Container Verwaltungs Tools wie docker auf einem präziseteren Satz von Container Tools: [Runc](https://github.com/opencontainers/runc) und [containerd](https://containerd.io/).

![Docker-Architektur unter Linux](media/docker-on-linux.png)

`runc` ist ein Linux-Befehlszeilen Tool zum Erstellen und Ausführen von Containern gemäß der [OCI-Container-Lauf Zeit Spezifikation](https://github.com/opencontainers/runtime-spec).

`containerd` ist ein Daemon, der den Lebenszyklus der Container von der herunterladen und Entpacken des Container Images in die Container Ausführung und-Überwachung verwaltet.

Unter Windows haben wir einen anderen Ansatz gewählt.  Als wir mit der Arbeit mit docker zur Unterstützung von Windows-Containern begonnen haben, haben wir direkt auf den HCS (hostcompute-Dienst) erstellt.  In [diesem Blogbeitrag](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) finden Sie Informationen dazu, warum wir die HCS erstellt haben und warum wir diesen Ansatz zunächst für Container verwendet haben.

![Anfängliche docker-Engine-Architektur unter Windows](media/hcs.png)

An diesem Punkt ruft docker weiterhin direkt in die HCS auf. Allerdings können die Container Verwaltungs Tools, die auf Windows-Container zugeschnitten sind, und der Windows-Container Host in containerd und runhcs so aufgerufen werden, wie Sie für containerd und Runc unter Linux aufrufen.

## <a name="runhcs"></a>runhcs

`runhcs` ist eine Verzweigung `runc`.  Wie `runc`ist `runhcs` ein Befehlszeilen Client für das Ausführen von Anwendungen, die gemäß dem Open Container Initiative-Format (OCI) verpackt sind, und ist eine konforme Implementierung der Open Container Initiative-Spezifikation.

Zu den funktionalen unterschieden zwischen Runc und runhcs zählen folgende:

* `runhcs` wird unter Windows ausgeführt.  Er kommuniziert mit den [HCS](containerd.md#hcs) , um Container zu erstellen und zu verwalten.
* `runhcs` können eine Vielzahl unterschiedlicher Containertypen ausführen.

  * Windows-und Linux [-Hyper-V-Isolation](../manage-containers/hyperv-container.md)
  * Windows-Prozess Container (Container Image muss dem Container Host entsprechen)

**Verwendung:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` ist der Name für die Container Instanz, die Sie starten. Der Name muss auf dem Container Host eindeutig sein.

Das Paket Verzeichnis (mit `-b bundle`) ist optional.  
Wie bei Runc werden Container mithilfe von Paketen konfiguriert. Das Paket eines Containers ist das Verzeichnis mit der OCI-Spezifikations Datei des Containers "config. JSON".  Der Standardwert für "Bundle" ist das aktuelle Verzeichnis.

Die OCI-Spezifikations Datei "config. JSON" muss zwei Felder aufweisen, damit Sie ordnungsgemäß ausgeführt werden kann:

* Ein Pfad zum Ablage Bereich des Containers.
* Ein Pfad zum Ebenenverzeichnis des Containers.

Folgende Container Befehle sind in runhcs verfügbar:

* Tools zum Erstellen und Ausführen eines Containers
  * **Ausführen** erstellt und führt einen Container aus.
  * **Erstellen eines** Containers

* Tools zum Verwalten von Prozessen, die in einem Container ausgeführt werden:
  * **Start** führt den benutzerdefinierten Prozess in einem erstellten Container aus.
  * **exec** führt einen neuen Prozess innerhalb des Containers aus.
  * **Pause** anhalten hält alle Prozesse innerhalb des Containers an
  * fortsetzen setzt alle Prozesse fort, die zuvor angehalten wurden.
  * **PS** PS zeigt die Prozesse an, die in einem Container ausgeführt werden.

* Tools zum Verwalten des Zustands eines Containers
  * **Status** gibt den Status eines Containers aus.
  * **Kill** sendet das angegebene Signal (Standard: SIGTERM) an den init-Prozess des Containers.
  * **Delete** löscht alle Ressourcen, die vom Container verwendet werden und häufig mit einem getrennten Container verwendet werden.

Der einzige Befehl, der als mehrfach Container- **Liste**betrachtet werden kann.  Dabei werden laufende oder angehaltene Container aufgelistet, die von runhcs mit dem angegebenen Stamm gestartet wurden.

### <a name="hcs"></a>HCS

Wir haben zwei Wrapper, die auf GitHub verfügbar sind, um mit den HCS zu verbinden. Da es sich bei den HCS um eine C-API handelt, erleichtern Wrapper das Abrufen der HCS aus Sprachen höherer Ebene.  

* [hcssit](https://github.com/microsoft/hcsshim) -hcsshim ist in go geschrieben, und es ist die Grundlage für runhcs.
Holen Sie sich die neueste Version von appveyor, oder erstellen Sie Sie selbst.
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-computevirtualization ist ein C# Wrapper für die HCS.

Wenn Sie die HCS (entweder direkt oder über einen Wrapper) verwenden oder einen Rust/haskell/InsertYourLanguage-Wrapper für die HCS erstellen möchten, hinterlassen Sie einen Kommentar.

Weitere Informationen zu den HCS finden Sie in der [dockercon-Präsentation von John Hart](https://www.youtube.com/watch?v=85nCF5S8Qok).

## <a name="containerdcri"></a>containerd/CRI

> [!IMPORTANT]
> Die CRI-Unterstützung ist nur in Server 2019/Windows 10 1809 und höher verfügbar.  Wir entwickeln containerd für Windows auch noch aktiv.
> Nur dev/Test.

Während OCI-Spezifikationen einen einzelnen Container definiert, beschreibt [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (containerruntime-Schnittstelle) Container als Arbeitsauslastung in einer freigegebenen Sandkasten Umgebung, die als Pod bezeichnet wird.  Pods können mindestens eine containerworkloads enthalten.  Mithilfe von Pods können containerorchestratoren wie Kubernetes und Service Fabric Mesh gruppierte Arbeits Auslastungen verarbeiten, die sich auf demselben Host mit einigen freigegebenen Ressourcen wie z. b. Arbeitsspeicher und vnets befinden.

containerd/CRI ermöglicht die folgende Kompatibilitäts Matrix für Pods:

| Hostbetriebssystem | Containerbetriebssystem | Isolierung | Pod-Unterstützung? |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | Ja – unterstützt echte Pods mit mehreren Containern. |
|  | Windows Server 2019/1809 | `process`* oder `hyperv` | Ja – unterstützt echte Multi-Container-Pods, wenn jedes Betriebssystem Betriebssystem dem Betriebssystem-VM-Betriebssystem entspricht. |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | Partiell – unterstützt Pod-Sand Fächer, die einen einzelnen Prozess isolierten Container pro Utility-VM unterstützen, wenn das Container Betriebssystem dem Betriebssystem-VM-Betriebssystem entspricht. |

\*Windows 10-Hosts unterstützen nur die Hyper-V-Isolation

Links zur CRI-Spezifikation:

* [Runpodsandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -Pod-Spezifikation
* " [Kreatecontainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) "-Arbeits Auslastungs Spezifikation

![Containerd basierte Container Umgebungen](media/containerd-platform.png)

Obwohl runhcs und containerd beide auf jedem Windows-System Server 2016 oder höher verwaltet werden können, erforderte das unterstützen von Pods (Gruppen von Containern) wichtige Änderungen an den Container Tools in Windows.  Die CRI-Unterstützung ist unter Windows Server 2019/Windows 10 1809 und höher verfügbar.
