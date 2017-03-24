---
title: "Windows-Container unter Windows 10"
description: "Containerbereitstellung – Schnellstart"
keywords: Docker, Container
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 996d3b1a8f7c8325ac66d331e1d62208c0cf6b53
ms.openlocfilehash: 091a3570291624a3be40e3aabb9f99a482cb6470
ms.lasthandoff: 02/27/2017

---

# Windows-Container unter Windows 10

Die Übung führt durch die einfache Bereitstellung und Verwendung des Windows-Containerfeatures unter Windows 10 Professional oder Enterprise (Anniversary Edition). Nach der Ausführung haben Sie die Containerrolle installiert und einen einfachen Hyper-V-Container bereitgestellt. Machen Sie sich vor diesem Schnellstart mit grundlegenden Containerkonzepten und der Terminologie vertraut. Diese Informationen finden Sie unter [Windows Containers Quick Start](./index.md) (Windows-Container – Schnellstart).

Dieser Schnellstart ist spezifisch für Hyper-V-Container unter Windows 10. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis.

**Voraussetzungen:**

- Ein physisches Computersystem mit Windows 10 Anniversary Edition (Professional oder Enterprise).   
- Dieser Schnellstart kann auf einem virtuellen Windows 10-Computer ausgeführt werden, doch die geschachtelte Virtualisierung muss aktiviert sein. Weitere Informationen finden Sie im [Handbuch „Geschachtelte Virtualisierung“](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

> Sie müssen wichtige Updates installieren, damit Windows-Container funktionieren. 
> Zum Überprüfen Ihrer Betriebssystemversion, führen Sie `winver.exe` aus, und vergleichen Sie die angezeigte Version mit dem [Windows 10-Updateverlauf](https://support.microsoft.com/en-us/help/12387/windows-10-update-history). 
> Stellen Sie sicher, dass Sie über die Version 14393.222 oder höher verfügen, bevor Sie fortfahren.

## 1. Installieren des Containerfeatures

Das Containerfeature muss aktiviert werden, bevor Sie mit Windows-Containern arbeiten können. Führen Sie dazu den folgenden Befehl in einer PowerShell-Sitzung mit **erhöhten Rechten** aus.

Wenn eine Fehlermeldung anzeigt, dass `Enable-WindowsOptionalFeature` nicht vorhanden ist, sollten Sie noch einmal überprüfen, ob Sie PowerShell als Administrator ausführen.

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

> Wenn Sie die aus der Technical Preview 5 stammenden Basisimages für Container bereits für Hyper-V-Container in Windows 10 verwendet haben, dürfen Sie nicht vergessen, OpLocks erneut zu aktivieren. Führen Sie den folgenden Befehl aus:  `Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 0 -Force`

## 2. Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Für diese Übung werden beide installiert. Führen Sie dazu die folgenden Befehle aus.

Laden Sie das Docker-Modul und den Docker-Client als ZIP-Archiv herunter.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-17.03.0-ce.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Erweitern Sie das ZIP-Archiv in „Programme“, die Archivinhalte befinden sich bereits im Docker-Verzeichnis.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Fügen Sie das Docker-Verzeichnis dem Systempfad hinzu.

```none
# Add path to this PowerShell session immediately
$env:path += ";$env:ProgramFiles\Docker"

# For persistent use after a reboot
$existingMachinePath = [Environment]::GetEnvironmentVariable("Path",[System.EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("Path", $existingMachinePath + ";$env:ProgramFiles\Docker", [EnvironmentVariableTarget]::Machine)
```

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

Rufen Sie das Nano Server-Basisimage per Pull ab.

```none
docker pull microsoft/nanoserver
```

Nachdem das Image per Pull abgerufen wurde, gibt die Ausführung von `docker images` eine Liste der installierten Images zurück, in diesem Fall das Nano Server-Image.

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Bitte lesen Sie die [Lizenzbedingungen](../images-eula.md) zum Betriebssystemimage für Windows-Container.

## 4. Bereitstellen Ihres ersten Containers

In diesem einfachen Beispiel wird ein „Hello World“-Containerimage erstellt und bereitgestellt. Am besten führen Sie diese Befehle in einer Windows-Eingabeaufforderung mit erhöhten Rechten oder mit Windows PowerShell aus.

> Windows PowerShell ISE funktioniert nicht für interaktive Sitzungen mit Containern. Auch wenn der Container ausgeführt wird, scheint die Ausführung angehalten zu sein.

Starten Sie zuerst einen Container mit einer interaktiven Sitzung aus dem `nanoserver`-Image. Sobald der Container gestartet wurde, wird Ihnen eine Befehlsshell für den Inhalt des Containers angezeigt.  

```none
docker run -it microsoft/nanoserver cmd
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

Das Ergebnis des Befehls `docker run` ist, dass ein Hyper-V-Container auf Basis des „Hello World“-Images erstellt wurde, danach ein „Hello World“-Beispielskript ausgeführt (Ausgabeecho über die Shell) und anschließend der Container beendet und entfernt wurde.
Nachfolgende Windows 10- und Containerschnellstarts behandeln das Erstellen und Bereitstellen von Anwendungen in Containern unter Windows 10.

## Nächste Schritte

[Windows-Container unter Windows Server](./quick-start-windows-server.md)

