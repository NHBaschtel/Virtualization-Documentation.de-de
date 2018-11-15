---
title: Containerbereitstellung – Schnellstart – Images
description: Containerbereitstellung – Schnellstart
keywords: Docker, Container
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
ms.openlocfilehash: 104c8f659e2b9709c24eb0230d9f32d6dca32c71
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948039"
---
# <a name="automating-builds-and-saving-images"></a>Automatisieren von Builds und Speichern von Images

In der vorhergehenden Schnellstart-Anleitung für Windows Server wurde ein Windows-Container auf Basis eines bereits vorhandenen .NET Core-Beispiels erstellt. Diese Übung enthält ausführliche Informationen zur automatischen Erstellung von Containerimages mit einer Dockerfile-Datei und zur Speicherung von Containerimages in der öffentlichen Registrierung von Docker Hub.

Dieser Schnellstart-Anleitung richtet sich gezielt an Windows Server-Container unter Windows Server 2016 und verwendet das Windows Server Core-Containerbasisimage. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis.

**Voraussetzungen:**

- Ein Computersystem (physisch oder virtuell), auf dem Windows Server 2016 ausgeführt wird
- Konfigurieren Sie dieses System mit dem Windows-Container-Feature und Docker. Eine exemplarische Vorgehensweise zu diesen Schritten finden Sie unter [Windows Containers on Windows Server](./quick-start-windows-server.md) (Windows-Container unter Windows Server).
- Eine Docker-ID – diese wird verwendet, um ein Containerimage mithilfe von Push an Docker Hub zu übertragen. Sollten Sie noch nicht über eine Docker-ID verfügen, können Sie diese über eine Registrierung bei [Docker Cloud](https://cloud.docker.com/) beziehen.

## <a name="1-container-image---dockerfile"></a>1. Containerimage – Dockerfile

Obwohl ein Container manuell erstellt, geändert und dann in einem neuen Containerimage erfasst werden kann, enthält Docker eine Methode zum Automatisieren dieses Prozesses mithilfe einer Dockerfile. Für diese Übung ist eine Docker-ID erforderlich. Sollten Sie noch nicht über eine Docker-ID verfügen, können Sie diese über eine Registrierung bei [Docker Cloud]( https://cloud.docker.com/) beziehen.

Erstellen Sie auf dem Containerhost das Verzeichnis `c:\build` und in diesem Verzeichnis eine Datei namens `Dockerfile`. Hinweis – Die Datei sollte keine Dateierweiterung haben.

```
powershell new-item c:\build\Dockerfile -Force
```

Öffnen Sie die Dockerfile-Datei in Editor.

```
notepad c:\build\Dockerfile
```

Kopieren Sie den folgenden Text in die Dockerfile-Datei, und speichern Sie sie. Diese Befehle weisen Docker an, ein neues Image mit `microsoft/iis` als Basis zu erstellen. Die Dockerfile-Datei führt dann die in der `RUN`-Anweisung angegebenen Befehle aus. In diesem Fall wird die Datei „index.html“ mit neuem Inhalt aktualisiert. 

Weitere Informationen zu Dockerfile-Dateien finden Sie unter [Dockerfiles on Windows](../manage-docker/manage-windows-dockerfile.md) (Dockerfile-Dateien unter Windows).

```
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

Der `docker build`-Befehl startet den Imageerstellungsprozess. Der Parameter `-t` weist den Erstellungsprozess an, das neue Image `iis-dockerfile` zu nennen. **Ersetzen Sie „user“ durch den Benutzernamen Ihres Docker-Kontos (Ihre Docker-ID)**. Sollten Sie noch nicht über ein Docker-Konto verfügen, können Sie sich durch die Registrierung bei [Docker Cloud](https://cloud.docker.com/) ein Konto anlegen.

```
docker build -t <user>/iis-dockerfile c:\Build
```

Abschließend können Sie mithilfe des Befehls `docker images` prüfen, ob das Image erstellt wurde.

```
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Stellen Sie jetzt mithilfe des folgenden Befehls einen Container bereit. Ersetzen Sie erneut „user“ durch Ihre Docker-ID.

```
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

Nachdem der Container erstellt wurde, navigieren Sie zur IP-Adresse des Containerhosts. Die „Hello World“-Anwendung sollte angezeigt werden.

![](media/dockerfile2.png)

Wenn Sie auf dem Containerhost zurück sind, rufen Sie mit `docker ps` den Namen des Containers ab, und entfernen Sie den Container mit `docker rm`. Hinweis – Ersetzen Sie den Namen des Containers in diesem Beispiel durch den tatsächlichen Containernamen.

Rufen Sie den Containernamen ab.

```
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```
Container beendet.

```
docker stop <container name>
```

Entfernen Sie den Container.

```
docker rm -f <container name>
```

## <a name="2-docker-push"></a>2. Docker Push

Docker-Containerimages können in einer Containerregistrierung gespeichert werden. Sobald ein Image in einer Registrierung gespeichert ist, kann es von vielen verschiedenen Containerhosts für die spätere Verwendung abgerufen werden. Docker stellt unter [Docker Hub](https://hub.docker.com/) eine öffentliche Registrierung für das Speichern von Containerimages bereit.

Für diese Übung wird das benutzerdefinierte Hello World-Image mithilfe von Push in Ihr eigenes Konto auf Docker Hub übertragen.

Melden Sie sich zuerst mithilfe des Befehls `docker login command` bei Ihrem Docker-Konto an.

```
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

Sobald Sie angemeldet sind, kann das Containerimage mithilfe von Push an Docker Hub übertragen werden. Verwenden Sie hierzu den Befehl `docker push`. **Ersetzen Sie „User“ durch Ihre Docker-ID**. 

```
docker push <user>/iis-dockerfile
```

Das Containerimage kann jetzt mithilfe von `docker pull` von Docker Hub auf jeden beliebigen Windows-Containerhost heruntergeladen werden. In diesem Tutorial löschen wir das bestehende Image und übertragen es mithilfe von Pull von Docker Hub. 

```
docker rmi <user>/iis-dockerfile
```

Das Ausführen von `docker images` zeigt, dass das Docker-Image entfernt wurde.

```
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

Abschließend können Sie nun „docker pull“ verwenden, um das Image mithilfe von Pull wieder auf den Containerhost zu übertragen. Ersetzen Sie „user“ durch den Benutzernamen Ihres Docker-Kontos. 

```
docker pull <user>/iis-dockerfile
```

## <a name="next-steps"></a>Nächste Schritte

In den folgenden Windows10-Lernprogrammen erfahren Sie, wie Sie eine ASP.NET-Beispielanwendung packen.

[Windows-Container unter Windows10](./quick-start-windows-10.md)
