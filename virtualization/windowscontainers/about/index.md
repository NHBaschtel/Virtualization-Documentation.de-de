---
title: Informationen zu Windows-Containern
description: Container sind eine Technologie zum Verpacken und Ausführen von apps – einschließlich Windows-apps – in verschiedenen Umgebungen lokal und in der Cloud. In diesem Thema wird erläutert, wie Microsoft, Windows und Azure Sie beim entwickeln und Bereitstellen von apps in Containern unterstützen, einschließlich der Verwendung von andocker und Azure Kubernetes-Dienst.
keywords: Docker, Container
author: taylorb-microsoft
ms.date: 10/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: acce214cc8991f20c979b6dbe636590416841cb9
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254290"
---
# <a name="windows-and-containers"></a>Windows und Container

Container sind eine Technologie zum Verpacken und Ausführen von Windows-und Linux-Anwendungen in verschiedenen Umgebungen lokal und in der Cloud. Container bieten eine vereinfachte, isolierte Umgebung, die das entwickeln, bereitstellen und Verwalten von apps vereinfacht. Container werden schnell gestartet und beendet, sodass Sie ideal für apps sind, die sich schnell an die sich ändernde Nachfrage anpassen müssen. Die einfache Art von Containern macht Sie auch zu einem nützlichen Tool zur Erhöhung der Dichte und Nutzung Ihrer Infrastruktur.

![Grafik, die zeigt, wie Container in der Cloud oder lokal ausgeführt werden können, unterstützt monolithische Apps oder microservices, die in nahezu jeder Sprache geschrieben sind.](media/about-3-box.png)

## <a name="the-microsoft-container-ecosystem"></a>Das Microsoft-Container-Ökosystem

Microsoft bietet eine Reihe von Tools und Plattformen, die Sie beim entwickeln und Bereitstellen von apps in Containern unterstützen:

- <strong>Ausführen von Windows-basierten oder Linux-basierten Containern unter Windows 10</strong> für die Entwicklung und das Testen mithilfe des [andockbaren Desktops](https://store.docker.com/editions/community/docker-ce-desktop-windows), in dem die in Windows integrierte Containerfunktionalität verwendet wird. Sie können auch [Container nativ unter Windows Server ausführen](../quick-start/set-up-environment.md?tabs=Windows-Server).
- <strong>Entwickeln, testen, veröffentlichen und Bereitstellen von Windows-basierten Containern</strong> mithilfe der [leistungsstarken Container Unterstützung in Visual Studio](https://docs.microsoft.com/visualstudio/containers/overview) und [Visual Studio-Code](https://code.visualstudio.com/docs/azure/docker), die Unterstützung für andocker, andocker Compose, Kubernetes, Helm und andere nützliche Funktionen beinhalten Technologien.
- <strong>Veröffentlichen Sie Ihre apps als Container Bilder</strong> im öffentlichen DockerHub, damit Sie von anderen verwendet werden können, oder von einer privaten [Azure Container-Registrierung](https://azure.microsoft.com/services/container-registry/) für die eigene Entwicklung und Bereitstellung Ihrer Organisation, indem Sie direkt in Visual Studio und Visual Studio-Code pushen und ziehen. .
- <strong>Bereitstellen von Containern in einer Skala auf Azure</strong> oder anderen Clouds:

  - Ziehen Sie Ihre APP (Container Bild) aus einer Container Registrierung wie der Azure Container-Registrierung, und stellen Sie Sie dann mithilfe eines Orchestrators wie [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) (in der Vorschau für Windows-basierte Apps) oder Azure-Dienst bereit, und verwalten Sie Sie. [ Stoff](https://docs.microsoft.com/azure/service-fabric/).
  - Der Azure Kubernetes-Dienst stellt Container auf Azure Virtual Machines bereit und verwaltet Sie im Maßstab, ganz gleich, ob es sich um Dutzende Container, Hunderte oder sogar Tausende handelt. Auf den virtuellen Azure-Computern wird entweder ein angepasstes Windows Server-Abbild (wenn Sie eine Windows-basierte App bereitstellen) oder ein angepasstes Ubuntu Linux-Abbild ausgeführt (wenn Sie eine Linux-basierte App bereitstellen).
- <strong>Bereitstellen von Containern lokal</strong> mithilfe [des Azure-Stapels mit dem AKS-Modul](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) (in Preview mit Linux-Containern) oder [Azure Stack mit openshift](https://docs.microsoft.com/azure/virtual-machines/linux/openshift-azure-stack). Sie können Kubernetes auch selbst unter Windows Server einrichten (siehe [Kubernetes unter Windows](../kubernetes/getting-started-kubernetes-windows.md)), und wir arbeiten an der Unterstützung für die Ausführung von [Windows-Containern auf der RedHat openshift-Container Plattform](https://techcommunity.microsoft.com/t5/Networking-Blog/Managing-Windows-containers-with-Red-Hat-OpenShift-Container/ba-p/339821) .

## <a name="how-containers-work"></a>Funktionsweise von Containern

Ein Container ist ein isoliertes, leichtes Silo zum Ausführen einer Anwendung auf dem Hostbetriebssystem. Container bauen auf dem Kernel des Hostbetriebssystems auf (was als Vergrabenes Sanitärsystem des Betriebssystems zu sehen ist), wie in diesem Diagramm dargestellt.

![Architektonisches Diagramm, das zeigt, wie Container auf dem Kernel ausgeführt werden](media/container-diagram.svg)

Während ein Container den Kernel des Hostbetriebssystems freigibt, erhält der Container keinen uneingeschränkten Zugriff darauf. Stattdessen erhält der Container eine isolierte – und in einigen Fällen virtualisierte – Ansicht des Systems. Beispielsweise kann ein Container auf eine virtualisierte Version des Dateisystems und der Registrierung zugreifen, doch alle Änderungen wirken sich nur auf den Container aus und werden beim Beenden verworfen. Zum Speichern von Daten kann der Container persistenten Speicher wie einen [Azure-Datenträger](https://azure.microsoft.com/services/storage/disks/) oder eine Dateifreigabe (einschließlich [Azure-Dateien](https://azure.microsoft.com/services/storage/files/)) bereitstellen.

Ein Container baut auf dem Kernel auf, aber der Kernel bietet nicht alle APIs und Dienste, die eine APP ausführen muss – die meisten von Ihnen werden von Systemdateien (Bibliotheken) bereitgestellt, die über dem Kernel im Benutzermodus ausgeführt werden. Da ein Container aus der Benutzermodus-Umgebung des Hosts isoliert ist, benötigt der Container eine eigene Kopie dieser Benutzermodus-Systemdateien, die in einem sogenannten Basis Bild verpackt sind. Das Basis Bild dient als grundlegende Ebene, auf der der Container erstellt wird, und bietet ihm Betriebssystemdienste, die nicht vom Kernel bereitgestellt werden. Aber wir werden uns später noch mehr über Container Bilder unterhalten.

## <a name="containers-vs-virtual-machines"></a>Container vs. virtuelle Maschinen

Im Gegensatz zu einem Container führt ein virtueller Computer (VMS) ein vollständiges Betriebssystem – einschließlich des eigenen Kernels – aus, wie in diesem Diagramm dargestellt.

![Architekturdiagramm, das zeigt, wie VMS neben dem Hostbetriebssystem ein vollständiges Betriebssystem ausführen](media/virtual-machine-diagram.svg)

Container und virtuelle Maschinen haben jeweils ihre Verwendung – in der Tat verwenden viele Bereitstellungen von Containern virtuelle Maschinen als Hostbetriebssystem, anstatt direkt auf der Hardware zu laufen, insbesondere beim Ausführen von Containern in der Cloud.

Weitere Informationen zu den Ähnlichkeiten und unterschieden dieser komplementären Technologien finden Sie unter [Container vs. virtuelle Computer](containers-vs-vm.md).

## <a name="container-images"></a>Container Bilder

Alle Container werden aus Container Bildern erstellt. Container Bilder sind ein Bündel von Dateien, die in einem Stapel von Ebenen organisiert sind, die sich auf Ihrem lokalen Computer oder in einer Remote Container Registrierung befinden. Das Container Bild besteht aus den Betriebssystemdateien des Benutzermodus, die für die Unterstützung Ihrer APP, Ihrer APP, aller Runtimes oder Abhängigkeiten Ihrer APP sowie aller anderen Konfigurationsdateien erforderlich sind, die Ihre APP ordnungsgemäß ausführen muss.

Microsoft bietet mehrere Bilder (sogenannte Basisbilder) an, die Sie als Ausgangspunkt zum Erstellen eines eigenen Container Bilds verwenden können:

* <strong>Windows</strong> – enthält den vollständigen Satz von Windows-APIs und-Systemdiensten (minus Serverrollen).
* <strong>Windows Server Core</strong> – ein kleineres Bild, das eine Teilmenge der Windows Server-APIs enthält, nämlich das vollständige .NET Framework. Es enthält auch die meisten Serverrollen, aber leider nur wenige, nicht Faxserver.
* <strong>Nano Server</strong> – das kleinste Windows Server-Abbild mit Unterstützung für die .net Core-APIs und einige Server Rollen.
* <strong>Windows 10-Core</strong> – eine Windows-Version, die von Hardwareherstellern für kleine Internet-Geräte verwendet wird, die Arm-oder x86/x64-Prozessoren ausführen.

Wie bereits erwähnt, bestehen Container Bilder aus einer Reihe von Ebenen. Jede Ebene enthält eine Reihe von Dateien, die, wenn Sie zusammen überlagert sind, Ihr Container Bild darstellen. Aufgrund der mehrstufigen Beschaffenheit von Containern müssen Sie nicht immer auf ein basisimage Zielen, um einen Windows-Container zu erstellen. Stattdessen können Sie ein anderes Bild anvisieren, das bereits das gewünschte Framework trägt. Das .net-Team veröffentlicht beispielsweise ein [.net Core-Abbild](https://hub.docker.com/_/microsoft-dotnet-core) , das die .net Core-Laufzeit trägt. Sie erspart Benutzern das Duplizieren des Installationsprozesses von .net Core – stattdessen können Sie die Layer dieses Container Bilds wieder verwenden. Das .net Core-Abbild selbst basiert auf dem Nano-Server.

Weitere Informationen finden Sie unter [Container Basisbilder](../manage-containers/container-base-images.md).

## <a name="container-users"></a>Container Benutzer

### <a name="containers-for-developers"></a>Container für Entwickler

Container helfen Entwicklern, schnellere apps zu erstellen und zu versenden. Mit Containern können Entwickler ein Container Bild erstellen, das in Sekunden gleichmäßig in verschiedenen Umgebungen bereitgestellt wird. Container fungieren als einfache Methode zum Freigeben von Code in Teams und zum Bootstrap einer Entwicklungsumgebung, ohne dass sich dies auf das Host-Dateisystem auswirkt.

Container sind portabel und vielseitig, können apps ausführen, die in einer beliebigen Sprache geschrieben sind, und Sie sind mit jedem Computer kompatibel, auf dem Windows 10, Version 1607 oder höher oder Windows 2016 oder höher ausgeführt wird. Entwickler können einen Container lokal auf Ihrem Laptop oder Desktop erstellen und testen und dann das gleiche Container Abbild in der privaten Cloud, öffentlichen Cloud oder dem Dienstanbieter des Unternehmens bereitstellen. Die natürliche Agilität von Containern unterstützt moderne App-Entwicklungsmuster in großflächigen virtualisierten Cloud-Umgebungen.

### <a name="containers-for-it-professionals"></a>Container für IT-Experten

Mithilfe von Containern können Administratoren Infrastrukturen erstellen, die einfacher zu aktualisieren und zu warten sind und die Hardwareressourcen umfassender nutzen. IT-Experten können Container verwenden, um standardisierte Umgebungen für Ihre Entwicklungs-, QA-und Produktionsteams bereitzustellen. Durch die Verwendung von Containern können Systemadministratoren die Unterschiede bei den Betriebssysteminstallationen und der zugrunde liegenden Infrastruktur abstrahieren.

## <a name="container-orchestration"></a>Container-Orchestrierung

Bei der Einrichtung einer Container basierten Umgebung sind Orchestrators ein wichtiger Bestandteil der Infrastruktur. Zwar können Sie ein paar Container manuell mithilfe von Docker und Windows verwalten, doch Apps verwenden häufig fünf, zehn oder sogar Hunderte von Containern, an denen Orchestrators teilnehmen.

Container-Orchestrators wurden für die Verwaltung von Containern im Maßstab und in der Produktion entwickelt. Orchestrators bieten Funktionen für:

- Bereitstellen im Maßstab
- Arbeits Auslastungsplanung
- Statusüberwachung
- Fehler beim Failover eines Knotens
- Nach oben oder unten skalieren
- Networking
- Dienstermittlung
- Koordinieren von App-Upgrades
- Cluster Knoten Affinität

Es gibt viele verschiedene Orchestrators, die Sie mit Windows-Containern verwenden können. die folgenden Optionen werden von Microsoft bereitgestellt:
- [Azure Kubernetes-Dienst (AKS)](https://docs.microsoft.com/azure/aks/intro-kubernetes) – verwenden eines verwalteten Azure Kubernetes-Diensts
- [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/) – verwenden eines verwalteten Diensts
- [Azure-Stack mit dem AKS-Modul](https://docs.microsoft.com/azure-stack/user/azure-stack-kubernetes-aks-engine-overview) – Verwenden des Azure Kubernetes-Diensts lokal
- [Kubernetes unter Windows](../kubernetes/getting-started-kubernetes-windows.md) – Einrichten von Kubernetes yourself unter Windows

## <a name="try-containers-on-windows"></a>Testen von Containern unter Windows

Die ersten Schritte mit Containern unter Windows Server oder Windows 10 finden Sie unter den folgenden Themen:
> [!div class="nextstepaction"]
> [Erste Schritte: Konfigurieren der Umgebung für Container](../quick-start/set-up-environment.md)

Hilfe bei der Entscheidung, welche Azure Services für Ihr Szenario geeignet sind, finden Sie unter [Azure Container Services](https://azure.microsoft.com/product-categories/containers/) und [auswählen, welche Azure-Dienste zum Hosten der Anwendung verwendet](https://docs.microsoft.com/azure/architecture/guide/technology-choices/compute-decision-tree)werden sollen.
