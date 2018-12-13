---
title: Windows und Linux-Container unter Windows 10
description: Containerbereitstellung – Schnellstart
keywords: Docker, Container, LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 4202908f8797a2b98ab657c45cd9a6b33191bd6f
ms.sourcegitcommit: 9dfef8d261f4650f47e8137a029e893ed4433a86
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/09/2018
ms.locfileid: "6224899"
---
# <a name="windows-and-linux-containers-on-windows-10"></a>Windows und Linux-Container unter Windows 10

Die Übung führt durch Erstellen und Ausführen von Windows und Linux-Container unter Windows 10. Nach Abschluss des Vorgangs haben Sie:

1. Installierte Docker für Windows
2. Führen Sie einen einfachen Windows-container
3. Führen Sie einen einfachen Linux-Container mit Linux-Container für Windows (LCOW)

Dieser Schnellstart bezieht sich speziell auf Windows10. Weitere Schnellstartdokumentation finden Sie im Inhaltsverzeichnis auf der linken Seite der Seite.

***Hyper-V-Isolation:*** Windows Server-Container erfordert Hyper-V-Isolation auf Windows 10, damit Entwickler die gleiche Kernel-Version und Konfiguration, die in der Produktion verwendet werden, Weitere Informationen zu Hyper-V-Isolation finden Sie auf der Seite " [Informationen zu Windows-Container](../about/index.md) ".

**Voraussetzungen:**

- Ein physisches Computersystem mit Windows 10 Fall Creators Update (Version 1709) oder höher (Professional oder Enterprise), [Hyper-V ausführen können](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)

> Wenn Sie nicht zum Ausführen von LCOW in diesem Lernprogramm erfahren möchten, werden Windows-Container unter Windows 10 Anniversary Update (Version 1607) oder höher ausgeführt werden.

## <a name="1-install-docker-for-windows"></a>1. Installieren von Docker für Windows

[Herunterladen von Docker für Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) und führen Sie das Installationsprogramm (Anmeldung erforderlich werden. Erstellen Sie ein Konto, wenn Sie nicht bereits eine haben). [Ausführliche Informationen zur Installation](https://docs.docker.com/docker-for-windows/install) finden Sie in der Dokumentation zu Docker.

> Wenn Sie bereits Docker installiert haben, stellen Sie sicher, dass Sie Version 18.02 oder höher unterstützen LCOW haben. Überprüfen, indem Sie mit `docker -v` oder *Docker zu*überprüfen.

> Die experimentelle Features-Option im *Docker-Einstellungen > Daemon* muss aktiviert sein, um LCOW-Container ausgeführt.

## <a name="2-switch-to-windows-containers"></a>2. Wechseln Sie zu Windows Containern

Nach der Installation führt Docker für Windows standardmäßig Linux-Container aus. Wechseln Sie zu mithilfe des Docker-Menüs in der Taskleiste oder durch Ausführen des folgenden Befehls in einer PowerShell-Aufforderung zu Windows-Containern `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon`.

![](./media/docker-for-win-switch.png)
> Beachten Sie, dass Windows-Container-Modus für LCOW Container zusätzlich zu Windows-Container ermöglicht.

## <a name="3-install-base-container-images"></a>3. Installieren von Basiscontainerimages

Windows-Container werden aus Basisimages erstellt. Der folgende Befehl ruft das Nano Server-Basisimage ab.

```
docker pull microsoft/nanoserver
```

Nachdem das Image per Pull abgerufen wurde, gibt die Ausführung von `docker images` eine Liste der installierten Images zurück, in diesem Fall das Nano Server-Image.

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Bitte lesen Sie den [Endbenutzer-Lizenzvertrag](../images-eula.md) zum Betriebssystemimage für Windows-Container.

## <a name="4-run-your-first-windows-container"></a>4. Führen Sie Ihres ersten Windows-Containers aus

In diesem einfachen Beispiel wird ein „Hello World“-Containerimage erstellt und bereitgestellt. Am besten führen Sie diese Befehle in einer Windows-Eingabeaufforderung mit erhöhten Rechten oder mit Windows PowerShell aus.
> Windows PowerShell ISE funktioniert nicht für interaktive Sitzungen mit Containern. Auch wenn der Container ausgeführt wird, scheint die Ausführung angehalten zu sein.

Starten Sie zuerst einen Container mit einer interaktiven Sitzung aus dem `nanoserver`-Image. Sobald der Container gestartet wurde, wird Ihnen eine Befehlsshell für den Inhalt des Containers angezeigt.  

```
docker run -it microsoft/nanoserver cmd
```

Erstellen Sie nun innerhalb des Containers ein einfaches „Hello World“-Skript.

```
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

Wenn Sie den Vorgang abgeschlossen haben, beenden Sie den Container.

```
exit
```

Erstellen Sie jetzt ein neues Containerimage aus dem geänderten Container. Führen Sie Folgendes aus, und notieren Sie sich die Container-ID, um eine Liste der Container anzuzeigen:

```
docker ps -a
```

Führen Sie den folgenden Befehl aus, um ein neues „Hello World“-Image zu erstellen: Ersetzen Sie <containerid> durch die ID Ihres Containers.

```
docker commit <containerid> helloworld
```

Nach Beendigung des Vorgangs verfügen Sie über ein benutzerdefiniertes Image, das ein „Hello World“-Skript enthält. Sie können es mit dem folgenden Befehl anzeigen:

```
docker images
```

Verwenden Sie abschließend den Befehl `docker run`, um den Container auszuführen.

```
docker run --rm helloworld powershell c:\helloworld.ps1
```

Das Ergebnis des Befehls `docker run` ist, dass ein Hyper-V-Container auf Basis des „Hello World“-Images erstellt wurde, danach ein „Hello World“-Beispielskript ausgeführt (Ausgabeecho über die Shell) und anschließend der Container beendet und entfernt wurde.
Nachfolgende Windows10- und Containerschnellstarts behandeln das Erstellen und Bereitstellen von Anwendungen in Containern unter Windows10.

## <a name="run-your-first-lcow-container"></a>Führen Sie Ihres ersten Containers LCOW aus

In diesem Beispiel wird ein BusyBox-Container bereitgestellt werden. Zunächst versuchen Sie, ein "Hello World" BusyBox Bild auszuführen.

```
docker run --rm busybox echo hello_world
```

Beachten Sie, dass dies ein Fehler zurückgegeben, wenn Docker versucht, um das Bild abzurufen. Dies tritt auf, da Dockers erfordert eine Richtlinie über die `--platform` Kennzeichen, um sicherzustellen, dass das Bild und die Host-Betriebssystem werden entsprechend verglichen. Da die Standard-Plattform in Windows-Container-Modus Windows ist, Hinzufügen einer `--platform linux` Flag per Pull abrufen und den Container auszuführen.

```
docker run --rm --platform linux busybox echo hello_world
```

Nachdem das Bild mit der Plattform angegeben, zieht die `--platform` Flag ist nicht mehr erforderlich. Führen Sie den Befehl ohne, um dies zu testen.

```
docker run --rm busybox echo hello_world
```

Führen Sie `docker images` um eine Liste der installierten Images zurückzugeben. In diesem Fall die Windows und Linux-Bilder.

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>Nächste Schritte

Bonus: Finden Sie unter der Docker entsprechenden [Blogbeitrag](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/) zum Ausführen von LCOW

Fahren Sie mit dem nächsten Lernprogramm fort, und erhalten Sie ein Beispiel zum [Erstellen einer Beispiel-App](./building-sample-app.md).
