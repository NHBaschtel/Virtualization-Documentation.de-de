---
title: Windows-und Linux-Container unter Windows 10
description: Containerbereitstellung – Schnellstart
keywords: docker, Container, LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a52c18f13d0d6bd2102f045827285821a187579b
ms.sourcegitcommit: 2f8fd4b2e7113fbb7c323d89f3c72df5e1a4437e
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/20/2019
ms.locfileid: "10045020"
---
# <a name="linux-containers-on-windows-10"></a>Linux-Container unter Windows 10

Die Übung führt Sie durch das Erstellen und Ausführen von Linux-Containern unter Windows 10.

In diesem Schnellstart können Sie Folgendes ausführen:

1. Installieren des docker-Desktops
2. Ausführen eines einfachen Linux-Containers mit Linux-Containern unter Windows (LCOW)

Dieser Schnellstart bezieht sich speziell auf Windows10. Weitere Schnellstart Dokumentation finden Sie im Inhaltsverzeichnis auf der linken Seite dieser Seite.

## <a name="prerequisites"></a>Voraussetzungen

Stellen Sie bitte sicher, dass Sie die folgenden Voraussetzungen erfüllen:
- Ein physisches Computersystem mit Windows 10 Professional, Windows 10 Enterprise oder Windows Server 2019, Version 1809 oder höher
- Stellen Sie sicher, dass [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) aktiviert ist.

***Hyper-V-Isolierung:*** Linux-Container unter Windows erfordern eine Hyper-V-Isolierung unter Windows 10, um Entwicklern den entsprechenden Linux-Kernel zur Verfügung zu stellen, um den Container auszuführen. Weitere Informationen zur Hyper-V-Isolierung finden Sie auf der Seite " [Informationen zu Windows-Container](../about/index.md) ".

## <a name="install-docker-desktop"></a>Installieren des docker-Desktops

Laden Sie den docker- [Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) herunter, und führen Sie das Installationsprogramm aus (Sie müssen sich anmelden. Erstellen Sie ein Konto, wenn Sie noch keines haben. [Ausführliche Informationen zur Installation](https://docs.docker.com/docker-for-windows/install) finden Sie in der Dokumentation zu Docker.

> Wenn Sie docker bereits installiert haben, stellen Sie sicher, dass Sie über Version 18,02 oder höher verfügen, um LCOW zu unterstützen. Überprüfen Sie `docker -v` , ob Sie docker ausführen oder *über*prüfen.

> Die Option "experimentelle Funktionen" in den docker- *Einstellungen #a0 Daemon* muss aktiviert sein, damit LCOW-Container ausgeführt werden können.

## <a name="run-your-first-lcow-container"></a>Ausführen des ersten LCOW-Containers

In diesem Beispiel wird ein busybox-Container bereitgestellt. Versuchen Sie zunächst, das busybox-Bild "Hello World" zu starten.

```console
docker run --rm busybox echo hello_world
```

Beachten Sie, dass dadurch ein Fehler zurückgegeben wird, wenn docker versucht, das Bild zu ziehen. Dies tritt auf, weil für Andockfenster eine `--platform` Direktive über das Flag erforderlich ist, um zu bestätigen, dass das Bild und das Hostbetriebssystem entsprechend angepasst werden. Da die Standardplattform im Windows-Container Modus Windows ist, fügen `--platform linux` Sie eine Kennzeichnung hinzu, um den Container zu ziehen und auszuführen.

```console
docker run --rm --platform linux busybox echo hello_world
```

Nachdem das Bild mit der angegebenen Plattform gezogen wurde, ist die `--platform` Kennzeichnung nicht mehr erforderlich. Führen Sie den Befehl ohne ihn aus, um dies zu testen.

```console
docker run --rm busybox echo hello_world
```

Ausführen `docker images` , um eine Liste der installierten Bilder zurückzugeben. In diesem Fall werden sowohl die Windows-als auch die Linux-Bilder verwendet.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

> [!TIP]
> Bonus: Lesen Sie den entsprechenden [Blogbeitrag](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) des andockers auf Running LCOW.

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Informationen zum Erstellen einer Beispiel-App](./building-sample-app.md)
