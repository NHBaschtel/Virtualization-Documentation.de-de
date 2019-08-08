---
title: Windows-Container Plattform
description: Erfahren Sie mehr über neue Container-Bausteine, die in Windows verfügbar sind.
keywords: LCOW, Linux-Container, Docker, Container, Container, CRI, runhcs, Runc
author: scooley
ms.date: 11/19/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: a0e62b32-0c4c-4dd4-9956-8056e9abd9e5
ms.openlocfilehash: 3107eb48dc9c75224b0c9dd9b436af6f0f451871
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998417"
---
# <a name="container-platform-tools-on-windows"></a>Tools für die Container Plattform unter Windows

Die Windows-Container Plattform wird erweitert! Docker war das erste Stück der Container Reise, jetzt bauen wir weitere Tools für die Container Plattform auf.

* [containerd/CRI](https://github.com/containerd/cri) -neu in Windows Server 2019/Windows 10 1809.
* [runhcs](https://github.com/Microsoft/hcsshim/tree/master/cmd/runhcs) – ein Windows-Container Host-Pendant zu Runc.
* [HCS](https://docs.microsoft.com/virtualization/api/) – der Host Compute-Service + handliche Shims, um die Nutzung zu vereinfachen.
  * [hcsshim](https://github.com/microsoft/hcsshim)
  * [DotNet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization)

In diesem Artikel wird die Windows-und Linux-Container Plattform sowie die einzelnen Container plattformtools erläutert.

## <a name="windows-and-linux-container-platform"></a>Windows-und Linux-Container Plattform

In Linux-Umgebungen basieren Container Verwaltungstools wie docker auf einem differenzierteren Satz von containertools: [Runc](https://github.com/opencontainers/runc) und [containerd](https://containerd.io/).

![Docker-Architektur unter Linux](media/docker-on-linux.png)

`runc` ist ein Linux-Befehlszeilentool zum Erstellen und Ausführen von Containern gemäß der [Spezifikation der OCI-Container Laufzeit](https://github.com/opencontainers/runtime-spec).

`containerd` ist ein Daemon, der den Lebenszyklus von Containern durch herunterladen und Auspacken des Container Bilds in die Container Ausführung und-Überwachung verwaltet.

Unter Windows haben wir einen anderen Ansatz gewählt.  Als wir mit der Verwendung von Docker zur Unterstützung von Windows-Containern begannen, haben wir direkt auf dem HCS (Host Compute Service) gebaut.  [Dieser Blogbeitrag](https://techcommunity.microsoft.com/t5/Containers/Introducing-the-Host-Compute-Service-HCS/ba-p/382332) enthält viele Informationen dazu, warum wir den HCS aufgebaut haben und warum wir diesen Ansatz zunächst für Container entwickelt haben.

![Anfängliche Andock Modul Architektur unter Windows](media/hcs.png)

An diesem Punkt ruft docker weiterhin direkt in den HCS an. In diesem Fall können die Container Verwaltungstools, die auf Windows-Container und den Windows-Container Host ausgeweitet werden, in containerd und runhcs so aufgerufen werden, wie Sie auf containerd und Runc unter Linux aufgerufen werden.

## <a name="runhcs"></a>runhcs

`runhcs` ist ein Fork von `runc`.  Wie `runc` `runhcs` ist ein Befehlszeilenclient zum Ausführen von Anwendungen, die gemäß dem OCI-Format (Open Container Initiative) verpackt sind und eine konforme Implementierung der Open Container-initiativ Spezifikation sind.

Zu den funktionellen unterschieden zwischen Runc und runhcs gehören:

* `runhcs` wird unter Windows ausgeführt.  Sie kommuniziert mit dem [HCS](containerd.md#hcs) , um Container zu erstellen und zu verwalten.
* `runhcs` kann eine Vielzahl unterschiedlicher Containertypen ausführen.

  * Windows-und Linux [-Hyper-V-Isolierung](../manage-containers/hyperv-container.md)
  * Windows-Prozesscontainer (Container Bild muss mit dem Container Host übereinstimmen)

**Syntax:**

``` cmd
runhcs run [ -b bundle ] <container-id>
```

`<container-id>` ist Ihr Name für die Containerinstanz, die Sie starten. Der Name muss auf dem Container Host eindeutig sein.

Das Bundle-Verzeichnis ( `-b bundle`using) ist optional.  
Wie bei Runc werden Container mithilfe von Bundles konfiguriert. Das Bundle eines Containers ist das Verzeichnis mit der OCI-Spezifikationsdatei "config. JSON" des Containers.  Der Standardwert für "Bundle" ist das aktuelle Verzeichnis.

Die OCI-Spezifikationsdatei "config. JSON" muss zwei Felder aufweisen, die ordnungsgemäß ausgeführt werden können:

* Ein Pfad zum Scratch-Bereich des Containers
* Ein Pfad zum layerverzeichnis des Containers

Zu den in runhcs verfügbaren Container Befehlen gehören:

* Tools zum Erstellen und Ausführen eines Containers
  * **Ausführen** erstellt und führt einen Container aus
  * **Erstellen** eines Containers

* Tools zum Verwalten von Prozessen, die in einem Container ausgeführt werden:
  * **Start** führt den benutzerdefinierten Prozess in einem erstellten Container aus.
  * **exec** führt einen neuen Prozess innerhalb des Containers aus.
  * **** Pause anhalten unterbricht alle Prozesse innerhalb des Containers
  * **Resume** setzt alle Prozesse fort, die zuvor angehalten wurden.
  * **PS** PS zeigt die Prozesse an, die in einem Container ausgeführt werden.

* Tools zum Verwalten des Zustands eines Containers
  * **Status** gibt den Zustand eines Containers aus
  * **Kill** sendet das angegebene Signal (Standard: SIGTERM) an den init-Prozess des Containers.
  * **Delete** löscht alle Ressourcen, die für den Container reserviert sind, der häufig mit einem getrennten Container verwendet wird.

Der einzige Befehl, der als Multi-Container angesehen werden kann, ist **Liste**.  Es werden die ausgeführten oder angehenden Container angezeigt, die von runhcs mit dem angegebenen Stamm begonnen wurden.

### <a name="hcs"></a>HCS

Wir verfügen über zwei Wrapper, die auf GitHub zur Schnittstelle mit dem HCS verfügbar sind. Da es sich bei HCS um eine C-API handelt, ist es einfach, den HCS aus Sprachen höherer Ebene anzurufen.  

* [hcsshim](https://github.com/microsoft/hcsshim) -hcsshim ist in go geschrieben und ist die Basis für runhcs.
Holen Sie sich die neueste Version von AppVeyor, oder erstellen Sie Sie selbst.
* [dotnet-computevirtualization](https://github.com/microsoft/dotnet-computevirtualization) -dotnet-computevirtualization ist ein C#-Wrapper für den HCS.

Wenn Sie den HCS verwenden möchten (entweder direkt oder über einen Wrapper) oder wenn Sie einen Rost/haskell/InsertYourLanguage-Wrapper rund um den HCS erstellen möchten, geben Sie bitte einen Kommentar ein.

Eine genauere Betrachtung des HCS finden Sie in der [DockerCon-Präsentation von John stark](https://www.youtube.com/watch?v=85nCF5S8Qok).

## <a name="containerdcri"></a>Container/CRI

> [!IMPORTANT]
> Die CRI-Unterstützung steht nur in Server 2019/Windows 10 1809 und höher zur Verfügung.  Wir entwickeln auch weiterhin aktiv Container für Windows.
> Nur dev/Test.

Während die OCI-Spezifikationen einen einzelnen Container definieren, beschreibt [CRI](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto) (Container Runtime Interface) Container als Arbeitsauslastung in einer freigegebenen Sandbox-Umgebung, die als Pod bezeichnet wird.  Pods können mindestens eine Containerauslastung enthalten.  Mit Pods können Container-Orchestrators wie Kubernetes und Service Fabric-Mesh gruppierte Arbeitsauslastungen behandeln, die sich auf demselben Host befinden sollten, mit einigen freigegebenen Ressourcen wie Speicher und vNETs.

containerd/CRI aktiviert die folgende Kompatibilitätsmatrix für Pods:

| Host Betriebssystem | Container-Betriebssystem | Isolation | Pod-Unterstützung? |
|:-------------------------------------------------------------------------|:-----------------------------------------------------------------------------|:---------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|
| <ul><li>Windows Server 2019/1809</ul></li><ul><li>Windows 10 1809</ul></li> | Linux | `hyperv` | Ja – unterstützt echte Multi-Container-Pods. |
|  | Windows Server 2019/1809 | `process`* oder `hyperv` | Ja – unterstützt true Multi-Container-Pods, wenn jedes Arbeits Auslastungs Container-Betriebssystem dem Dienstprogramm VM-Betriebssystem entspricht. |
|  | Windows Server 2016,</br>Windows Server 1709,</br>Windows Server 1803 | `hyperv` | Teilweise – unterstützt Pod-Sandboxen, die einen einzelnen Prozess isolierten Container pro Dienstprogramm-VM unterstützen können, wenn das Container Betriebssystem dem Dienstprogramm VM-Betriebssystem entspricht. |

\ * Windows 10-Hosts unterstützen nur die Hyper-V-Isolierung

Links zur CRI-Spezifikation:

* [RunPodSandbox](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L24) -Pod-Spezifikation
* [CreateContainer](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/apis/cri/runtime/v1alpha2/api.proto#L47) – Arbeits Auslastungs Spezifikation

![Container basierte Container Umgebungen](media/containerd-platform.png)

Während runHCS und containerd beide auf einem beliebigen Windows-System Server 2016 oder höher verwalten können, mussten unterstützende Pods (Container Gruppen) wichtige Änderungen an den containertools in Windows erforderlich sein.  Die CRI-Unterstützung steht unter Windows Server 2019/Windows 10 1809 und höher zur Verfügung.
