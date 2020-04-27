---
title: Windows- und Linux-Container unter Windows 10
description: Containerbereitstellung – Schnellstart
keywords: Docker, Container, LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/24/2020
ms.locfileid: "74909580"
---
# <a name="linux-containers-on-windows-10"></a>Linux-Container unter Windows 10

Diese Übung führt Sie schrittweise durch das Erstellen und Ausführen von Linux-Containern unter Windows 10.

In diesem Schnellstart lernen Sie Folgendes:

1. Installieren von Docker Desktop
2. Ausführen eines einfachen Linux-Containers mithilfe von Linux-Containern unter Windows (LCOW)

Dieser Schnellstart bezieht sich speziell auf Windows 10. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis.

## <a name="prerequisites"></a>Voraussetzungen

Stellen Sie sicher, dass Sie die folgenden Anforderungen erfüllen:
- Ein physisches Computersystem mit Windows 10 Professional, Windows 10 Enterprise oder Windows Server 2019, Version 1809 oder höher.
- Stellen Sie sicher, dass [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) aktiviert ist.

***Hyper-V-Isolierung:*** Für Linux-Container unter Windows ist Hyper-V-Isolierung unter Windows 10 erforderlich, damit für Entwickler der entsprechende Linux-Kernel zum Ausführen des Containers bereitgestellt wird. Weitere Informationen zur Hyper-V-Isolierung finden Sie auf der Seite [Informationen zu Windows-Containern](../about/index.md).

## <a name="install-docker-desktop"></a>Installieren von Docker Desktop

Laden Sie [Docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) herunter, und führen Sie das Installationsprogramm aus. (Sie müssen sich anmelden. Erstellen Sie ein Konto, sofern noch nicht vorhanden.) [Ausführliche Informationen zur Installation](https://docs.docker.com/docker-for-windows/install) finden Sie in der Dokumentation zu Docker.

> Wenn Sie Docker bereits installiert haben, stellen Sie sicher, dass Sie über Version 18.02 oder höher verfügen, um LCOW zu unterstützen. Überprüfen Sie dies, indem Sie `docker -v` ausführen oder *About Docker* (Info zu Docker) überprüfen.

> Die Option „experimental features“ (experimentelle Features) in *Docker Settings > Daemon* (Docker-Einstellungen > Daemon) muss aktiviert werden, um LCOW-Container ausführen zu können.

## <a name="run-your-first-lcow-container"></a>Ausführen Ihres ersten LCOW-Containers

Für dieses Beispiel wird ein BusyBox-Container bereitgestellt. Versuchen Sie zunächst, ein „Hello World“-BusyBox-Image auszuführen.

```console
docker run --rm busybox echo hello_world
```

Beachten Sie, dass ein Fehler zurückgegeben wird, wenn Docker versucht, das Image zu pullen. Dies liegt daran, dass Docker über das Flag `--platform` eine Direktive erfordert, um zu bestätigen, dass das Image und das Hostbetriebssystem zusammenpassen. Da die Standardplattform im Windows-Containermodus Windows ist, fügen Sie ein Flag `--platform linux` zum Pullen und Ausführen des Containers hinzu.

```console
docker run --rm --platform linux busybox echo hello_world
```

Nachdem das Image mit der angegebenen Plattform gepullt wurde, ist das Flag `--platform` nicht mehr erforderlich. Führen Sie den Befehl ohne das Flag aus, um dies zu testen.

```console
docker run --rm busybox echo hello_world
```

Führen Sie `docker images` aus, um eine Liste der installierten Images zurückzugeben. In diesem Fall handelt es sich sowohl um Windows- als auch um Linux-Images.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Bonus: Lesen Sie den entsprechenden Docker-[Blogbeitrag](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) zum Ausführen von LCOW.

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Erfahren Sie, wie eine Beispiel-App erstellt wird](./building-sample-app.md)
