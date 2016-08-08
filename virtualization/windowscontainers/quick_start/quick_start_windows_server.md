---
title: Windows-Container unter Windows Server
description: "Containerbereitstellung – Schnellstart"
keywords: Docker, Container
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 6c7ce9f1767c6c6391cc6d33a553216bd815ff72
ms.openlocfilehash: 4e7123dcff2564bd264c91f228941e86a0723674

---

# Windows-Container unter Windows Server

**Dieser Inhalt ist vorläufig und kann geändert werden.**

Die Übung führt durch die einfache Bereitstellung und Verwendung des Windows-Container-Features unter Windows Server. Nach der Ausführung haben Sie die Containerrolle installiert und einen einfachen Windows Server-Container bereitgestellt. Machen Sie sich vor diesem Schnellstart mit grundlegenden Containerkonzepten und der Terminologie vertraut. Diese Informationen finden Sie unter [Windows Containers Quick Start](./quick_start.md) (Windows-Container – Schnellstart).

Dieser Schnellstart ist spezifisch für Windows Server-Container unter Windows Server 2016. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis.

**Voraussetzungen:**

Ein Computersystem (physisch oder virtuell), das [Windows Server 2016 Technical Preview 5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview) ausführt.

Ein vollständig konfiguriertes Windows Server-Image ist in Azure verfügbar. Um dieses Image zu verwenden, stellen Sie durch Klicken auf die Schaltfläche unten eine virtuelle Maschine bereit. Die Bereitstellung dauert ca. 10 Minuten. Sobald sie abgeschlossen ist, melden Sie sich beim virtuellen Azure-Computer an, und fahren Sie mit Schritt 4 dieses Tutorials fort. 

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## 1. Installieren des Containerfeatures

Das Containerfeature muss aktiviert werden, bevor Sie mit Windows-Containern arbeiten können. Führen Sie dazu den folgenden Befehl in einer PowerShell-Sitzung mit erhöhten Rechten aus.

```none
Install-WindowsFeature containers
```

Starten Sie den Computer neu, wenn die Installation des Features abgeschlossen ist.

```none
Restart-Computer -Force
```

## 2. Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Für diese Übung werden beide installiert.

Laden Sie das Docker-Modul und den Docker-Client als ZIP-Archiv herunter.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Erweitern Sie das ZIP-Archiv in „Programme“, die Archivinhalte befinden sich bereits im Docker-Verzeichnis.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Fügen Sie das Docker-Verzeichnis dem Systempfad hinzu. Starten Sie anschließend die PowerShell-Sitzung neu, damit der geänderte Pfad erkannt wird.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Führen Sie den folgenden Befehl aus, um Docker als Windows-Dienst zu installieren.

```none
& $env:ProgramFiles\docker\dockerd.exe --register-service
```

Nach Abschluss der Installation kann der Dienst gestartet werden.

```none
Start-Service docker
```

## 3. Installieren von Basiscontainerimages

Windows-Container werden in Vorlagen oder Images bereitgestellt. Bevor ein Container bereitgestellt werden kann, muss ein Basisimage des Betriebssystems heruntergeladen werden. Mit den folgenden Befehlen wird das Basisimage für Windows Server Core heruntergeladen.

Installieren Sie zunächst den Paketanbieter für Containerimages.

```none
Install-PackageProvider ContainerImage -Force
```

Installieren Sie dann das Windows Server Core-Image. Dieser Vorgang kann einige Zeit dauern. Sie können also eine Pause machen und den Vorgang fortsetzen, sobald der Download abgeschlossen ist.

```none
Install-ContainerImage -Name WindowsServerCore    
```

Nachdem das Basisimage installiert wurde, muss der Docker-Dienst neu gestartet werden.

```none
Restart-Service docker
```

In dieser Phase gibt die Ausführung von `docker images` eine Liste der installierten Images zurück, in diesem Fall das Windows Server Core-Image.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
windowsservercore   10.0.14300.1000     dbfee88ee9fd        7 weeks ago         9.344 GB
```

Bevor Sie fortfahren, müssen Sie die Version dieses Images als „latest“ (neueste) kennzeichnen. Führen Sie dazu den folgenden Befehl aus:

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

Ausführliche Informationen zu Windows-Containerimages finden Sie unter [Verwalten von Containerimages](../management/manage_images.md).

## 4. Bereitstellen Ihres ersten Containers

Für diese Übung laden Sie ein vorab erstelltes IIS-Image von der Docker Hub-Registrierung herunter und stellen einen einfachen Container bereit, der IIS ausführt.  

Um Docker Hub nach Windows-Containerimages zu durchsuchen, führen Sie `docker search Microsoft` aus.  

```none
docker search microsoft

NAME                                         DESCRIPTION                                     
microsoft/sample-django:windowsservercore    Django installed in a Windows Server Core ...   
microsoft/dotnet35:windowsservercore         .NET 3.5 Runtime installed in a Windows Se...   
microsoft/sample-golang:windowsservercore    Go Programming Language installed in a Win...   
microsoft/sample-httpd:windowsservercore     Apache httpd installed in a Windows Server...   
microsoft/iis:windowsservercore              Internet Information Services (IIS) instal...   
microsoft/sample-mongodb:windowsservercore   MongoDB installed in a Windows Server Core...   
microsoft/sample-mysql:windowsservercore     MySQL installed in a Windows Server Core b...   
microsoft/sample-nginx:windowsservercore     Nginx installed in a Windows Server Core b...  
microsoft/sample-python:windowsservercore    Python installed in a Windows Server Core ...   
microsoft/sample-rails:windowsservercore     Ruby on Rails installed in a Windows Serve...  
microsoft/sample-redis:windowsservercore     Redis installed in a Windows Server Core b...   
microsoft/sample-ruby:windowsservercore      Ruby installed in a Windows Server Core ba...   
microsoft/sample-sqlite:windowsservercore    SQLite installed in a Windows Server Core ...  
```

Laden Sie das IIS-Image mit `docker pull` herunter.  

```none
docker pull microsoft/iis:windowsservercore
```

Der Imagedownload kann mit dem `docker images`-Befehl überprüft werden. Beachten Sie dabei, dass sowohl das Basisimage (windowsservercore) als auch das IIS-Image angezeigt wird.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

Stellen Sie den IIS-Container mit `docker run` bereit.

```none
docker run -d -p 80:80 microsoft/iis:windowsservercore ping -t localhost
```

Mit diesem Befehl wird das IIS-Image als Hintergrunddienst ausgeführt (-d) und das Netzwerk so konfiguriert, dass Port 80 des Containerhosts Port 80 des Containers zugeordnet ist.
Weitere Informationen zum Befehl „Docker Run“ finden Sie in der [Referenz zu „Docker Run“ auf Docker.com]( https://docs.docker.com/engine/reference/run/).


Ausgeführte Container können mit dem `docker ps`-Befehl angezeigt werden. Notieren Sie sich den Containernamen, denn er wird in einem späteren Schritt verwendet.

```none
docker ps

CONTAINER ID    IMAGE                             COMMAND               CREATED              STATUS   PORTS                NAMES
9cad3ea5b7bc    microsoft/iis:windowsservercore   "ping -t localhost"   About a minute ago   Up       0.0.0.0:80->80/tcp   grave_jang
```

Öffnen Sie auf einem anderen Computer einen Webbrowser, und geben Sie die IP-Adresse des Containerhosts ein. Wenn alles richtig konfiguriert wurde, sollte der IIS-Begrüßungsbildschirm angezeigt werden. Dies erfolgt über die IIS-Instanz, die im Windows-Container gehostet wird.

**Hinweis:** Wenn Sie Azure verwenden, werden die externe IP-Adresse des virtuellen Computers und eine konfigurierte Netzwerksicherheit benötigt. Weitere Informationen finden Sie unter [Erstellen einer Regel in einer Netzwerksicherheitsgruppe]( https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg).

![](media/iis1.png)

Wenn Sie wieder auf dem Containerhost zurück sind, entfernen Sie den Container mit dem `docker rm`-Befehl. Hinweis – Ersetzen Sie den Namen des Containers in diesem Beispiel durch den tatsächlichen Containernamen.

```none
docker rm -f grave_jang
```
## Nächste Schritte

[Containerimages unter Windows Server](./quick_start_images.md)

[Windows-Container unter Windows 10](./quick_start_windows_10.md)



<!--HONumber=Aug16_HO1-->


