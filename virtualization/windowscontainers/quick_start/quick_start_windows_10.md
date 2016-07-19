---
title: "Windows-Container unter Windows 10"
description: "Containerbereitstellung – Schnellstart"
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 07/07/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 5f42cae373b1f8f0484ffac82f5ebc761c37d050
ms.openlocfilehash: 9ef41ff031e8b7bc463e71f39ee6a3b8e4fd846e

---

# Windows-Container unter Windows 10

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

Die Übung führt durch die einfache Bereitstellung und Verwendung des Windows-Containerfeatures unter Windows 10 (Insiders-Build 14372 und höher). Nach der Ausführung haben Sie die Containerrolle installiert und einen einfachen Hyper-V-Container bereitgestellt. Machen Sie sich vor diesem Schnellstart mit grundlegenden Containerkonzepten und der Terminologie vertraut. Diese Informationen finden Sie unter [Windows Containers Quick Start](./quick_start.md) (Windows-Container – Schnellstart). 

Dieser Schnellstart ist spezifisch für Hyper-V-Container unter Windows 10. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis.

**Voraussetzungen:**

- Ein physisches Computersystem, das eine [Version von Windows 10 Insiders](https://insider.windows.com/) ausführt.   
- Dieser Schnellstart kann auf einem virtuellen Windows 10-Computer ausgeführt werden, doch die geschachtelte Virtualisierung muss aktiviert sein. Weitere Informationen finden Sie im [Handbuch „Geschachtelte Virtualisierung“](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

## 1. Installieren des Containerfeatures

Das Containerfeature muss aktiviert werden, bevor Sie mit Windows-Containern arbeiten können. Führen Sie dazu den folgenden Befehl in einer PowerShell-Sitzung mit erhöhten Rechten aus. 

```none
Enable-WindowsOptionalFeature -Online -FeatureName containers -All
```

Da Windows 10 nur Hyper-V-Container unterstützt, muss das Hyper-V-Feature ebenfalls aktiviert werden. Um das Hyper-V-Feature mithilfe von PowerShell zu aktivieren, führen Sie den folgenden Befehl in einer PowerShell-Sitzung mit erhöhten Rechten aus.

```none
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

Starten Sie den Computer neu, wenn die Installation abgeschlossen ist.

```none
Restart-Computer -Force
```

## 2. Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Für diese Übung werden beide installiert. Führen Sie dazu die folgenden Befehle aus. 

Erstellen Sie einen Ordner für die ausführbaren Docker-Dateien.

```none
New-Item -Type Directory -Path $env:ProgramFiles\docker\
```

Laden Sie den Docker-Daemon herunter.

```none
Invoke-WebRequest https://master.dockerproject.org/windows/amd64/dockerd.exe -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Laden Sie den Docker-Client herunter.

```none
Invoke-WebRequest https://master.dockerproject.org/windows/amd64/docker.exe -OutFile $env:ProgramFiles\docker\docker.exe
```

Fügen Sie das Docker-Verzeichnis dem Systempfad hinzu.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:ProgramFiles\docker\", [EnvironmentVariableTarget]::Machine)
```

Starten Sie die PowerShell-Sitzung neu, damit der geänderte Pfad erkannt wird.

Führen Sie den folgenden Befehl aus, um Docker als Windows-Dienst zu installieren.

```none
dockerd --register-service
```

Nach Abschluss der Installation kann der Dienst gestartet werden.

```none
Start-Service Docker
```

## 3. Installieren von Basiscontainerimages

Windows-Container werden in Vorlagen oder Images bereitgestellt. Bevor ein Container bereitgestellt werden kann, muss ein Basisimage des Betriebssystems für den Container heruntergeladen werden. Die folgenden Befehle laden das Nano Server-Basisiamge herunter.
    
> Dieses Verfahren gilt für Windows Insider-Builds größer als 14372 und vorübergehend so lange, bis „Docker Pull“ funktionsfähig ist.

Laden Sie das Nano Server-Basisimage herunter. 

```none
Start-BitsTransfer https://aka.ms/tp5/6b/docker/nanoserver -Destination nanoserver.tar.gz
```

Installieren Sie das Basisimage.

```none  
docker load -i nanoserver.tar.gz
```

In dieser Phase gibt die Ausführung von `docker images` eine Liste der installierten Images zurück, in diesem Fall das Nano Server-Image.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1030     3f5112ddd185        3 weeks ago         810.2 MB
```

Bevor Sie fortfahren, müssen Sie die Version dieses Images als „latest“ (neueste) kennzeichnen. Führen Sie dazu den folgenden Befehl aus:

```none
docker tag microsoft/nanoserver:10.0.14300.1030 nanoserver:latest
```

Ausführliche Informationen zu Windows-Containerimages finden Sie unter [Verwalten von Containerimages](../management/manage_images.md).

## 4. Bereitstellen Ihres ersten Containers

Für dieses einfache Beispiel wurde bereits ein .NET Core-Image erstellt. Laden Sie dieses Image mit dem `docker pull`-Befehl herunter.

Bei der Ausführung wird ein Container gestartet, die einfache .NET Core-Anwendung ausgeführt und dann der Container beendet. 

```none
docker pull microsoft/sample-dotnet
```

Dies kann mit dem `docker images`-Befehl überprüft werden.

```none
docker 

REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
microsoft/sample-dotnet  latest              28da49c3bff4        41 hours ago        918.3 MB
nanoserver               10.0.14300.1030     3f5112ddd185        3 weeks ago         810.2 MB
nanoserver               latest              3f5112ddd185        3 weeks ago         810.2 MB
```

Führen Sie den Container mit dem `docker run`-Befehl aus. Das folgende Beispiel legt den `--rm`-Parameter fest. Dies weist das Docker-Modul an, den Container zu löschen, sobald er nicht mehr ausgeführt wird. 

Weitere Informationen zum Befehl „Docker Run“ finden Sie in der [Referenz zu „Docker Run“ auf Docker.com]( https://docs.docker.com/engine/reference/run/).

```none
docker run --isolation=hyperv --rm microsoft/sample-dotnet
```

**Hinweis**: Wenn ein Fehler ausgelöst wird, der auf ein Zeitüberschreitungsereignis hinweist, führen Sie das folgende PowerShell-Skript aus, und wiederholen Sie den Vorgang.

```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

Das Ergebnis des Befehls `docker run` ist, dass ein Hyper-V-Container auf Basis des Beispiel-DotNet-Images erstellt wurde, danach eine Beispiel-App ausgeführt (Ausgabeecho über die Shell) und dann der Container beendet und entfernt wurde. Nachfolgende Windows 10- und Containerschnellstarts behandeln das Erstellen und Bereitstellen von Anwendungen in Containern unter Windows 10.

## Nächste Schritte

[Windows-Container unter Windows Server](./quick_start_windows_server.md)





<!--HONumber=Jul16_HO2-->


