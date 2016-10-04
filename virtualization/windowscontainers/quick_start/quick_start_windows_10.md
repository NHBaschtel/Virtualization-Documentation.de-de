---
title: "Windows-Container unter Windows 10"
description: "Containerbereitstellung – Schnellstart"
keywords: Docker, Container
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
translationtype: Human Translation
ms.sourcegitcommit: 0fae34a5a85678a25c47b0312650e67aa6cd7efd
ms.openlocfilehash: 74686f222e8eec1daacd45a9e388d94abf6381f4

---

# Windows-Container unter Windows 10

Die Übung führt durch die einfache Bereitstellung und Verwendung des Windows-Containerfeatures unter Windows 10 Professional oder Enterprise (Anniversary Edition). Nach der Ausführung haben Sie die Containerrolle installiert und einen einfachen Hyper-V-Container bereitgestellt. Machen Sie sich vor diesem Schnellstart mit grundlegenden Containerkonzepten und der Terminologie vertraut. Diese Informationen finden Sie unter [Windows Containers Quick Start](./quick_start.md) (Windows-Container – Schnellstart).

Dieser Schnellstart ist spezifisch für Hyper-V-Container unter Windows 10. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis.

**Voraussetzungen:**

- Ein physisches Computersystem mit Windows 10 Anniversary Edition (Professional oder Enterprise).   
- Dieser Schnellstart kann auf einem virtuellen Windows 10-Computer ausgeführt werden, doch die geschachtelte Virtualisierung muss aktiviert sein. Weitere Informationen finden Sie im [Handbuch „Geschachtelte Virtualisierung“](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

> Wichtige Updates sind erforderlich, damit das Feature „Windows-Container“ funktioniert. Installieren Sie alle Updates, bevor Sie dieses Tutorial durcharbeiten.

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

> Wenn Sie die aus der Technical Preview 5 stammenden Basisimages für Container bereits für Hyper-V-Container in Windows 10 verwendet haben, dürfen Sie nicht vergessen, OpLocks erneut zu aktivieren. Führen Sie den folgenden Befehl aus:  `Set-ItemProperty -Path 'HKLM:SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers' -Name VSmbDisableOplocks -Type DWord -Value 0 -Force`

## 2. Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Für diese Übung werden beide installiert. Führen Sie dazu die folgenden Befehle aus.

Laden Sie das Docker-Modul und den Docker-Client als ZIP-Archiv herunter.

```none
Invoke-WebRequest "https://master.dockerproject.org/windows/amd64/docker-1.13.0-dev.zip" -OutFile "$env:TEMP\docker-1.13.0-dev.zip" -UseBasicParsing
```

Erweitern Sie das ZIP-Archiv in „Programme“, die Archivinhalte befinden sich bereits im Docker-Verzeichnis.

```none
Expand-Archive -Path "$env:TEMP\docker-1.13.0-dev.zip" -DestinationPath $env:ProgramFiles
```

Fügen Sie das Docker-Verzeichnis dem Systempfad hinzu.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot.
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
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

Ausführliche Informationen zu Windows-Containerimages finden Sie unter [Verwalten von Containerimages](../management/manage_images.md).

> Bitte lesen Sie sich die [Lizenzbedingungen](../Images_EULA.md) zum Betriebssystemimage für Windows-Container durch.

## 4. Bereitstellen Ihres ersten Containers

In diesem einfachen Beispiel wird ein „Hello World“-Containerimage erstellt und bereitgestellt. Am besten führen Sie diese Befehle in einer Windows-Befehlsshell mit erhöhten Rechten aus.

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

[Windows-Container unter Windows Server](./quick_start_windows_server.md)



<!--HONumber=Sep16_HO5-->


