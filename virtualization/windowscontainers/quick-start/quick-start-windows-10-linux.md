---
title: Windows und Linux-Container unter Windows 10
description: Containerbereitstellung – Schnellstart
keywords: Docker, Container, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 036e4f80eaa6e7ce2c151d7732e670c0492bc61f
ms.sourcegitcommit: 95cec99aa8e817d3e3cb2163bd62a32d9e8f7181
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/18/2018
ms.locfileid: "8973813"
---
# <a name="linux-containers-on-windows-10"></a>Linux-Container unter Windows 10

> [!div class="op_single_selector"]
> - [Linux-Container unter Windows](quick-start-windows-10-linux.md)
> - [Windows-Container unter Windows](quick-start-windows-10.md)

Die Übung führt durch Erstellen und Ausführen von Linux-Container unter Windows 10.

In diesem Schnellstart werden Sie Folgendes ausführen:

1. Installierte Docker für Windows
2. Führen Sie einen einfachen Linux-Container mit Linux-Container für Windows (LCOW)

Dieser Schnellstart bezieht sich speziell auf Windows10. Weitere Schnellstartdokumentation finden Sie in der Tabelle der Inhalt auf der linken Seite der Seite.

## <a name="prerequisites"></a>Voraussetzungen

Stellen Sie sicher, dass Sie die folgenden Anforderungen erfüllen:
- Ein physisches Computersystem mit Windows 10 Professional oder Enterprise mit Fall Creators Update (Version 1709) oder höher
- Stellen Sie sicher, dass [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) aktiviert ist.

***Hyper-V-Isolation:*** Linux-Container unter Windows erfordert Hyper-V-Isolierung unter Windows 10 damit Entwickler mit der entsprechenden Linux-Kernel Sie den Container ausführen. Informationen zu Hyper-V kann mehr Isolation auf der Seite " [Informationen zu Windows-Container](../about/index.md) " gefunden werden.

## <a name="install-docker-for-windows"></a>Installieren von Docker für Windows

Laden Sie [Docker für Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) herunter und führen Sie das Installationsprogramm (Sie müssen sich anmelden. Erstellen Sie ein Konto, wenn Sie nicht bereits eine haben). [Ausführliche Informationen zur Installation](https://docs.docker.com/docker-for-windows/install) finden Sie in der Dokumentation zu Docker.

> Wenn Sie bereits Docker installiert haben, stellen Sie sicher, dass Sie Version 18.02 oder höher unterstützen LCOW haben. Überprüfen, indem Sie mit `docker -v` oder *Über Docker*überprüfen.

> Experimentelle Features in option *Docker-Einstellungen > Daemon* muss aktiviert sein, um LCOW Container ausgeführt.

## <a name="run-your-first-lcow-container"></a>Führen Sie Ihres ersten LCOW Containers aus

In diesem Beispiel wird ein BusyBox-Container bereitgestellt werden. Versuchen Sie zunächst, ein "Hello World" BusyBox Image ausführen.

```console
docker run --rm busybox echo hello_world
```

Beachten Sie, dass dies ein Fehler zurückgegeben, wenn Docker versucht, um das Bild abzurufen. In diesem Fall, da Dockers erfordert eine Richtlinie über die `--platform` Kennzeichen, um sicherzustellen, dass das Bild und die Host-Betriebssystem entsprechend zugeordnet sind. Da die Standard-Plattform in Windows-Container-Modus Windows ist, Hinzufügen einer `--platform linux` Flag per Pull abrufen und den Container auszuführen.

```console
docker run --rm --platform linux busybox echo hello_world
```

Nachdem das Bild mit der Plattform angegeben, zieht die `--platform` Flag ist nicht mehr erforderlich. Führen Sie den Befehl ohne, um dies zu testen.

```console
docker run --rm busybox echo hello_world
```

Führen Sie `docker images` um eine Liste der installierten Images zurück. In diesem Fall Windows und Linux-Bilder.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Bonus: Finden Sie entsprechende des Docker- [Blogbeitrag](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) LCOW ausgeführt.

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Hier erfahren Sie, wie Sie eine Beispiel-App](./building-sample-app.md)