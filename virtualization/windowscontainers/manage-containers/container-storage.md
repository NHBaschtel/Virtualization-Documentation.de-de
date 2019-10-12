---
title: Übersicht über Container Speicher
description: So können Windows Server-Container Host- und andere Speichertypen verwenden
keywords: Container, Volume, Speicher, Mount, Binden von Bereitstellungen
author: cwilhit
ms.openlocfilehash: fba08de884d59cc1b656895ec2b7078ba3975269
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209750"
---
# <a name="container-storage-overview"></a>Übersicht über Container Speicher

<!-- Great diagram would be great! -->

Dieses Thema bietet eine Übersicht über die verschiedenen Möglichkeiten, wie Container Speicher unter Windows verwenden. Container Verhalten sich anders als virtuelle Computer, wenn es um Speicher geht. Von Natur aus werden Container entwickelt, um zu verhindern, dass eine in Ihnen ausgeführte APP vom Schreib Zustand über das Dateisystem des Hosts abläuft. Container verwenden standardmäßig einen "Scratch"-Platz, aber Windows bietet auch eine Möglichkeit zum Beibehalten des Speichers.

## <a name="scratch-space"></a>Scratch-Platz

Windows-Container verwenden standardmäßig ephemerer Speicher. Alle Container-e/a-Vorgänge werden in einem "Scratch-Bereich" ausgeführt, und jeder Container erhält seinen eigenen Kratzer. Datei Erstellungs-und Dateischreibvorgänge werden im Scratch-Bereich erfasst und nicht dem Host entgehen. Wenn eine Containerinstanz beendet wird, werden alle Änderungen, die im Scratch-Bereich aufgetreten sind, verworfen. Wenn eine neue Containerinstanz gestartet wird, wird ein neuer Scratch-Bereich für die Instanz bereitgestellt.

## <a name="layer-storage"></a>Schichtspeicher

Wie in der [Container Übersicht](../about/index.md)beschrieben, handelt es sich bei Container Bildern um ein Bündel von Dateien, die in einer Reihe von Ebenen ausgedrückt werden. Schichtspeicher sind alle Dateien, die in den Container integriert sind. Bei jedem `docker pull` und `docker run` des Containers sind diese identisch.

### <a name="where-layers-are-stored-and-how-to-change-it"></a>Wo Schichten gespeichert werden und wie Sie diese ändern

Bei einer Standardinstallation werden die Schichten unter `C:\ProgramData\docker` gespeichert und auf die Verzeichnisse "Image" und "Windowsfilter" verteilt. Sie können den Speicherort der Schichten mithilfe der `docker-root`-Konfiguration ändern, wie in der Dokumentation [Docker-Modul unter Windows](../manage-docker/configure-docker-daemon.md) erläutert.

> [!NOTE]
> Für die Schichtspeicher wird nur NTFS unterstützt. ReFS wird nicht unterstützt.

Sie sollten keine Dateien der Schichtverzeichnisse ändern – diese werden sorgfältig verwaltet mithilfe von Befehlen wie:

- [Docker Bilder](https://docs.docker.com/engine/reference/commandline/images/)
- [Docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [Docker Abruf](https://docs.docker.com/engine/reference/commandline/pull/)
- [Docker laden](https://docs.docker.com/engine/reference/commandline/load/)
- [Docker speichern](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>Unterstützte Vorgänge im Schichtspeicher

Beim Ausführen von Containern können die meisten NTFS-Vorgänge, mit Ausnahme der Transaktionen, verwendet werden. Dies beinhaltet das Festlegen von ACLs, wobei alle ACLs innerhalb des Containers geprüft werden. Wenn Prozesse mit mehreren Benutzern in einem Container ausgeführt werden sollen, können Sie Benutzer in Ihrer `Dockerfile`mit `RUN net user /create ...` erstellen, Dateizugriffssteuerungslisten festlegen und dann Vorgänge für diesen Benutzer mithilfe der [Dockerfile-USER-Direktive](https://docs.docker.com/engine/reference/builder/#user) konfigurieren.

## <a name="persistent-storage"></a>Persistenter Speicher

Windows-Container unterstützen Mechanismen zum Bereitstellen von persistentem Speicher über BIND-Bereitstellung und Volumes. Weitere Informationen finden Sie unter [persistenter Speicher in Containern](./persistent-storage.md).

## <a name="storage-limits"></a>Speichergrenzwerte

Ein gängiges Muster für Windows-Anwendungen ist das Abfragen des Speicherplatzes vor der Installation oder vor dem Erstellen neuer Dateien oder als Auslöser für das Bereinigen temporärer Dateien.  Mit dem Ziel, die Anwendungskompatibilität zu maximieren, stellt das Laufwerk C: in einem Windows-Container eine virtuelle freie Größe von 20 GB dar.

Einige Benutzer möchten möglicherweise diese Standardeinstellung überschreiben und den freien Speicherplatz auf einen kleineren oder größeren Wert konfigurieren. Dies kann durch die Option "Größe" innerhalb der Konfiguration "Storage-Opt" erreicht werden.

### <a name="examples"></a>Beispiele

Befehlszeile: `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

Oder Sie können die Docker-Konfigurationsdatei direkt ändern:

```Docker Configuration File
"storage-opts": [
    "size=50GB"
  ]
```

> [!TIP]
> Diese Methode funktioniert auch für den andocker-Build. Weitere Informationen zum Ändern der Docker-Konfigurationsdatei finden Sie im Dokument [Konfigurieren von Docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file).
