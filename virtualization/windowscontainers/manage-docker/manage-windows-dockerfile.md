---
title: Dockerfile und Windows-Container
description: Erstellen Sie Dockerfile-Dateien für Windows-Container.
keywords: Docker, Container
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
ms.openlocfilehash: 9fef74c029dc3efc220b1f9924d2695cdbaa61be
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909660"
---
# <a name="dockerfile-on-windows"></a>Dockerfile unter Windows

Die Docker-Engine enthält Tools zum Automatisieren der Erstellung von Container Images. Obwohl Sie Container Images manuell erstellen können, indem Sie den `docker commit`-Befehl ausführen, bietet die Übernahme eines automatisierten Abbild Erstellungs Prozesses zahlreiche Vorteile, darunter:

- Speichern von Containerimages als Code.
- Schnelle und präzise Neuerstellung von Containerimages für Wartung und Aktualisierung.
- Fortlaufende Integration von Containerimages in den Entwicklungszyklus.

Die Docker-Komponenten, die diese Automatisierung steuern, sind die Dockerfile-Datei und der `docker build`-Befehl.

Die dockerfile-Datei ist eine Textdatei, die die Anweisungen enthält, die zum Erstellen eines neuen Container Images erforderlich sind. Diese Anweisungen beinhalten die Identifikation eines vorhandenen Images, das als Basis verwendet werden kann, Befehle, die während des Erstellungsprozesses ausgeführt werden, und einen Befehl, der bei der Bereitstellung neuer Instanzen des Containerimages ausgeführt wird.

Docker Build ist der Docker-Engine-Befehl, der eine dockerfile-Datei nutzt und den Image Erstellungs Prozess auslöst.

In diesem Thema erfahren Sie, wie Sie dockerfiles mit Windows-Containern verwenden, die grundlegende Syntax verstehen und die gängigsten dockerfile-Anweisungen verwenden.

In diesem Dokument wird das Konzept von Container Images und Container Image Ebenen erörtert. Weitere Informationen zu Images und Bildschichten finden Sie unter [Containerbasis Images](../manage-containers/container-base-images.md).

Einen umfassenden Einblick in dockerfiles finden Sie in der [dockerfile-Referenz](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Grundlegende Syntax

In ihrer grundlegendsten Form kann eine Dockerfile-Datei sehr einfach sein. Das folgende Beispiel erstellt ein neues Image, das IIS und eine „Hello World“-Website beinhaltet. Dieses Beispiel enthält Kommentare (mit `#` gekennzeichnet), die jeden Schritt erläutern. Die nachfolgenden Abschnitte dieses Artikels gehen ausführlicher auf Dockerfile-Syntaxregeln und -Anweisungen ein.

>[!NOTE]
>Es muss eine dockerfile-Datei ohne Erweiterung erstellt werden. Um dies in Windows zu tun, erstellen Sie die Datei mit dem Editor Ihrer Wahl, und speichern Sie Sie dann mit der Notation "dockerfile" (einschließlich der Anführungszeichen).

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM mcr.microsoft.com/windows/servercore:ltsc2019

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Weitere Beispiele für dockerfiles für Windows finden Sie unter [dockerfile für Windows-Repository](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Anweisungen

Dockerfile-Anweisungen stellen der Docker-Engine die Anweisungen zur Verfügung, die zum Erstellen eines Container Images erforderlich sind. Diese Anweisungen werden nacheinander und in der richtigen Reihenfolge ausgeführt. In den folgenden Beispielen finden Sie die am häufigsten verwendeten Anweisungen in dockerfiles. Eine umfassende Liste der dockerfile-Anweisungen finden Sie in der [dockerfile-Referenz](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

Die `FROM`-Anweisung legt das Containerimage fest, das während der Erstellung des neuen Images verwendet wird. Beispielsweise wird bei Verwendung von Anweisung `FROM mcr.microsoft.com/windows/servercore` das resultierende Image vom Windows Server Core-Basisbetriebssystemimage abgeleitet und ist davon abhängig. Ist das angegebene Image nicht auf dem System vorhanden, auf dem der „Docker Build“-Prozess ausgeführt wird, versucht das Docker-Modul, das Image von einer öffentlichen oder privaten Imageregistrierung herunterzuladen.

Das Format der FROM-Anweisung sieht wie folgt aus:

```dockerfile
FROM <image>
```

Im folgenden finden Sie ein Beispiel für den from-Befehl:

So laden Sie die ltsc2019-Version von Windows Server Core vom Microsoft Container Registry (MCR) herunter:
```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
```

Ausführlichere Informationen finden Sie in der [from-Referenz](https://docs.docker.com/engine/reference/builder/#from).

### <a name="run"></a>RUN

Die `RUN`-Anweisung gibt Befehle an, die ausgeführt und im neuen Containerimage erfasst werden sollen. Diese Befehle können Elemente wie das Installieren von Software sowie das Erstellen von Dateien, Verzeichnissen und Umgebungskonfigurationen enthalten.

Die Run-Anweisung sieht wie folgt aus:

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Der Unterschied zwischen dem exec-und dem shellformular besteht darin, wie die `RUN` Anweisung ausgeführt wird. Bei Verwendung des Exec-Formats wird das angegebene Programm explizit ausgeführt.

Im folgenden finden Sie ein Beispiel für das Exec-Formular:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN ["powershell", "New-Item", "c:/test"]
```

Das resultierende Image führt den `powershell New-Item c:/test` Befehl aus:

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

Im folgenden Beispiel wird der gleiche Vorgang in shellform ausgeführt:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell New-Item c:\test
```

Das resultierende Bild verfügt über eine Run-Anweisung `cmd /S /C powershell New-Item c:\test`.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Überlegungen zur Verwendung von Run with Windows

Wenn unter Windows die `RUN`-Anweisung mit dem Exec-Format verwendet wird, müssen umgekehrte Schrägstriche mit Escapezeichen versehen werden.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Wenn das Zielprogramm ein Windows Installer ist, müssen Sie das Setup über das `/x:<directory>`-Flag extrahieren, bevor Sie das eigentliche (unbeaufsichtigte) Installationsverfahren starten können. Sie müssen auch warten, bis der Befehl beendet wird, bevor Sie etwas anderes tun. Andernfalls wird der Prozess vorzeitig beendet, ohne etwas zu installieren. Weitere Informationen finden Sie im folgenden Beispiel.

#### <a name="examples-of-using-run-with-windows"></a>Beispiele für die Verwendung von Run with Windows

Die folgende dockerfile-Beispieldatei verwendet die-Funktion zum Installieren von IIS im Container Image:

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

In diesem Beispiel wird das verteilbare Paket von Visual Studio installiert. `Start-Process` und der Parameter `-Wait` werden verwendet, um das Installationsprogramm auszuführen. Dadurch wird sichergestellt, dass die Installation abgeschlossen wird, bevor Sie mit der nächsten Anweisung in der dockerfile-Datei fortfahren.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Ausführliche Informationen zur Run-Anweisung finden Sie in der [Run-Referenz](https://docs.docker.com/engine/reference/builder/#run).

### <a name="copy"></a>KOPIEREN

Die `COPY` Anweisung kopiert Dateien und Verzeichnisse in das Dateisystem des Containers. Die Dateien und Verzeichnisse müssen sich in einem Pfad befinden, der relativ zur dockerfile-Datei ist.

Das Format der `COPY` Anweisung lautet wie folgt:

```dockerfile
COPY <source> <destination>
```

Wenn die Quelle oder das Ziel Leerzeichen enthält, schließen Sie den Pfad in eckige Klammern und doppelte Anführungszeichen ein, wie im folgenden Beispiel gezeigt:

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Überlegungen zur Verwendung von Copy mit Windows

Unter Windows müssen im Zielformat Schrägstriche verwendet werden. Diese `COPY` Anweisungen sind z. b. gültig:

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

In der Zwischenzeit funktionieren folgende Formate mit umgekehrten Schrägstrichen nicht:

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Beispiele für die Verwendung von Copy mit Windows

Im folgenden Beispiel wird der Inhalt des Quell Verzeichnisses einem Verzeichnis mit dem Namen `sqllite` im Container Image hinzugefügt:

```dockerfile
COPY source /sqlite/
```

Im folgenden Beispiel werden alle Dateien, die mit config beginnen, dem `c:\temp`-Verzeichnis des Container Images hinzugefügt:

```dockerfile
COPY config* c:/temp/
```

Ausführlichere Informationen zur `COPY`-Anweisung finden Sie in der [Referenz zum Kopieren](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>HINZUFÜGEN

Die Add-Anweisung ähnelt der Copy-Anweisung, aber mit noch mehr Funktionen. Die `ADD`-Anweisung kann nicht nur Dateien vom Host in das Containerimage kopieren, sondern auch von einem Remotestandort aus mit einer URL-Spezifikation.

Das Format der `ADD` Anweisung lautet wie folgt:

```dockerfile
ADD <source> <destination>
```

Wenn die Quelle oder das Ziel Leerzeichen enthält, schließen Sie den Pfad in eckige Klammern und doppelte Anführungszeichen ein:

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Überlegungen zum Ausführen von Add with Windows

Unter Windows müssen im Zielformat Schrägstriche verwendet werden. Diese `ADD` Anweisungen sind z. b. gültig:

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

In der Zwischenzeit funktionieren folgende Formate mit umgekehrten Schrägstrichen nicht:

```dockerfile
ADD test1.txt c:\temp\
```

Außerdem erweitert die `ADD`-Anweisung unter Linux komprimierte Pakete beim Kopieren. Diese Funktionalität ist in Windows nicht verfügbar.

#### <a name="examples-of-using-add-with-windows"></a>Beispiele für die Verwendung von Add with Windows

Im folgenden Beispiel wird der Inhalt des Quell Verzeichnisses einem Verzeichnis mit dem Namen `sqllite` im Container Image hinzugefügt:

```dockerfile
ADD source /sqlite/
```

Im folgenden Beispiel werden alle Dateien, die mit "config" beginnen, dem `c:\temp`-Verzeichnis des Container Images hinzugefügt.

```dockerfile
ADD config* c:/temp/
```

Im folgenden Beispiel wird python für Windows in das `c:\temp`-Verzeichnis des Container Images heruntergeladen.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Ausführlichere Informationen zur `ADD`-Anweisung finden Sie in der [Add-Referenz](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

Die `WORKDIR`-Anweisung legt ein Arbeitsverzeichnis für andere Dockerfile-Anweisungen wie z.B. `RUN` und `CMD` sowie das Arbeitsverzeichnis für ausgeführte Instanzen des Containerimages fest.

Das Format der `WORKDIR` Anweisung lautet wie folgt:

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Überlegungen zur Verwendung von WorkDir mit Windows

Wenn das Arbeitsverzeichnis unter Windows einen umgekehrten Schrägstrich enthält, muss es mit Escapezeichen versehen werden.

```dockerfile
WORKDIR c:\\windows
```

**Beispiele**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

Ausführliche Informationen zur `WORKDIR`-Anweisung finden Sie in der [WorkDir-Referenz](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

Die `CMD`-Anweisung legt den Standardbefehl fest, der bei der Bereitstellung einer Instanz des Containerimages ausgeführt werden soll. Wenn der Container beispielsweise einen nginx-Webserver als Host verwendet, können die `CMD` Anweisungen zum Starten des Webservers mit einem Befehl wie `nginx.exe`enthalten. Wenn mehrere `CMD`-Anweisungen in einer Dockerfile-Datei angegeben sind, wird nur die letzte ausgewertet.

Das Format der `CMD` Anweisung lautet wie folgt:

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Überlegungen zur Verwendung von cmd mit Windows

Unter Windows müssen in Dateipfaden, die in der `CMD`-Anweisung angegeben werden, Schrägstriche oder mit Escapezeichen versehene umgekehrte Schrägstriche `\\` verwendet werden. Die folgenden `CMD` Anweisungen sind gültig:

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

Das folgende Format ohne die entsprechenden Schrägstriche funktioniert jedoch nicht:

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

Ausführlichere Informationen zur `CMD`-Anweisung finden Sie in der [cmd-Referenz](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Escapezeichen

In vielen Fällen muss eine dockerfile-Anweisung mehrere Zeilen umfassen. Zu diesem Zweck können Sie ein Escapezeichen verwenden. In einer Dockerfile-Anweisung wird als Escapezeichen standardmäßig ein umgekehrter Schrägstrich verwendet: `\`. Da der umgekehrte Schrägstrich jedoch auch ein Dateipfad Trennzeichen in Windows ist, kann die Verwendung für mehrere Zeilen zu Problemen führen. Um dies zu umgehen, können Sie eine parserdirektive verwenden, um das Standardescapezeichen zu ändern. Weitere Informationen zu parserdirektiven finden Sie unter [Parser-Direktiven](https://docs.docker.com/engine/reference/builder/#parser-directives).

Im folgenden Beispiel wird eine einzelne Lauf Anweisung gezeigt, die mehrere Zeilen mit dem Standardescapezeichen umfasst:

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Platzieren Sie zum Ändern des Escapezeichens eine Escape-Parser-Anweisung in der ersten Zeile der Dockerfile-Datei. Dies wird im folgenden Beispiel veranschaulicht.

>[!NOTE]
>Es können nur zwei Werte als Escapezeichen verwendet werden: `\` und `` ` ``.

```dockerfile
# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Weitere Informationen zur Escape-Parser-Direktive finden Sie unter [Escape-Parser-Direktive](https://docs.docker.com/engine/reference/builder/#escape).

## <a name="powershell-in-dockerfile"></a>PowerShell in Dockerfile

### <a name="powershell-cmdlets"></a>PowerShell-Cmdlets

PowerShell-Cmdlets können mit dem `RUN`-Vorgang in einer dockerfile-Datei ausgeführt werden.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>Rest-Aufrufe

Das PowerShell-Cmdlet "`Invoke-WebRequest`" kann beim Sammeln von Informationen oder Dateien aus einem Webdienst nützlich sein. Wenn Sie z. b. ein Image erstellen, das python enthält, können Sie `$ProgressPreference` auf `SilentlyContinue` festlegen, um schnellere Downloads zu erzielen, wie im folgenden Beispiel gezeigt.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` funktioniert auch in Nano Server.

Eine weitere Möglichkeit zur Verwendung von PowerShell zum Herunterladen von Dateien während der Imageerstellung ist die Verwendung der .NET WebClient-Bibliothek. Dies kann die Downloadleistung verbessern. Im folgenden Beispiel wird die Python-Software mit der WebClient-Bibliothek heruntergeladen.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Der WebClient wird von Nano Server derzeit nicht unterstützt.

### <a name="powershell-scripts"></a>PowerShell-Skripts

In einigen Fällen kann es hilfreich sein, ein Skript in die Container zu kopieren, die Sie während der Abbild Erstellung verwenden, und dann das Skript aus dem Container heraus auszuführen.

>[!NOTE]
>Dadurch wird das Zwischenspeichern von Bildebenen eingeschränkt und die Lesbarkeit der dockerfile-Datei verringert.

In diesem Beispiel wird ein Skript vom Buildcomputer mithilfe der `ADD`-Anweisung in den Container kopiert. Dieses Skript wird dann mit der RUN-Anweisung ausgeführt.

```
FROM mcr.microsoft.com/windows/servercore:ltsc2019
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker-Build

Nachdem eine dockerfile-Datei erstellt und auf dem Datenträger gespeichert wurde, können Sie `docker build` ausführen, um das neue Image zu erstellen. Der `docker build`-Befehl unterstützt verschiedene optionale Parameter und einen Pfad zur Dockerfile-Datei. Eine vollständige Dokumentation zu docker Build, einschließlich einer Liste aller Buildoptionen, finden Sie in der [buildreferenz](https://docs.docker.com/engine/reference/commandline/build/#build).

Das Format des `docker build` Befehls lautet wie folgt:

```dockerfile
docker build [OPTIONS] PATH
```

Beispielsweise wird mit dem folgenden Befehl ein Image mit dem Namen "IIS" erstellt.

```dockerfile
docker build -t iis .
```

Wenn der Buildprozess initiiert wurde, zeigt die Ausgabe den Status an und gibt alle ausgelösten Fehler zurück.

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM mcr.microsoft.com/windows/servercore:ltsc2019
 ---> 6801d964fda5

Step 2 : RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
 ---> Running in ae8759fb47db

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0

Enabling feature(s)
The operation completed successfully.

 ---> 4cd675d35444
Removing intermediate container ae8759fb47db

Step 3 : RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
 ---> Running in 9a26b8bcaa3a
 ---> e2aafdfbe392
Removing intermediate container 9a26b8bcaa3a

Successfully built e2aafdfbe392
```

Das Ergebnis ist ein neues Container Image, das in diesem Beispiel den Namen "IIS" hat.

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Weitere Informationen und Verweise

- [Optimieren von dockerfiles und docker Build für Windows](optimize-windows-dockerfile.md)
- [Dockerfile-Referenz](https://docs.docker.com/engine/reference/builder/)
