---
title: Windows-Container unter Windows10
description: "Containerbereitstellung – Schnellstart"
keywords: Docker, Container
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 66bb1ba56fde13d76392ddb4a1f8e6855201e7a3
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/21/2017
---
# Windows-Container unter Windows10

Die Übung führt durch die einfache Bereitstellung und Verwendung des Windows-Containerfeatures unter Windows 10 Professional oder Enterprise (Anniversary Edition). Nach Abschluss des Vorgangs haben Sie Docker für Windows installiert und einen einfachen Container ausgeführt. Machen Sie sich vor diesem Schnellstart mit grundlegenden Containerkonzepten und der Terminologie vertraut. Diese Informationen finden Sie unter [Windows Containers Quick Start](./index.md) (Windows-Container – Schnellstart).

Dieser Schnellstart bezieht sich speziell auf Windows10. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis.

***Hyper-V-Isolation:*** Windows Server-Container erfordert Hyper-V-Isolation auf Windows10, damit Entwickler die gleiche Kernel-Version und -Konfiguration nutzen können, die in der Produktion verwendet werden. Weitere Informationen hierzu finden Sie auf der Seite [Informationen zu Windows-Containern](../about/index.md).

**Voraussetzungen:**

- Ein physisches Computersystem mit Windows 10 Anniversary Edition oder Creators Update (Professional oder Enterprise).   
- Dieser Schnellstart kann auf einem virtuellen Windows10-Computer ausgeführt werden, doch die geschachtelte Virtualisierung muss aktiviert sein. Weitere Informationen finden Sie im [Handbuch „Geschachtelte Virtualisierung“](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

> Sie müssen wichtige Updates installieren, damit Windows-Container funktionieren.
> Zum Überprüfen Ihrer Betriebssystemversion, führen Sie `winver.exe` aus, und vergleichen Sie die angezeigte Version mit dem [Windows 10-Updateverlauf](https://support.microsoft.com/en-us/help/12387/windows-10-update-history).
> Stellen Sie sicher, dass Sie über die Version 14393.222 oder höher verfügen, bevor Sie fortfahren.

## 1. Installieren von Docker für Windows

[Laden Sie Docker für Windows herunter](https://download.docker.com/win/stable/InstallDocker.msi) und führen Sie das Installationsprogramm aus. [Ausführliche Informationen zur Installation](https://docs.docker.com/docker-for-windows/install) finden Sie in der Dokumentation zu Docker.

## 2. Wechseln Sie zu Windows Containern

Nach der Installation führt Docker für Windows standardmäßig Linux-Container aus. Wechseln Sie zu mithilfe des Docker-Menüs in der Taskleiste oder durch Ausführen des folgenden Befehls in einer PowerShell-Aufforderung zu Windows-Containern `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon`.

![](./media/docker-for-win-switch.png)

## 3. Installieren von Basiscontainerimages

Windows-Container werden aus Basisimages erstellt. Der folgende Befehl ruft das Nano Server-Basisimage ab.

```none
docker pull microsoft/nanoserver
```

Nachdem das Image per Pull abgerufen wurde, gibt die Ausführung von `docker images` eine Liste der installierten Images zurück, in diesem Fall das Nano Server-Image.

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Bitte lesen Sie den [Endbenutzer-Lizenzvertrag](../images-eula.md) zum Betriebssystemimage für Windows-Container.

## 4. Ausführen Ihres ersten Containers

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
Nachfolgende Windows10- und Containerschnellstarts behandeln das Erstellen und Bereitstellen von Anwendungen in Containern unter Windows10.

## Nächste Schritte

[Windows-Container unter Windows Server](./quick-start-windows-server.md)
