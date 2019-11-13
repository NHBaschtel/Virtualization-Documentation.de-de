---
title: Windows-und Linux-Container unter Windows 10
description: Containerbereitstellung – Schnellstart
keywords: docker, Container, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a664b5b8eb87adffdf7eba3ffca9f4194128df80
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288128"
---
# <a name="get-started-run-your-first-windows-container"></a>Erste Schritte: Ausführen des ersten Windows-Containers

In diesem Thema wird beschrieben, wie Sie Ihren ersten Windows-Container nach dem Einrichten Ihrer Umgebung ausführen, wie unter [Erste Schritte: Vorbereiten von Windows für Container](./set-up-environment.md)beschrieben. Zum Ausführen eines Containers installieren Sie zuerst ein Basisabbild, das eine grundlegende Schicht von Betriebssystemdiensten für ihren Container bereitstellt. Anschließend erstellen und führen Sie ein Container Bild aus, das auf dem Basis Bild basiert. Weitere Informationen finden Sie unter.

## <a name="install-a-container-base-image"></a>Installieren eines Containerbasis Bilds

Alle Container werden aus Container Bildern erstellt. Microsoft bietet mehrere Starter Bilder, so genannte Basisbilder, zur Auswahl an (Weitere Informationen finden Sie unter [Container Basisbilder](../manage-containers/container-base-images.md)). Mit diesen Verfahren wird das Lightweight Nano Server-Basisabbild heruntergeladen und installiert.

1. Öffnen Sie ein Eingabeaufforderungsfenster (wie die integrierte Eingabeaufforderung, PowerShell oder das [Windows-Terminal](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)), und führen Sie dann den folgenden Befehl aus, um das Basisabbild herunterzuladen und zu installieren:

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > Wenn eine Fehlermeldung angezeigt wird, die `no matching manifest for unknown in the manifest list entries`besagt, stellen Sie sicher, dass docker nicht für die Ausführung von Linux-Containern konfiguriert ist.

2. Nachdem das Bild heruntergeladen wurde, lesen Sie das [EULA](../images-eula.md) , während Sie warten – überprüfen Sie, ob es auf Ihrem System vorhanden ist, indem Sie Ihr lokales docker-Abbild-Repository Abfragen. Durch Ausführen des `docker images` Befehls wird eine Liste der installierten Bilder zurückgegeben.

   Hier ist ein Beispiel für die Ausgabe, die das Nano-Server Bild zeigt.

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Ausführen eines Windows-Containers

In diesem einfachen Beispiel wird ein Container Bild "Hello World" erstellt und bereitgestellt. Führen Sie die folgenden Befehle in einem erweiterten Eingabeaufforderungsfenster aus, um die optimale Benutzerfreundlichkeit zu gewährleisten (aber verwenden Sie nicht die Windows PowerShell-ISE – es funktioniert nicht für interaktive Sitzungen mit Containern, da die Container anscheinend hängen).

1. Starten Sie einen Container mit einer interaktiven Sitzung aus `nanoserver` dem Bild, indem Sie den folgenden Befehl in das Eingabeaufforderungsfenster eingeben:

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. Nachdem der Container gestartet wurde, ändert das Eingabeaufforderungsfenster den Kontext in den Container. Im Container erstellen wir eine einfache Textdatei "Hello World" und schließen dann den Container, indem Sie die folgenden Befehle eingeben:

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. Rufen Sie die Container-ID für den Container ab, den Sie soeben beendet haben, indem Sie den [docker PS](https://docs.docker.com/engine/reference/commandline/ps/) -Befehl ausführen:

   ```console
   docker ps -a
   ```

4. Erstellen Sie ein neues "HelloWorld"-Bild, das die Änderungen im ersten Container enthält, den Sie ausgeführt haben. Führen Sie dazu den Befehl [docker Commit](https://docs.docker.com/engine/reference/commandline/commit/) aus, der durch `<containerid>` die ID Ihres Containers ersetzt wird:

   ```console
   docker commit <containerid> helloworld
   ```

   Nach Beendigung des Vorgangs verfügen Sie über ein benutzerdefiniertes Image, das ein „Hello World“-Skript enthält. Dies kann mit dem Befehl [Andocken Bilder](https://docs.docker.com/engine/reference/commandline/images/) angezeigt werden.

   ```console
   docker images
   ```

   Hier ist ein Beispiel für die Ausgabe angegeben:

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. Führen Sie schließlich den neuen Container mithilfe des Befehls [Andocken ausführen](https://docs.docker.com/engine/reference/commandline/run/) mit dem `--rm` Parameter aus, der den Container automatisch entfernt, sobald die Befehlszeile (cmd. exe) beendet wird.

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   Das Ergebnis ist, dass ein Container aus dem "HelloWorld"-Bild erstellt wurde, eine Instanz von "cmd. exe" in dem Container gestartet wurde, der unsere Datei liest, und die Dateiinhalte in die Shell ausgeben, und der Container angehalten und dann entfernt wurde.

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Informationen zum containerisieren einer Beispiel-App](./building-sample-app.md)
