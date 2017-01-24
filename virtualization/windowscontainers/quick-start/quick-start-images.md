---
title: "Containerbereitstellung – Schnellstart – Images"
description: "Containerbereitstellung – Schnellstart"
keywords: Docker, Container
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
translationtype: Human Translation
ms.sourcegitcommit: 5d746be15856aad683bf684d2eef573d732ab457
ms.openlocfilehash: 6add396bea629d5438cde5892458f6c8405bf644

---

# Containerimages unter Windows Server

In der vorhergehenden Schnellstart-Anleitung für Windows Server wurde ein Windows-Container auf Basis eines bereits vorhandenen .NET Core-Beispiels erstellt. Diese Übung enthält ausführliche Informationen zum manuellen Erstellen von benutzerdefinierten Containerimages, zur automatischen Erstellung von Containerimages mit einer Dockerfile-Datei und zur Speicherung von Containerimages in der öffentlichen Registrierung von Docker Hub.

Dieser Schnellstart-Anleitung richtet sich gezielt an Windows Server-Container unter Windows Server 2016 und verwendet das Windows Server Core-Containerbasisimage. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis.

**Voraussetzungen:**

- Ein Computersystem (physisch oder virtuell), auf dem Windows Server 2016 ausgeführt wird
- Konfigurieren Sie dieses System mit dem Windows-Container-Feature und Docker. Eine exemplarische Vorgehensweise zu diesen Schritten finden Sie unter [Windows Containers on Windows Server](./quick-start-windows-server.md) (Windows-Container unter Windows Server).
- Eine Docker-ID – diese wird verwendet, um ein Containerimage mithilfe von Push an Docker Hub zu übertragen. Sollten Sie noch nicht über eine Docker-ID verfügen, können Sie diese über eine Registrierung bei [Docker Cloud](https://cloud.docker.com/) beziehen.

## 1. Containerimage – manuell

Für optimale Ergebnisse absolvieren Sie diese Übung in einer Windows-Befehlsshell (cmd.exe).

Der erste Schritt beim manuellen Erstellen eines Containerimages ist die Bereitstellung eines Containers. In diesem Beispiel stellen Sie einen IIS-Container auf der Basis des zuvor erstellten IIS-Images bereit. Nachdem der Container bereitgestellt wurde, arbeiten Sie in einer Shellsitzung innerhalb des Containers. Die interaktive Sitzung wird mit dem `-it`-Flag eingeleitet. Detaillierte Informationen zu „Docker Run“-Befehlen finden Sie unter [„Docker run reference“ auf Docker.com](https://docs.docker.com/engine/reference/run/) („Docker Run“-Referenz). 

> Aufgrund der Größe des Windows Server Core-Basisimages kann dieser Vorgang eine Weile dauern.

```none
docker run -d --name myIIS -p 80:80 microsoft/iis
```

Nun wird der Container im Hintergrund ausgeführt. Der Standardbefehl im Container ist `ServiceMonitor.exe`, womit der IIS-Status überwacht und der Container automatisch beendet wird, sobald IIS stoppt. Weitere Informationen darüber, wie dieses Image erstellt wurde, finden Sie unter [Microsoft/docker-iis](https://github.com/Microsoft/iis-docker) auf GitHub.

Starten Sie nun eine interaktive Befehlszeile im Container. Damit können Sie Befehle in einem aktiven Container ausführen, ohne IIS oder ServiceMonitor zu beenden.

```none
docker exec -i myIIS cmd 
```

Als Nächstes nehmen Sie eine Änderung an dem ausgeführten Container vor. Führen Sie zum Entfernen des IIS-Begrüßungsbildschirms den folgenden Befehl aus.

```none
del C:\inetpub\wwwroot\iisstart.htm
```

Und führen Sie den folgenden Befehl aus, um die IIS-Standardwebsite durch eine neue statische Website zu ersetzen.

```none
echo "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

Wechseln Sie auf einem anderen System zur IP-Adresse des Containerhosts. Die „Hello World“-Anwendung sollte jetzt angezeigt werden.

**Hinweis:** Wenn Sie in Azure arbeiten, muss eine Netzwerksicherheits-Gruppenregel vorhanden sein, die Datenverkehr über Port 80 zulässt. Weitere Informationen finden Sie unter [Erstellen einer Regel in einer Netzwerksicherheitsgruppe](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg).

![](media/hello.png)

Wenn Sie zurück im Container sind, beenden Sie die interaktive Containersitzung.

```none
exit
```

Der geänderte Container kann jetzt in einem neuen Containerimage erfasst werden. Dazu benötigen Sie den Containernamen. Dieses können Sie mit dem Befehl `docker ps -a` suchen.

```none
docker ps -a

CONTAINER ID     IMAGE                             COMMAND   CREATED             STATUS   PORTS   NAMES
489b0b447949     microsoft/iis   "cmd"     About an hour ago   Exited           pedantic_lichterman
```

Um ein neues Containerimage zu erstellen, verwenden Sie den Befehl `docker commit`. „Docker Commit“ hat die Form „docker commit Containername Neuer_Imagename“. Hinweis – Ersetzen Sie den Namen des Containers in diesem Beispiel durch den tatsächlichen Containernamen.

```none
docker commit pedantic_lichterman modified-iis
```

Um zu überprüfen, ob das neue Image erstellt wurde, verwenden Sie den `docker images`-Befehl.  

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
modified-iis        latest              3e4fdb6ed3bc        About a minute ago   10.17 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago          9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago          9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago          9.344 GB
```

Dieses Image kann jetzt bereitgestellt werden. Der resultierende Container enthält alle erfassten Änderungen.

## 2. Containerimage – Dockerfile

In der letzten Übung wurde ein Container manuell erstellt, geändert und dann in einem neuen Container-Image erfasst. Docker bietet eine Methode für das Automatisieren dieses Prozesses mithilfe einer Dockerfile-Datei. Diese Übung liefert fast die gleichen Ergebnisse wie die letzte, doch diesmal ist der Prozess automatisiert. Für diese Übung ist eine Docker-ID erforderlich. Sollten Sie noch nicht über eine Docker-ID verfügen, können Sie diese über eine Registrierung bei [Docker Cloud]( https://cloud.docker.com/) beziehen.

Erstellen Sie auf dem Containerhost das Verzeichnis `c:\build` und in diesem Verzeichnis eine Datei namens `Dockerfile`. Hinweis – Die Datei sollte keine Dateierweiterung haben.

```none
powershell new-item c:\build\Dockerfile -Force
```

Öffnen Sie die Dockerfile-Datei in Editor.

```none
notepad c:\build\Dockerfile
```

Kopieren Sie den folgenden Text in die Dockerfile-Datei, und speichern Sie sie. Diese Befehle weisen Docker an, ein neues Image mit `microsoft/iis` als Basis zu erstellen. Die Dockerfile-Datei führt dann die in der `RUN`-Anweisung angegebenen Befehle aus. In diesem Fall wird die Datei „index.html“ mit neuem Inhalt aktualisiert. 

Weitere Informationen zu Dockerfile-Dateien finden Sie unter [Dockerfiles on Windows](../manage-docker/manage-windows-dockerfile.md) (Dockerfile-Dateien unter Windows).

```none
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

Der `docker build`-Befehl startet den Imageerstellungsprozess. Der Parameter `-t` weist den Erstellungsprozess an, das neue Image `iis-dockerfile` zu nennen. **Ersetzen Sie „user“ durch den Benutzernamen Ihres Docker-Kontos (Ihre Docker-ID)**. Sollten Sie noch nicht über ein Docker-Konto verfügen, können Sie sich durch die Registrierung bei [Docker Cloud](https://cloud.docker.com/) ein Konto anlegen.

```none
docker build -t <user>/iis-dockerfile c:\Build
```

Abschließend können Sie mithilfe des Befehls `docker images` prüfen, ob das Image erstellt wurde.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Stellen Sie jetzt mithilfe des folgenden Befehls einen Container bereit. Ersetzen Sie erneut „user“ durch Ihre Docker-ID.

```none
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

Nachdem der Container erstellt wurde, navigieren Sie zur IP-Adresse des Containerhosts. Die „Hello World“-Anwendung sollte angezeigt werden.

![](media/dockerfile2.png)

Wenn Sie auf dem Containerhost zurück sind, rufen Sie mit `docker ps` den Namen des Containers ab, und entfernen Sie den Container mit `docker rm`. Hinweis – Ersetzen Sie den Namen des Containers in diesem Beispiel durch den tatsächlichen Containernamen.

Rufen Sie den Containernamen ab.

```none
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

Entfernen Sie den Container.

```none
docker rm -f <container name>
```

## 3. Docker Push

Docker-Containerimages können in einer Containerregistrierung gespeichert werden. Sobald ein Image in einer Registrierung gespeichert ist, kann es von vielen verschiedenen Containerhosts für die spätere Verwendung abgerufen werden. Docker stellt unter [Docker Hub](https://hub.docker.com/) eine öffentliche Registrierung für das Speichern von Containerimages bereit.

Für diese Übung wird das benutzerdefinierte Hello World-Image mithilfe von Push in Ihr eigenes Konto auf Docker Hub übertragen.

Melden Sie sich zuerst mithilfe des Befehls `docker login command` bei Ihrem Docker-Konto an.

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

Sobald Sie angemeldet sind, kann das Containerimage mithilfe von Push an Docker Hub übertragen werden. Verwenden Sie hierzu den Befehl `docker push`. **Ersetzen Sie „User“ durch Ihre Docker-ID**. 

```none
docker push <user>/iis-dockerfile
```

Das Containerimage kann jetzt mithilfe von `docker pull` von Docker Hub auf jeden beliebigen Windows-Containerhost heruntergeladen werden. In diesem Tutorial löschen wir das bestehende Image und übertragen es mithilfe von Pull von Docker Hub. 

```none
docker rmi <user>/iis-dockerfile
```

Das Ausführen von `docker images` zeigt, dass das Docker-Image entfernt wurde.

```none
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

Abschließend können Sie nun „docker pull“ verwenden, um das Image mithilfe von Pull wieder auf den Containerhost zu übertragen. Ersetzen Sie „user“ durch den Benutzernamen Ihres Docker-Kontos. 

```none
docker pull <user>/iis-dockerfile
```

## Nächste Schritte

[Windows-Container unter Windows 10](./quick-start-windows-10.md)



<!--HONumber=Jan17_HO4-->


