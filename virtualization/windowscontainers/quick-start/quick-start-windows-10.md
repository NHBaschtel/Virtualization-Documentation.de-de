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
ms.openlocfilehash: 07f5929505226a50a161b4ae7df5669c2ad89d83
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380424"
---
# <a name="windows-containers-on-windows-10"></a>Windows-Container unter Windows10

> [!div class="op_single_selector"]
> - [Linux-Container unter Windows](quick-start-windows-10-linux.md)
> - [Windows-Container unter Windows](quick-start-windows-10.md)

Die Übung führt durch Erstellen und Ausführen von Windows-Container unter Windows 10.

In diesem Schnellstart führen Sie Aktionen aus:

1. Installieren von Docker für Windows
2. Ausführen eines einfachen Windows Containers

Dieser Schnellstart bezieht sich speziell auf Windows10. Weitere Schnellstartdokumentation finden Sie in der Tabelle der Inhalt auf der linken Seite der Seite.

## <a name="prerequisites"></a>Voraussetzungen
Stellen Sie sicher, dass Sie die folgenden Anforderungen erfüllen:
- Ein physisches Computersystem mit Windows 10 Professional oder Enterprise mit Anniversary Update (Version 1607) oder höher. 
- Stellen Sie sicher, dass [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) aktiviert ist.

***Hyper-V-Isolation:*** Windows Server-Container erfordert Hyper-V-Isolation auf Windows 10 damit Entwickler die gleiche Kernel-Version und Konfiguration, die in der Produktion verwendet werden soll, mehr Informationen zu Hyper-V-Isolation finden Sie auf der Seite " [Informationen zu Windows-Container](../about/index.md) ".

> [!NOTE]
> In der Version von Windows Oktober Update 2018 verbieten wir nicht mehr Benutzer aus einen Windows-Container in Windows 10 Enterprise oder Professional Testzwecken Dev/im Prozess-Isolation ausgeführt. Finden Sie die [häufig gestellte Fragen](../about/faq.md) , um mehr zu erfahren.

## <a name="install-docker-for-windows"></a>Installieren von Docker für Windows

Herunterladen von [Docker für Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows) und führen Sie das Installationsprogramm (Sie müssen sich anmelden. Erstellen Sie ein Konto, wenn Sie nicht bereits eine haben). [Ausführliche Informationen zur Installation](https://docs.docker.com/docker-for-windows/install) finden Sie in der Dokumentation zu Docker.

## <a name="switch-to-windows-containers"></a>Wechseln Sie zu Windows Containern

Nach der Installation führt Docker für Windows standardmäßig Linux-Container aus. Wechseln Sie zu Windows-Container, die mithilfe des Docker-Menüs oder mithilfe des folgenden Befehls in einer PowerShell-Aufforderung:

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

## <a name="install-base-container-images"></a>Installieren von Basiscontainerimages

Windows-Container werden aus Basisimages erstellt. Der folgende Befehl ruft das Nano Server-Basisimage ab.

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

Nachdem das Image per Pull abgerufen wurde, gibt die Ausführung von `docker images` eine Liste der installierten Images zurück, in diesem Fall das Nano Server-Image.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Bitte lesen Sie die Windows-Container OS Image [EULA](../images-eula.md).

## <a name="run-your-first-windows-container"></a>Führen Sie Ihre ersten Windows-Container

In diesem einfachen Beispiel wird ein "Hello World"-containerimage erstellt und bereitgestellt werden. Am besten führen Sie diese Befehle in einer Windows-Eingabeaufforderung mit erhöhten Rechten oder mit Windows PowerShell aus.

> Windows PowerShell ISE funktioniert nicht für interaktive Sitzungen mit Containern. Auch wenn der Container ausgeführt wird, scheint die Ausführung angehalten zu sein.

Starten Sie zuerst einen Container mit einer interaktiven Sitzung aus dem `nanoserver`-Image. Sobald der Container gestartet wurde, wird Ihnen eine Befehlsshell für den Inhalt des Containers angezeigt.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

Innerhalb des Containers erstellen wir eine einfache Textdatei von "Hello World".

```cmd
echo "Hello World!" > Hello.txt
```   

Wenn Sie den Vorgang abgeschlossen haben, beenden Sie den Container.

```cmd
exit
```

Erstellen Sie jetzt ein neues Containerimage aus dem geänderten Container. Führen Sie Folgendes aus, und notieren Sie sich die Container-ID, um eine Liste der Container anzuzeigen:

```console
docker ps -a
```

Führen Sie den folgenden Befehl aus, um ein neues „Hello World“-Image zu erstellen: Ersetzen Sie <containerid> durch die ID Ihres Containers.

```console
docker commit <containerid> helloworld
```

Nach Beendigung des Vorgangs verfügen Sie über ein benutzerdefiniertes Image, das ein „Hello World“-Skript enthält. Sie können es mit dem folgenden Befehl anzeigen:

```console
docker images
```

Verwenden Sie abschließend den Befehl `docker run`, um den Container auszuführen.

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

Das Ergebnis des der `docker run` Befehl ist, dass ein Container unter Hyper-V-Isolierung Ausführen von "-Image erstellt wurde, eine Instanz der Cmd im Container gestartet wurde, und einen Wert liefern unserer Datei (ausgabeecho der Shell) und dann den Container ausgeführt beendet und entfernt wurde.

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Hier erfahren Sie, wie Sie eine Beispiel-app erstellen](./building-sample-app.md)