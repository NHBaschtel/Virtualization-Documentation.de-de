---
title: "Windows-Container unter Windows 10"
description: "Containerbereitstellung – Schnellstart"
keywords: Docker, Container
author: neilpeterson
manager: timlt
ms.date: 08/17/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: fac57150de3ffd6c7d957dd628b937d5c41c1b35
ms.openlocfilehash: 57d35f9e871bdd3bd0798833bcbaf6a7948a65f2

---

# Windows-Container unter Windows 10

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

Die Übung führt durch die einfache Bereitstellung und Verwendung des Windows-Containerfeatures unter Windows 10 Professional oder Enterprise (Anniversary Edition). Nach der Ausführung haben Sie die Containerrolle installiert und einen einfachen Hyper-V-Container bereitgestellt. Machen Sie sich vor diesem Schnellstart mit grundlegenden Containerkonzepten und der Terminologie vertraut. Diese Informationen finden Sie unter [Windows Containers Quick Start](./quick_start.md) (Windows-Container – Schnellstart). 

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

Führen Sie den folgenden Befehl aus, sobald der Computer neu gestartet wurde, um ein bekanntes Problem mit der Technical Preview des Features „Windows-Container“ zu beheben.  

 ```none
Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 1 -Force
```

> In aktuellen Versionen müssen Sie OpLocks deaktivieren, um Hyper-V-Container zuverlässig verwenden zu können. Zum erneuten Aktivieren von OpLocks verwenden Sie den folgenden Befehl:  `Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 0 -Force`

## 2. Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Für diese Übung werden beide installiert. Führen Sie dazu die folgenden Befehle aus. 

Laden Sie das Docker-Modul und den Docker-Client als ZIP-Archiv herunter.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Erweitern Sie das ZIP-Archiv in „Programme“, die Archivinhalte befinden sich bereits im Docker-Verzeichnis.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Fügen Sie das Docker-Verzeichnis dem Systempfad hinzu.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:ProgramFiles\docker\", [EnvironmentVariableTarget]::Machine)
```

Starten Sie die PowerShell-Sitzung neu, damit der geänderte Pfad erkannt wird.

Führen Sie den folgenden Befehl aus, um Docker als Windows-Dienst zu installieren.

```none
& $env:ProgramFiles\docker\dockerd.exe --register-service
```

Nach Abschluss der Installation kann der Dienst gestartet werden.

```none
Start-Service Docker
```

## 3. Installieren von Basiscontainerimages

Windows-Container werden in Vorlagen oder Images bereitgestellt. Bevor ein Container bereitgestellt werden kann, muss ein Basisimage des Betriebssystems für den Container heruntergeladen werden. Die folgenden Befehle laden das Nano Server-Basisiamge herunter.

Rufen Sie das Nano Server-Basisimage per Pull ab. 

```none
docker pull microsoft/nanoserver
```

Nachdem das Image per Pull abgerufen wurde, gibt die Ausführung von `docker images` eine Liste der installierten Images zurück, in diesem Fall das Nano Server-Image.

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              3a703c6e97a2        7 weeks ago         969.8 MB
```

Ausführliche Informationen zu Windows-Containerimages finden Sie unter [Verwalten von Containerimages](../management/manage_images.md).

## 4. Bereitstellen Ihres ersten Containers

In diesem einfachen Beispiel wird ein „Hello World“-Containerimage erstellt und bereitgestellt. Am besten führen Sie diese Befehle in einer Windows-Befehlsshell mit erhöhten Rechten aus.

Starten Sie zuerst einen Container mit einer interaktiven Sitzung aus dem `nanoserver`-Image. Sobald der Container gestartet wurde, wird Ihnen eine Befehlsshell für den Inhalt des Containers angezeigt.  

```none
docker run -it nanoserver cmd
```

Erstellen Sie nun innerhalb des Containers ein einfaches „Hello World“-Skript.

```none
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

Wenn Sie den Vorgang abgeschlossen haben, beenden Sie den Container.

```none
exit
```

Erstellen Sie jetzt ein neues Containerimage aus dem geänderten Container. Führen Sie Folgendes aus, und notieren Sie sich die Container-ID, um eine Liste der Container anzuzeigen:

```none
docker ps -a
```

Führen Sie den folgenden Befehl aus, um ein neues „Hello World“-Image zu erstellen: Ersetzen Sie <containerid> durch die ID Ihres Containers.

```none
docker commit <containerid> helloworld
```

Nach Beendigung des Vorgangs verfügen Sie über ein benutzerdefiniertes Image, das ein „Hello World“-Skript enthält. Sie können es mit dem folgenden Befehl anzeigen:

```none
docker images
```

Verwenden Sie abschließend den Befehl `docker run`, um den Container auszuführen.

```none
docker run --rm helloworld powershell c:\helloworld.ps1
```

Das Ergebnis des Befehls `docker run` ist, dass ein Hyper-V-Container auf Basis des „Hello World“-Images erstellt wurde, danach ein „Hello World“-Beispielskript ausgeführt (Ausgabeecho über die Shell) und anschließend der Container beendet und entfernt wurde. Nachfolgende Windows 10- und Containerschnellstarts behandeln das Erstellen und Bereitstellen von Anwendungen in Containern unter Windows 10.

## Nächste Schritte

[Windows-Container unter Windows Server](./quick_start_windows_server.md)





<!--HONumber=Aug16_HO3-->


