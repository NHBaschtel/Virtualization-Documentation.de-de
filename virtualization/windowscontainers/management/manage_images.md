---
title: Windows-Containerimages
description: Erstellen und Verwalten von Containerimages mit Windows-Containern.
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
ms.sourcegitcommit: 3db43b433e7b1a9484d530cf209ea80ef269a307
ms.openlocfilehash: 505cc64fa19fb9fc8c2d5c109830f460f09332dd

---

# Windows-Containerimages

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

Containerimages dienen zum Bereitstellen von Containern. Diese Images können ein Betriebssystem, Anwendungen und sämtliche Anwendungsabhängigkeiten enthalten. Sie können z. B. ein Containerimage entwickeln, das mit Nano Server, IIS und einer in IIS ausgeführten Anwendung vorkonfiguriert wurde. Dieses Containerimage kann anschließend für die spätere Verwendung in einer Containerregistrierung gespeichert, auf einem beliebigen Windows-Containerhost (lokal, Cloud oder Containerdienst) bereitgestellt und außerdem als Basis für ein neues Containerimage verwendet werden.

Es gibt zwei Arten von Containerimages:

**Basisbetriebssystemimages**: Werden von Microsoft bereitgestellt und enthalten die wesentlichen Betriebssystemkomponenten. 

**Containerimages**: Ein benutzerdefiniertes Containerimage, das vom Basisbetriebssystemimage abgeleitet wird.

## Basisbetriebssystemimages

### Installieren eines Images

Images von Containerbetriebssystemen können mithilfe des PowerShell-Moduls „ContainerImage“ bestimmt und installiert werden. Sie müssen dieses Modul erst installieren, um es verwenden zu können. Installieren Sie das Modul mit dem folgenden Befehl. Weitere Informationen zur Verwendung des PowerShell-Moduls „Containers Image OneGet“ finden Sie unter [Container Image Provider](https://github.com/PowerShell/ContainerProvider) (Containerimageanbieter). 

```none
Install-PackageProvider ContainerImage -Force
```

Nach der Installation kann mit `Find-ContainerImage` eine Liste der Basisbetriebssystemimages zurückgegeben werden.

```none
Find-ContainerImage

Name                 Version          Source           Summary
----                 -------          ------           -------
NanoServer           10.0.14300.1010  ContainerImag... Container OS Image of Windows Server 2016 Technical...
WindowsServerCore    10.0.14300.1000  ContainerImag... Container OS Image of Windows Server 2016 Technical...
```

Zum Herunterladen und Installieren des Basisbetriebssystem-Images für Nano Server führen Sie die folgenden Schritte aus. Der Parameter `-version` ist optional. Wenn keine Version für das Basisbetriebssystem-Image angegeben wird, wird die neueste Version installiert.

```none
Install-ContainerImage -Name NanoServer -Version 10.0.14300.1010
```

Mit diesem Befehl wird auch das Basisbetriebssystemimage für Windows Server Core heruntergeladen und installiert. Der Parameter `-version` ist optional. Wenn keine Version für das Basisbetriebssystem-Image angegeben wird, wird die neueste Version installiert.

```none
Install-ContainerImage -Name WindowsServerCore -Version 10.0.14300.1000
```

Überprüfen Sie mit dem Befehl `docker images`, ob die Images Installiert wurden. 

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     40356b90dc80        2 weeks ago         793.3 MB
windowsservercore   10.0.14304.1000     7837d9445187        2 weeks ago         9.176 GB
```  

Nach der Installation sollten Sie die Images auch mit dem Tag „latest“ kennzeichnen. Anweisungen dazu finden Sie im Abschnitt „Kennzeichnen von Images“ weiter unten.

> Wenn das Basisbetriebssystemimage heruntergeladen wurde, aber beim Ausführen von `docker images` nicht angezeigt wird, starten Sie den Docker-Dienst mithilfe des Systemsteuerungs-Applets für Dienste oder den Befehlen „sc stop docker“ und „sc start docker“ neu.

### Kennzeichnen von Images

Wenn auf ein Containerimage anhand des Namens verwiesen wird, sucht das Docker-Modul nach der neuesten Version des Images. Wenn die neueste Version nicht bestimmt werden kann, wird der folgende Fehler ausgelöst.

```none
docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

Nach der Installation von Windows Server Core- oder Nano Server-Basisbetriebssystemimages müssen diese mit „Latest“ als neueste Versionen gekennzeichnet werden. Verwenden Sie hierzu den Befehl `docker tag`. 

Weitere Informationen zu `docker tag` finden Sie auf docker.com unter [Tag, push, and pull your image](https://docs.docker.com/mac/step_six/) (Kennzeichnung, Push- und Pullvorgänge für Ihr Image). 

```none
docker tag <image id> windowsservercore:latest
```

Nach dem Kennzeichnen werden in der Ausgabe von `docker images` zwei Versionen desselben Images angezeigt: eines mit dem Tag der Imageversion und ein zweites mit dem Tag „latest“. Jetzt kann mit dem Namen auf das Image verwiesen werden.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14300.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### Offlineinstallation

Basisbetriebssystemimages können auch ohne Internetzugang installiert werden. Laden Sie zu diesem Zweck das Image auf einen Computer mit Internetzugang herunter, kopieren Sie es auf das Zielsystem, und importieren Sie das Image dann mithilfe des Befehls `Install-ContainerOSImages`.

Bereiten Sie das **mit dem Internet verbundene** System vor dem Herunterladen des Basisbetriebssystemimages durch Ausführen des folgenden Befehls mit dem Containerimageanbieter vor.

```none
Install-PackageProvider ContainerImage -Force
```

So geben Sie eine Liste der Images aus dem PowerShell OneGet-Paket-Manager zurück:

```none
Find-ContainerImage
```

Ausgabe:

```none
Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.14300.1010         Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.14300.1000         Container OS Image of Windows Server 2016 Techn...
```

Verwenden Sie den Befehl `Save-ContainerImage`, um ein Image herunterzuladen.

```none
Save-ContainerImage -Name NanoServer -Path c:\container-image
```

Das heruntergeladene Containerimage kann jetzt auf den **Offlinecontainerhost** kopiert und mit dem Befehl `Install-ContainerOSImage` installiert werden.

```none
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### Deinstallieren des Betriebssystemimages

Basisbetriebssystemimages können mithilfe des Befehls `Uninstall-ContainerOSImage` deinstalliert werden. Im folgende Beispiel wird das NanoServer-Basisbetriebssystemimage deinstalliert.

```none
Uninstall-ContainerOSImage -FullName CN=Microsoft_NanoServer_10.0.14304.1003
```

## Containerimages

### Auflisten von Images

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.14300.1000     6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.14300.1010     8572198a60f1        2 weeks ago          0 B
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






<!--HONumber=Jun16_HO3-->


