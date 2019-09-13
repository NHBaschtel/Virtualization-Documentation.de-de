---
title: Windows-und Linux-Container unter Windows 10
description: Containerbereitstellung – Schnellstart
keywords: docker, Container, LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 3d651a4a68acefa25f1b647b1b33618bbfb91ae9
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129370"
---
# <a name="get-started-run-your-first-container"></a>Erste Schritte: Ausführen des ersten Containers

Im [vorherigen Segment](./set-up-environment.md)haben wir unsere Umgebung für die Ausführung von Containern konfiguriert. In dieser Übung wird gezeigt, wie Sie ein Container Bild ziehen und ausführen.

## <a name="install-container-base-image"></a>Container-Basisabbild installieren

Alle Container werden aus `container images`instanziiert. Microsoft bietet mehrere "Starter"-Bilder ( `base images`genannt) zur Auswahl an. Der folgende Befehl ruft das Nano Server-Basisimage ab.

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

> [!TIP]
> Wenn eine Fehlermeldung angezeigt wird, die `no matching manifest for unknown in the manifest list entries`besagt, stellen Sie sicher, dass docker nicht für die Ausführung von Linux-Containern konfiguriert ist.

Nachdem das Bild gezogen wurde, können Sie es auf Ihrem Computer überprüfen, indem Sie Ihr lokales docker-Bild-Repository Abfragen. Durch Ausführen des `docker images` Befehls wird eine Liste der installierten Bilder zurückgegeben – in diesem Fall das Nano-Server-Image.

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Bitte lesen Sie den Windows Containers OS-Bild- [Lizenzvertrag](../images-eula.md).

## <a name="run-your-first-windows-container"></a>Ausführen des ersten Windows-Containers

In diesem einfachen Beispiel wird ein Container Bild "Hello World" erstellt und bereitgestellt. Führen Sie die folgenden Befehle in einer erhöhten Windows CMD-Shell oder PowerShell aus, um optimale Ergebnisse zu bieten.

> Windows PowerShell ISE funktioniert nicht für interaktive Sitzungen mit Containern. Auch wenn der Container ausgeführt wird, scheint die Ausführung angehalten zu sein.

Starten Sie zuerst einen Container mit einer interaktiven Sitzung aus dem `nanoserver`-Image. Nachdem der Container gestartet wurde, wird eine Befehlsshell im Container angezeigt.  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

Im Container erstellen wir eine einfache Textdatei "Hello World".

```cmd
echo "Hello World!" > Hello.txt
```   

Wenn Sie den Vorgang abgeschlossen haben, beenden Sie den Container.

```cmd
exit
```

Erstellen Sie ein neues Container Bild aus dem geänderten Container. Führen Sie die folgenden Schritte aus, um eine Liste der Container anzuzeigen, die ausgeführt werden oder beendet wurden, und notieren Sie sich die Container-ID.

```console
docker ps -a
```

Führen Sie den folgenden Befehl aus, um ein neues „Hello World“-Image zu erstellen: Ersetzen Sie `<containerid>` durch die ID Ihres Containers.

```console
docker commit <containerid> helloworld
```

Nach Beendigung des Vorgangs verfügen Sie über ein benutzerdefiniertes Image, das ein „Hello World“-Skript enthält. Sie können es mit dem folgenden Befehl anzeigen:

```console
docker images
```

Führen Sie schließlich den Container mithilfe des `docker run` Befehls aus.

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

Das Ergebnis des `docker run` Befehls besteht darin, dass ein Container aus dem "HelloWorld"-Bild erstellt wurde, eine Instanz von cmd im Container gestartet wurde und eine Lesung unserer Datei ausgeführt wurde (die Ausgabe wurde in die Shell wiedergegeben), und dann wurde der Container angehalten und entfernt.

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Informationen zum containerisieren einer Beispiel-App](./building-sample-app.md)
