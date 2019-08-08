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
ms.openlocfilehash: 93c56dba88715df41cab054cda676879b275380b
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999207"
---
# <a name="automating-builds-and-saving-images"></a>Automatisieren von Builds und Speichern von Images

In der vorhergehenden Schnellstart-Anleitung für Windows Server wurde ein Windows-Container auf Basis eines bereits vorhandenen .NET Core-Beispiels erstellt. In dieser Übung wird gezeigt, wie Sie ein eigenes Container Bild aus einem Dockerfile erstellen und das Container Bild in der öffentlichen Registrierung des andockbaren Hubs speichern.

Dieser Schnellstart ist speziell für Windows Server-Container unter Windows Server 2019 oder Windows Server 2016 und verwendet das Windows Server Core-Containerbasis Abbild. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis.

## <a name="prerequisites"></a>Voraussetzungen

Stellen Sie bitte sicher, dass Sie die folgenden Voraussetzungen erfüllen:

- Ein physisches oder virtuelles Computersystem, auf dem Windows Server 2019 oder Windows Server 2016 ausgeführt wird.
- Konfigurieren Sie dieses System mit dem Windows-Container Feature und Docker. Eine exemplarische Vorgehensweise zu diesen Schritten finden Sie unter [Windows-Container unter Windows Server](./quick-start-windows-server.md).
- Eine Docker-ID – diese wird verwendet, um ein Containerimage mithilfe von Push an Docker Hub zu übertragen. Sollten Sie noch nicht über eine Docker-ID verfügen, können Sie diese über eine Registrierung bei [Docker Cloud](https://cloud.docker.com/) beziehen.

## <a name="container-image---dockerfile"></a>Container Bild – Dockerfile

Obwohl ein Container manuell erstellt, geändert und dann in einem neuen Containerimage erfasst werden kann, enthält Docker eine Methode zum Automatisieren dieses Prozesses mithilfe einer Dockerfile. Für diese Übung ist eine Docker-ID erforderlich. Sollten Sie noch nicht über eine Docker-ID verfügen, können Sie diese über eine Registrierung bei [Docker Cloud](https://cloud.docker.com/) beziehen.

Erstellen Sie auf dem Containerhost das Verzeichnis `c:\build` und in diesem Verzeichnis eine Datei namens `Dockerfile`. Hinweis – Die Datei sollte keine Dateierweiterung haben.

```console
powershell new-item c:\build\Dockerfile -Force
```

Öffnen Sie die Dockerfile-Datei in Editor.

```console
notepad c:\build\Dockerfile
```

Kopieren Sie den folgenden Text in die Dockerfile-Datei, und speichern Sie sie. Diese Befehle weisen Docker an, ein neues Image mit `microsoft/iis` als Basis zu erstellen. Die Dockerfile-Datei führt dann die in der `RUN`-Anweisung angegebenen Befehle aus. In diesem Fall wird die Datei „index.html“ mit neuem Inhalt aktualisiert.

Weitere Informationen zu Dockerfile-Dateien finden Sie unter [Dockerfiles on Windows](../manage-docker/manage-windows-dockerfile.md) (Dockerfile-Dateien unter Windows).

```dockerfile
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

Der `docker build`-Befehl startet den Imageerstellungsprozess. Der Parameter `-t` weist den Erstellungsprozess an, das neue Image `iis-dockerfile` zu nennen. **Ersetzen Sie „user“ durch den Benutzernamen Ihres Docker-Kontos (Ihre Docker-ID)**. Sollten Sie noch nicht über ein Docker-Konto verfügen, können Sie sich durch die Registrierung bei [Docker Cloud](https://cloud.docker.com/) ein Konto anlegen.

```console
docker build -t <user>/iis-dockerfile c:\Build
```

Abschließend können Sie mithilfe des Befehls `docker images` prüfen, ob das Image erstellt wurde.

```console
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Stellen Sie jetzt mithilfe des folgenden Befehls einen Container bereit. Ersetzen Sie erneut „user“ durch Ihre Docker-ID.

```console
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

Nachdem der Container erstellt wurde, navigieren Sie zur IP-Adresse des Containerhosts. Die „Hello World“-Anwendung sollte angezeigt werden.

![](media/dockerfile2.png)

Wenn Sie auf dem Containerhost zurück sind, rufen Sie mit `docker ps` den Namen des Containers ab, und entfernen Sie den Container mit `docker rm`. Hinweis – Ersetzen Sie den Namen des Containers in diesem Beispiel durch den tatsächlichen Containernamen.

Rufen Sie den Containernamen ab.

```console
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

Container beenden

```console
docker stop <container name>
```

Entfernen Sie den Container.

```console
docker rm -f <container name>
```

## <a name="docker-push"></a>Andocken-Push

Docker-Containerimages können in einer Containerregistrierung gespeichert werden. Sobald ein Image in einer Registrierung gespeichert ist, kann es von vielen verschiedenen Containerhosts für die spätere Verwendung abgerufen werden. Docker stellt unter [Docker Hub](https://hub.docker.com/) eine öffentliche Registrierung für das Speichern von Containerimages bereit.

Für diese Übung wird das benutzerdefinierte Hello World-Image mithilfe von Push in Ihr eigenes Konto auf Docker Hub übertragen.

Melden Sie sich zuerst mithilfe des Befehls `docker login command` bei Ihrem Docker-Konto an.

```console
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

Sobald Sie angemeldet sind, kann das Containerimage mithilfe von Push an Docker Hub übertragen werden. Verwenden Sie hierzu den Befehl `docker push`. **Ersetzen Sie „User“ durch Ihre Docker-ID**. 

```console
docker push <user>/iis-dockerfile
```

Wenn der andocker jede Ebene nach oben in den docker-Hub schiebt, überspringt Andocken Layer, die bereits im Andockfenster vorhanden sind, oder in anderen Registern (fremd Ebenen).  So werden beispielsweise neuere Versionen von Windows Server Core, die in der Microsoft-Container Registrierung gehostet werden, oder Layer aus einer privaten Unternehmensregistrierung übersprungen und nicht auf den docker-Hub verschoben.

Das Containerimage kann jetzt mithilfe von `docker pull` von Docker Hub auf jeden beliebigen Windows-Containerhost heruntergeladen werden. In diesem Tutorial löschen wir das bestehende Image und übertragen es mithilfe von Pull von Docker Hub. 

```console
docker rmi <user>/iis-dockerfile
```

Das Ausführen von `docker images` zeigt, dass das Docker-Image entfernt wurde.

```console
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

> [!div class="nextstepaction"]
> [Container unter Windows 10](./quick-start-windows-10.md)
