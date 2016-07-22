---
title: Containerdatenvolumes
description: Erstellen und Verwalten von Datenvolumes mit Windows-Containern.
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: f5998534-917b-453c-b873-2953e58535b1
translationtype: Human Translation
ms.sourcegitcommit: 493b669bc47fc589486a82cfea73a0bb1e88cf79
ms.openlocfilehash: 26c010e79a4913b2e138f6d1d78f9406dbacbc6b

---

# Containerdatenvolumes

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

Beim Erstellen von Containern müssen Sie möglicherweise ein neues Datenverzeichnis erstellen oder dem Container ein vorhandenes Verzeichnis hinzufügen. Dies erreichen Sie durch Hinzufügen von Datenvolumes. Datenvolumes sind sowohl für den Container als auch für den Containerhost sichtbar, und Daten können von beiden gemeinsam genutzt werden. Datenvolumes können auch von mehreren Containern auf demselben Containerhost gemeinsam genutzt werden. Dieses Dokument beschreibt die Erstellung, Überprüfung und Entfernung von Datenvolumes.

## Datenvolumes

### Erstellen eines neuen Datenvolumes

Erstellen Sie ein neues Datenvolume mit dem Parameter `-v` des Befehls `docker run`. Neue Datenvolumes werden auf dem Host standardmäßig unter „c:\ProgramData\Docker\volumes“ gespeichert.

Dieses Beispiel erstellt ein Datenvolume mit dem Namen „new-data-volume“. Der ausgeführte Container kann unter „c:\new-data-volume“ auf dieses Datenvolume zugreifen.

```none
docker run -it -v c:\new-data-volume windowsservercore cmd
```

Weitere Informationen zum Erstellen von Volumes finden Sie auf docker.com unter [Manage data in containers](https://docs.docker.com/engine/userguide/containers/dockervolumes/#data-volumes) (Verwalten von Daten in Containern).

### Einbinden eines vorhandenen Verzeichnisses

Zusätzlich zum Erstellen eines neuen Datenvolumes können Sie ein vorhandenes Verzeichnis aus dem Host an den Container übergeben. Auch dies lässt sich mit dem Parameter `-v` des Befehls `docker run` erreichen. Alle Dateien im Hostverzeichnis sind im Container ebenfalls verfügbar. Jegliche Dateien, die vom Container im eingebundenen Volume erstellt werden, sind auf dem Host verfügbar. Ein und dasselbe Verzeichnis kann in mehrere Container eingebunden werden. In dieser Konfiguration können Daten von Containern gemeinsam genutzt werden.

In diesem Beispiel ist das Quellverzeichnis „c:\source“ als „c:\destination“ in einen Container eingebunden..

```none
docker run -it -v c:\source:c:\destination windowsservercore cmd
```

Weitere Informationen zum Einbinden von Hostverzeichnissen finden Sie auf docker.com unter [Manage data in containers](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume) (Verwalten von Daten in Containern).

### Einbinden einzelner Dateien

Eine einzelne Datei kann in einer Container eingebunden werden, indem der Dateiname explizit angegeben wird. In diesem Beispiel enthält das freigegebene Verzeichnis viele Dateien, es ist jedoch nur die Datei „config.ini“ innerhalb des Containers verfügbar. 

```none
docker run -it -v c:\container-share\config.ini windowsservercore cmd
```

Im ausgeführten Container ist nur die Datei „config.ini“ sichtbar.

```none
c:\container-share>dir
 Volume in drive C has no label.
 Volume Serial Number is 7CD5-AC14

 Directory of c:\container-share

04/04/2016  12:53 PM    <DIR>          .
04/04/2016  12:53 PM    <DIR>          ..
04/04/2016  12:53 PM    <SYMLINKD>     config.ini
               0 File(s)              0 bytes
               3 Dir(s)  21,184,208,896 bytes free
```

Weitere Informationen zum Einbinden von einzelnen Dateien finden Sie auf docker.com unter [Manage data in containers](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-directory-as-a-data-volume) (Verwalten von Daten in Containern).

### Einbinden des gesamten Laufwerks

Sie können ein gesamtes Laufwerk mithilfe eines ähnlichen Befehls wie diesem einbinden: Beachten Sie jedoch, dass der Befehl keinen umgekehrten Schrägstrich enthalten darf.

```none
docker run -it -v d: windowsservercore cmd
```

Zum aktuellen Zeitpunkt ist keine Teileinbindung des zweiten Laufwerks möglich. Beispielsweise ist Folgendes nicht möglich:

```none
docker run -it -v d:\source:d:\destination windowsservercore cmd
```

### Datenvolumecontainer

Datenvolumes können durch Verwendung des Parameters `--volumes-from` des Befehls `docker run` aus anderen ausgeführten Containern geerbt werden. Mithilfe dieser Vererbung kann ein Container explizit zum Hosten von Datenvolumes von Anwendungen mit Containern erstellt werden. 

Dieses Beispiel bindet die Datenvolumes aus dem Container „cocky_bell“ in einen neuen Container ein. Sobald der neue Container gestartet wurde, stehen die Daten in diesen Volumes den Anwendungen zur Verfügung, die im Container ausgeführt werden.  

```none
docker run -it --volumes-from cocky_bell windowsservercore cmd
```

Weitere Informationen zu Datencontainern finden Sie auf docker.com unter [Manage data in containers](https://docs.docker.com/engine/userguide/containers/dockervolumes/#mount-a-host-file-as-a-data-volume) (Verwalten von Daten in Containern).

### Überprüfen von freigegebenen Datenvolumes

Eingebundene Volumes können mithilfe des Befehls `docker inspect` angezeigt werden.

```none
docker inspect backstabbing_kowalevski
```

Dieser Befehl gibt Informationen zum Container zurück, einschließlich eines Abschnitts namens „Mounts“, der Daten über die eingebundenen Volumes enthält, z. B. das Quell- und das Zielverzeichnis.

```none
"Mounts": [
    {
        "Source": "c:\\container-share",
        "Destination": "c:\\data",
        "Mode": "",
        "RW": true,
        "Propagation": ""
}
```

Weitere Informationen zum Überprüfen von Volumes finden Sie auf docker.com unter [Manage data in containers](https://docs.docker.com/engine/userguide/containers/dockervolumes/#locating-a-volume) (Verwalten von Daten in Containern).




<!--HONumber=Jul16_HO3-->


