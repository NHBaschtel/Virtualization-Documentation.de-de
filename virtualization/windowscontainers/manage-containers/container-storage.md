---
title: Übersicht über Container Speicher
description: So können Windows Server-Container Host- und andere Speichertypen verwenden
keywords: Container, Volume, Speicher, Mount, Binden von Bereitstellungen
author: cwilhit
ms.openlocfilehash: fba08de884d59cc1b656895ec2b7078ba3975269
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910270"
---
# <a name="container-storage-overview"></a>Übersicht über Container Speicher

<!-- Great diagram would be great! -->

Dieses Thema bietet einen Überblick über die verschiedenen Möglichkeiten, wie Container unter Windows Speicher verwenden. Container Verhalten sich im Hinblick auf den Speicher anders als virtuelle Computer. Naturgemäß werden Container erstellt, um zu verhindern, dass eine in Ihnen ausgelaufende App den Zustand über das Dateisystem des Hosts schreibt. Container verwenden standardmäßig einen "temporären" Speicherplatz, aber Windows bietet auch eine Möglichkeit, den Speicher beizubehalten.

## <a name="scratch-space"></a>Sicherer Speicherbereich

Windows-Container verwenden standardmäßig kurzlebigen Speicher. Alle Container-e/a-Vorgänge erfolgen in einem "temporären Speicherplatz", und jeder Container erhält einen eigenen Grund. Dateierstellung und Datei Schreibvorgänge werden im temporären Speicherplatz aufgezeichnet und nicht mit dem Host versehen. Wenn eine Container Instanz beendet wird, werden alle Änderungen, die im temporären Bereich aufgetreten sind, entfernt. Wenn eine neue Container Instanz gestartet wird, wird ein neuer temporärer Speicherplatz für die Instanz bereitgestellt.

## <a name="layer-storage"></a>Schichtspeicher

Wie in der [Übersicht über Container](../about/index.md)beschrieben, handelt es sich bei Container Images um ein Bündel von Dateien, die als eine Reihe von Ebenen ausgedrückt werden. Ebenenspeicher sind alle Dateien, die in den Container integriert sind. Bei jedem `docker pull` und `docker run` des Containers sind diese identisch.

### <a name="where-layers-are-stored-and-how-to-change-it"></a>Wo Schichten gespeichert werden und wie Sie diese ändern

Bei einer Standardinstallation werden die Schichten unter `C:\ProgramData\docker` gespeichert und auf die Verzeichnisse "Image" und "Windowsfilter" verteilt. Sie können den Speicherort der Schichten mithilfe der `docker-root`-Konfiguration ändern, wie in der Dokumentation [Docker-Modul unter Windows](../manage-docker/configure-docker-daemon.md) erläutert.

> [!NOTE]
> Für die Schichtspeicher wird nur NTFS unterstützt. ReFS wird nicht unterstützt.

Sie sollten keine Dateien der Schichtverzeichnisse ändern – diese werden sorgfältig verwaltet mithilfe von Befehlen wie:

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [Docker Abruf](https://docs.docker.com/engine/reference/commandline/pull/)
- [Docker laden](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>Unterstützte Vorgänge im Schichtspeicher

Beim Ausführen von Containern können die meisten NTFS-Vorgänge, mit Ausnahme der Transaktionen, verwendet werden. Dies beinhaltet das Festlegen von ACLs, wobei alle ACLs innerhalb des Containers geprüft werden. Wenn Prozesse mit mehreren Benutzern in einem Container ausgeführt werden sollen, können Sie Benutzer in Ihrer `Dockerfile`mit `RUN net user /create ...` erstellen, Dateizugriffssteuerungslisten festlegen und dann Vorgänge für diesen Benutzer mithilfe der [Dockerfile-USER-Direktive](https://docs.docker.com/engine/reference/builder/#user) konfigurieren.

## <a name="persistent-storage"></a>Persistent Storage

Windows-Container unterstützen Mechanismen zum Bereitstellen von persistentem Speicher über Bindungs Aufstellungen und Volumes. Weitere Informationen finden Sie unter [persistente Speicherung in Containern](./persistent-storage.md).

## <a name="storage-limits"></a>Speicher Limits

Ein gängiges Muster für Windows-Anwendungen ist das Abfragen des Speicherplatzes vor der Installation oder vor dem Erstellen neuer Dateien oder als Auslöser für das Bereinigen temporärer Dateien.  Mit dem Ziel, die Anwendungs Kompatibilität zu maximieren, stellt das Laufwerk "C:" in einem Windows-Container eine virtuelle, kostenlose Größe von 20 GB dar.

Einige Benutzer können diese Standardeinstellung außer Kraft setzen und den freien Speicherplatz auf einen kleineren oder größeren Wert konfigurieren. Dies kann mit der Option "size" in der Konfiguration "Storage-Opt" erreicht werden.

### <a name="examples"></a>Beispiele

Befehlszeile: `docker run --storage-opt "size=50GB" mcr.microsoft.com/windows/servercore:ltsc2019 cmd`

Oder Sie können die Docker-Konfigurationsdatei direkt ändern:

```Docker Configuration File
"storage-opts": [
    "size=50GB"
  ]
```

> [!TIP]
> Diese Methode funktioniert auch für docker Build. Weitere Informationen zum Ändern der Docker-Konfigurationsdatei finden Sie im Dokument [Konfigurieren von Docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file).
