---
title: Windows-Containerimages
description: Erstellen und Verwalten von Containerimages mit Windows-Containern.
keywords: Docker, Container
author: neilpeterson
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
translationtype: Human Translation
ms.sourcegitcommit: 7b5cf299109a967b7e6aac839476d95c625479cd
ms.openlocfilehash: 8b9ec6370d1f9f9187fbb6d74168e9e88391b657

---

# Windows-Containerimages

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

>Windows-Container werden mit Docker verwaltet. Die Dokumentation zu Windows-Containern ergänzt die Dokumentation, die Sie auf [docker.com](https://www.docker.com/) finden.

Containerimages dienen zum Bereitstellen von Containern. Diese Images können ein Anwendungen und sämtliche Anwendungsabhängigkeiten enthalten. Sie können z. B. ein Containerimage entwickeln, das mit Nano Server, IIS und einer in IIS ausgeführten Anwendung vorkonfiguriert wurde. Dieses Containerimage kann anschließend für die spätere Verwendung in einer Containerregistrierung gespeichert, auf einem beliebigen Windows-Containerhost (lokal, Cloud oder Containerdienst) bereitgestellt und außerdem als Basis für ein neues Containerimage verwendet werden.

### Installieren eines Images

Vor der Arbeit mit Windows-Containern muss ein Basisimage installiert werden. Basisimages sind mit Windows Server Core oder Nano Server als zugrunde liegendem Betriebssystem verfügbar. Informationen zu den unterstützten Konfigurationen finden Sie unter [Systemanforderungen für Windows-Container](../deployment/system_requirements.md).

Zum Installieren des Basisimages für Windows Server Core führen Sie folgenden Befehl aus:

```none
docker pull microsoft/windowsservercore
```

Zum Installieren des Basisimages für Nano Server führen Sie folgenden Befehl aus:

```none
docker pull microsoft/nanoserver
```

### Auflisten von Images

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        9 weeks ago         7.764 GB
microsoft/nanoserver          latest              3a703c6e97a2        9 weeks ago         969.8 MB
```

### Erstellen eines neuen Images

Sie können ein neues Containerimage aus einem vorhandenen Container erstellen. Verwenden Sie hierzu den Befehl `docker commit`. Dieses Beispiel erstellt ein neues Containerimage mit dem Namen „windowsservercoreiis“.

```none
docker commit 475059caef8f windowsservercoreiis
```

### Entfernen eines Images

Containerimages können nicht entfernt werden, wenn ein Container, selbst mit dem Status „Beendet“, eine Abhängigkeit vom Image aufweist.

Wenn Sie ein Image mit Docker entfernen, kann auf die Images anhand des Imagenamens oder der ID verwiesen werden.

```none
docker rmi windowsservercoreiis
```

### Imageabhängigkeit

Zum Anzeigen von Imageabhängigkeiten mit Docker kann der Befehl `docker history` verwendet werden.

```none
docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

Die Docker Hub-Registrierung enthält vordefinierte Images, die auf einen Containerhost heruntergeladen werden können. Sobald diese Images heruntergeladen wurden, können sie als Basis für Windows-Containeranwendungen verwendet werden.

Verwenden Sie den Befehl `docker search`, um eine Liste der auf Docker Hub verfügbaren Images anzuzeigen. Hinweis: Die Basisbetriebssystemimages für Windows Server Core oder Nano Server müssen installiert werden, ehe diese abhängigen Images von Docker Hub bezogen werden.

Die meisten dieser Images verfügen über eine Windows Server Core- und eine Nano Server-Version. Um eine bestimmte Version abzurufen, fügen Sie das Tag „:windowsservercore“ oder „:nanoserver“ hinzu. Das Tag „latest“ gibt standardmäßig die Windows Server Core-Version zurück, sofern nicht nur eine Nano Server-Version verfügbar ist.


```none
docker search *

NAME                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/sample-django  Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35       .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/sample-golang  Go Programming Language installed in a Win...   1                    [OK]
microsoft/sample-httpd   Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis            Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/sample-mongodb MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/sample-mysql   MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-nginx   Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-node    Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-python  Python installed in a Windows Server Core ...   1                    [OK]
microsoft/sample-rails   Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/sample-redis   Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-ruby    Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-sqlite  SQLite installed in a Windows Server Core ...   1                    [OK]
```

### Docker Pull

Verwenden Sie den Befehl `docker pull`, um ein Image von Docker Hub herunterzuladen. Weitere Informationen finden Sie unter [Docker Pull auf Docker.com](https://docs.docker.com/engine/reference/commandline/pull/).

```none
docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

Bei Ausführung von `docker images` wird das Image jetzt angezeigt.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.14300.1000     6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```

> Wenn in Docker ein Fehler auftritt, stellen Sie sicher, dass auf dem Containerhost die neuesten kumulativen Updates installiert wurden. Das TP5-Update finden Sie unter [KB3157663]( https://support.microsoft.com/en-us/kb/3157663).

### Docker Push

Containerimages können auch auf Docker Hub oder in eine vertrauenswürdige Docker Trusted Registry hochgeladen werden. Nachdem diese Bilder hochgeladen wurden, können sie heruntergeladen und in verschiedenen Windows-Container-Umgebungen verwendet werden.

Um ein Containerimage in Docker Hub hochzuladen, melden Sie sich zuerst bei der Registrierung an. Weitere Informationen finden Sie unter [Docker Login auf Docker.com]( https://docs.docker.com/engine/reference/commandline/login/).

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: username
Password:

Login Succeeded
```

Nachdem Sie sich bei Docker Hub oder Docker Trusted Registry angemeldet haben, können Sie ein Containerimage mit `docker push` hochladen. Auf das Containerimage kann anhand des Namens oder der ID verwiesen werden. Weitere Informationen finden Sie unter [Docker Push auf Docker.com]( https://docs.docker.com/engine/reference/commandline/push/).

```none
docker push username/containername

The push refers to a repository [docker.io/username/containername]
b567cea5d325: Pushed
00f57025c723: Pushed
2e05e94480e9: Pushed
63f3aa135163: Pushed
469f4bf35316: Pushed
2946c9dcfc7d: Pushed
7bfd967a5e43: Pushed
f64ea92aaebc: Pushed
4341be770beb: Pushed
fed398573696: Pushed
latest: digest: sha256:ae3a2971628c04d5df32c3bbbfc87c477bb814d5e73e2787900da13228676c4f size: 2410
```

Jetzt ist das Containerimage verfügbar, und Sie können mit `docker pull` darauf zugreifen.






<!--HONumber=Aug16_HO4-->


