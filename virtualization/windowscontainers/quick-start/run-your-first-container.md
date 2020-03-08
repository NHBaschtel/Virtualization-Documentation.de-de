---
title: Windows-und Linux-Container unter Windows 10
description: Containerbereitstellung – Schnellstart
keywords: docker, Container, lkuh
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 551d405d836cfb16b587ef78bc2d5f5abbd8648f
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853814"
---
# <a name="get-started-run-your-first-windows-container"></a>Erste Schritte: Ausführen Ihres ersten Windows-Containers

In diesem Thema wird beschrieben, wie Sie Ihren ersten Windows-Container nach dem Einrichten Ihrer Umgebung wie unter Erste Schritte [: Vorbereiten von Windows für Container](./set-up-environment.md)beschrieben einrichten. Zum Ausführen eines Containers installieren Sie zunächst ein Basis Image, das eine grundlegende Ebene von Betriebssystemdiensten für ihren Container bereitstellt. Anschließend erstellen und führen Sie ein Container Image aus, das auf dem Basis Image basiert. Weitere Informationen finden Sie unter.

## <a name="install-a-container-base-image"></a>Installieren eines Containerbasis Images

Alle Container werden aus Containerimages erstellt. Microsoft bietet verschiedene Starter Images, die als Basis Images bezeichnet werden, um eine Auswahl zu treffen (Weitere Informationen finden Sie unter [Container Basis Images](../manage-containers/container-base-images.md)). Durch diese Vorgehensweise wird das Lightweight Nano Server-Basis Image abgerufen (heruntergeladen und installiert).

1. Öffnen Sie ein Eingabe Aufforderungs Fenster (z. b. die integrierte Eingabeaufforderung, PowerShell oder das [Windows-Terminal](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)), und führen Sie dann den folgenden Befehl aus, um das Basis Image herunterzuladen und zu installieren:

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > Wenn eine Fehlermeldung angezeigt wird, die `no matching manifest for unknown in the manifest list entries`lautet, stellen Sie sicher, dass docker nicht für das Ausführen von Linux-Containern konfiguriert ist.

2. Nachdem das Herunterladen des Images abgeschlossen ist – lesen [Sie die Lizenz](../images-eula.md) Bedingungen, während Sie warten – überprüfen Sie das vorhanden sein Ihres Systems, indem Sie das lokale docker-imagerepository Abfragen. Wenn Sie den Befehl `docker images` ausführen, wird eine Liste der installierten Images zurückgegeben.

   Im folgenden finden Sie ein Beispiel für die Ausgabe, die das Nano Server-Image zeigt.

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Ausführen eines Windows-Containers

In diesem einfachen Beispiel wird ein "Hallo Welt"-Container Image erstellt und bereitgestellt. Um die optimale Leistung zu erzielen, führen Sie diese Befehle in einem Eingabe Aufforderungs Fenster mit erhöhten Rechten aus (verwenden Sie jedoch nicht die Windows PowerShell ISE – es funktioniert nicht für interaktive Sitzungen mit Containern, da die Container scheinbar hängen bleiben).

1. Starten Sie einen Container mit einer interaktiven Sitzung aus dem `nanoserver` Image, indem Sie den folgenden Befehl in das Eingabe Aufforderungs Fenster eingeben:

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. Nachdem der Container gestartet wurde, ändert das Eingabe Aufforderungs Fenster den Kontext in den Container. Im Container erstellen wir eine einfache Textdatei mit dem Namen "Hallo Welt" und beenden dann den Container, indem wir die folgenden Befehle eingeben:

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. Sie erhalten die Container-ID für den Container, den Sie soeben beendet haben, indem Sie den Befehl " [docker PS](https://docs.docker.com/engine/reference/commandline/ps/) " ausführen:

   ```console
   docker ps -a
   ```

4. Erstellen Sie ein neues "HelloWorld"-Image, das die Änderungen im ersten Container enthält, den Sie ausgeführt haben. Führen Sie hierzu den Befehl [docker Commit](https://docs.docker.com/engine/reference/commandline/commit/) aus, und ersetzen Sie `<containerid>` durch die ID Ihres Containers:

   ```console
   docker commit <containerid> helloworld
   ```

   Nach Beendigung des Vorgangs verfügen Sie über ein benutzerdefiniertes Image, das ein „Hello World“-Skript enthält. Dies kann mit dem Befehl " [docker Images](https://docs.docker.com/engine/reference/commandline/images/) " angezeigt werden.

   ```console
   docker images
   ```

   Hier ist ein Beispiel für die Ausgabe angegeben:

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. Führen Sie abschließend den neuen Container mithilfe des Befehls " [docker Run](https://docs.docker.com/engine/reference/commandline/run/) " mit dem `--rm`-Parameter aus, der den Container automatisch entfernt, sobald die Befehlszeile (cmd. exe) beendet wird.

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   Das Ergebnis ist, dass ein Container aus dem "HelloWorld"-Image erstellt wurde, eine Instanz von "cmd. exe" im Container gestartet wurde, die unsere Datei liest und den Dateiinhalt an die Shell ausgibt, und dann der Container beendet und entfernt wurde.

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Informationen zum containerisieren einer Beispiel-App](./building-sample-app.md)
