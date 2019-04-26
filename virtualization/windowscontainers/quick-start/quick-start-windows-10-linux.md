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
ms.openlocfilehash: f9b54dbc9fc7c79bdb9b9aa106d5811401c365f3
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578631"
---
# <a name="linux-containers-on-windows-10"></a>Linux-Container unter Windows 10

> [!div class="op_single_selector"]
> - [Linux-Container unter Windows](quick-start-windows-10-linux.md)
> - [Windows-Container unter Windows](quick-start-windows-10.md)

Die Übung führt durch Erstellen und Ausführen von Linux-Container unter Windows 10.

In diesem Schnellstart führen Sie Aktionen aus:

1. Installierte Docker für Windows
2. Führen Sie einen einfachen Linux-Container mit Linux-Container für Windows (LCOW)

Dieser Schnellstart bezieht sich speziell auf Windows10. Weitere Schnellstartdokumentation finden Sie in der Tabelle der Inhalt auf der linken Seite der Seite.

## <a name="prerequisites"></a>Voraussetzungen

Stellen Sie sicher, dass Sie die folgenden Anforderungen erfüllen:
- Ein physisches Computersystem mit Windows 10 Professional oder Enterprise mit Fall Creators Update (Version 1709) oder höher
- Stellen Sie sicher, dass [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) aktiviert ist.

***Hyper-V-Isolation:*** Linux-Container unter Windows erfordert Hyper-V-Isolation auf Windows 10 damit Entwickler mit der entsprechenden Linux-Kernel Sie den Container ausführen. Informationen zu Hyper-V kann mehr Isolation auf der Seite " [Informationen zu Windows-Container](../about/index.md) " gefunden werden.

## <a name="install-docker-for-windows"></a>Installieren von Docker für Windows

Herunterladen von [Docker für Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) und führen Sie das Installationsprogramm (Sie müssen sich anmelden. Erstellen Sie ein Konto, wenn Sie nicht bereits eine haben). [Ausführliche Informationen zur Installation](https://docs.docker.com/docker-for-windows/install) finden Sie in der Dokumentation zu Docker.

> Wenn Sie bereits Docker installiert haben, stellen Sie sicher, dass Sie Version 18.02 oder höher unterstützen LCOW haben. Überprüfen, indem ausführen `docker -v` oder *Docker zu*überprüfen.

> Die Option "experimentelle Features" in *Docker-Einstellungen > Daemon* muss aktiviert werden, um LCOW Container auszuführen.

## <a name="run-your-first-lcow-container"></a>Führen Sie Ihres ersten Containers LCOW aus

In diesem Beispiel wird ein BusyBox-Container bereitgestellt werden. Zunächst versuchen Sie, ein "Hello World" BusyBox Bild auszuführen.

```console
docker run --rm busybox echo hello_world
```

Beachten Sie, dass dies ein Fehler zurückgegeben, wenn Docker versucht, um das Bild abzurufen. Dies tritt auf, da Dockers erfordert eine Richtlinie über die `--platform` Flag zu bestätigen, dass das Bild und die Host-Betriebssystem werden entsprechend verglichen. Da die Standard-Plattform in Windows-Container-Modus Windows ist, Hinzufügen einer `--platform linux` Flag per Pull abrufen und den Container auszuführen.

```console
docker run --rm --platform linux busybox echo hello_world
```

Nachdem das Bild mit der Plattform angegeben, zieht die `--platform` Flag ist nicht mehr erforderlich. Führen Sie den Befehl ohne, um dies zu testen.

```console
docker run --rm busybox echo hello_world
```

Führen Sie `docker images` um eine Liste der installierten Images zurückzugeben. In diesem Fall die Windows und Linux-Bilder.

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
> [Hier erfahren Sie, wie Sie eine Beispiel-app erstellen](./building-sample-app.md)