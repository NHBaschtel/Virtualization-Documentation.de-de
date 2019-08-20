---
title: Windows-und Linux-Container unter Windows 10
description: Containerbereitstellung – Schnellstart
keywords: docker, Container, LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: aa7a20d914fdb65597c0f31ef6d53b91f6497be4
ms.sourcegitcommit: 2f8fd4b2e7113fbb7c323d89f3c72df5e1a4437e
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/20/2019
ms.locfileid: "10044960"
---
# <a name="windows-containers-on-windows-10"></a>Windows-Container unter Windows10

Die Übung führt Sie durch das Erstellen und Ausführen von Windows-Containern unter Windows 10.

In diesem Schnellstart können Sie Folgendes ausführen:

1. Installieren des docker-Desktops
2. Ausführen eines einfachen Windows-Containers

Dieser Schnellstart bezieht sich speziell auf Windows10. Weitere Schnellstart Dokumentation finden Sie im Inhaltsverzeichnis auf der linken Seite dieser Seite.

## <a name="prerequisites"></a>Voraussetzungen
Stellen Sie bitte sicher, dass Sie die folgenden Voraussetzungen erfüllen:
- Ein physisches Computersystem, auf dem Windows 10 Professional oder Enterprise mit Anniversary Update (Version 1607) oder höher ausgeführt wird. 
- Stellen Sie sicher, dass [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) aktiviert ist.

***Hyper-V-Isolierung:*** Windows Server-Container erfordern Hyper-v-Isolierung unter Windows 10, um Entwicklern die gleiche Kernel Version und-Konfiguration zur Verfügung zu stellen, die in der Produktion verwendet wird, weitere Informationen zur Hyper-v-Isolierung finden Sie auf der [Container Seite zu Windows](../about/index.md) .

> [!NOTE]
> In der Veröffentlichung von Windows October Update 2018 verbieten wir Benutzern nicht mehr die Ausführung eines Windows-Containers im Prozess Isolationsmodus unter Windows 10 Enterprise oder Professional für dev/Test-Zwecke. Weitere Informationen finden Sie in den [FAQ](../about/faq.md) .

## <a name="install-docker-desktop"></a>Installieren des docker-Desktops

Laden Sie den docker- [Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) herunter, und führen Sie das Installationsprogramm aus (Sie müssen sich anmelden. Erstellen Sie ein Konto, wenn Sie noch keines haben. [Ausführliche Informationen zur Installation](https://docs.docker.com/docker-for-windows/install) finden Sie in der Dokumentation zu Docker.

## <a name="switch-to-windows-containers"></a>Wechseln zu Windows-Containern

Nach der Installation wird der Docker-Desktop standardmäßig mit Linux-Containern ausgeführt. Wechseln Sie zu Windows-Containern entweder über das Docking-Tray-Menü oder durch Ausführen des folgenden Befehls in einer PowerShell-Eingabeaufforderung:

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

## <a name="install-base-container-images"></a>Installieren von Basiscontainerimages

Windows-Container werden aus Basisimages erstellt. Der folgende Befehl ruft das Nano Server-Basisimage ab.

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

> [!NOTE]
> Wenn eine Fehlermeldung angezeigt wird, die `no matching manifest for unknown in the manifest list entries`besagt, stellen Sie sicher, dass Sie nicht erwarten, dass Sie einen Linux-Container ziehen.

Nachdem das Image per Pull abgerufen wurde, gibt die Ausführung von `docker images` eine Liste der installierten Images zurück, in diesem Fall das Nano Server-Image.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Bitte lesen Sie den Windows Containers OS-Bild- [Lizenzvertrag](../images-eula.md).

## <a name="run-your-first-windows-container"></a>Ausführen des ersten Windows-Containers

In diesem einfachen Beispiel wird ein Container Bild "Hello World" erstellt und bereitgestellt. Am besten führen Sie diese Befehle in einer Windows-Eingabeaufforderung mit erhöhten Rechten oder mit Windows PowerShell aus.

> Windows PowerShell ISE funktioniert nicht für interaktive Sitzungen mit Containern. Auch wenn der Container ausgeführt wird, scheint die Ausführung angehalten zu sein.

Starten Sie zuerst einen Container mit einer interaktiven Sitzung aus dem `nanoserver`-Image. Sobald der Container gestartet wurde, wird Ihnen eine Befehlsshell für den Inhalt des Containers angezeigt.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

Im Container wird eine einfache Textdatei "Hello World" erstellt.

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

Das Ergebnis des `docker run` Befehls besteht darin, dass ein Container, der unter Hyper-V-Isolierung ausgeführt wird, aus dem "HelloWorld"-Bild erstellt wurde, eine Instanz von cmd im Container gestartet und eine Lesung unserer Datei ausgeführt wurde (die Ausgabe wurde in die Shell wiedergegeben) und dann der Container gestoppt und entfernt.

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Informationen zum Erstellen einer Beispiel-App](./building-sample-app.md)
