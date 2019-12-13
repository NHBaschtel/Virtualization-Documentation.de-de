---
title: Windows-und Linux-Container unter Windows 10
description: Containerbereitstellung – Schnellstart
keywords: docker, Container, lkuh
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909580"
---
# <a name="linux-containers-on-windows-10"></a>Linux-Container unter Windows 10

Die Übung führt Sie durch das Erstellen und Ausführen von Linux-Containern unter Windows 10.

In diesem Schnellstart wird Folgendes erreicht:

1. Installieren von Docker Desktop
2. Ausführen eines einfachen Linux-Containers mithilfe von Linux-Containern unter Windows (lkuh)

Dieser Schnellstart bezieht sich speziell auf Windows 10. Weitere Informationen zur Schnellstart Dokumentation finden Sie im Inhaltsverzeichnis auf der linken Seite dieser Seite.

## <a name="prerequisites"></a>Voraussetzungen

Stellen Sie sicher, dass die folgenden Anforderungen erfüllt sind:
- Ein physisches Computersystem, auf dem Windows 10 Professional, Windows 10 Enterprise oder Windows Server 2019, Version 1809 oder höher, ausgeführt wird
- Stellen Sie sicher, dass [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) aktiviert ist.

***Hyper-V-Isolation:*** Für Linux-Container unter Windows ist die Hyper-V-Isolation unter Windows 10 erforderlich, damit Entwickler den entsprechenden Linux-Kernel zum Ausführen des Containers bereitstellen können. Weitere Informationen zur Hyper-V-Isolation finden Sie auf der Seite [Informationen zu Windows-Containern](../about/index.md) .

## <a name="install-docker-desktop"></a>Installieren von Docker Desktop

Laden Sie [docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) herunter, und führen Sie das Installationsprogramm aus. (Sie müssen sich anmelden. Erstellen Sie ein Konto, wenn Sie noch keines besitzen). [Ausführliche Informationen zur Installation](https://docs.docker.com/docker-for-windows/install) finden Sie in der Dokumentation zu Docker.

> Wenn Sie docker bereits installiert haben, stellen Sie sicher, dass Sie Version 18,02 oder höher zur Unterstützung von lcow haben. Überprüfen Sie, ob Sie `docker -v` ausführen oder überprüfen, ob *docker*

> Die Option "experimentelle Features" in *docker-Einstellungen > Daemon* muss aktiviert werden, um lkuh-Container auszuführen.

## <a name="run-your-first-lcow-container"></a>Ausführen Ihres ersten lkuh-Containers

In diesem Beispiel wird ein Container mit dem Namen "busybox" bereitgestellt. Versuchen Sie zunächst, das busybox-Image "Hallo Welt" auszuführen.

```console
docker run --rm busybox echo hello_world
```

Beachten Sie, dass dies einen Fehler zurückgibt, wenn docker versucht, das Bild abzurufen. Dies liegt daran, dass docker über das `--platform`-Flag eine-Direktive erfordert, um zu bestätigen, dass das Image und das Host Betriebssystem entsprechend übereinstimmen. Da die Standardplattform im Windows-Container Modus Windows ist, fügen Sie ein `--platform linux` Flag zum Abrufen und Ausführen des Containers hinzu.

```console
docker run --rm --platform linux busybox echo hello_world
```

Nachdem das Bild mit der genannten Plattform abgerufen wurde, ist das `--platform`-Flag nicht mehr erforderlich. Führen Sie den Befehl ohne den Befehl aus, um dies zu testen.

```console
docker run --rm busybox echo hello_world
```

Führen Sie `docker images` aus, um eine Liste der installierten Images zurückzugeben. In diesem Fall werden die Windows-und Linux-Images angezeigt.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Bonus: siehe den dazugehörigen [Blogbeitrag](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) von Docker zum Ausführen von lkuh.

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Erfahren Sie, wie Sie eine Beispiel-app erstellen.](./building-sample-app.md)
