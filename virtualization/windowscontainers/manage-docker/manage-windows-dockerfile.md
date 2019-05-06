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
ms.openlocfilehash: 9ff6256ab9708533f72e9b3210f8a5fd32f4048a
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610270"
---
# <a name="dockerfile-on-windows"></a>Dockerfile unter Windows

Das Docker-Modul umfasst Tools für die Erstellung von containerimages automatisiert. Während Sie Container-Images durch Ausführen manuell erstellen können die `docker commit` Befehl Übernahme eines automatisierten imageerstellungsprozesses bietet viele Vorteile, einschließlich:

- Speichern von Containerimages als Code.
- Schnelle und präzise Neuerstellung von Containerimages für Wartung und Aktualisierung.
- Fortlaufende Integration von Containerimages in den Entwicklungszyklus.

Die Docker-Komponenten, die diese Automatisierung steuern, sind die Dockerfile-Datei und der `docker build`-Befehl.

Die dockerfile-Datei ist eine Textdatei mit Anweisungen erforderlich, um ein neues containerimage zu erstellen. Diese Anweisungen beinhalten die Identifikation eines vorhandenen Images, das als Basis verwendet werden kann, Befehle, die während des Erstellungsprozesses ausgeführt werden, und einen Befehl, der bei der Bereitstellung neuer Instanzen des Containerimages ausgeführt wird.

Docker Build, der Docker-Modul-Befehl, der eine dockerfile-Datei nutzt und den imageerstellungsprozess ist.

In diesem Thema wird erläutert, wie dockerfile-Dateien mit Windows-Container verwenden, ihre grundlegende Syntax verstehen und was bedeutet die am häufigsten verwendeten Dockerfile-Anweisungen.

Dieses Dokument wird das Konzept der containerimages und containerimageebenen erläutert. Wenn Sie mehr über Images und imageebenen erfahren möchten, finden Sie unter [der Schnellstart-Anleitung für Bilder](../quick-start/quick-start-images.md).

Eine vollständige Übersicht über Dockerfiles finden Sie in der [Dockerfile-Referenz](https://docs.docker.com/engine/reference/builder/).

## <a name="basic-syntax"></a>Grundlegende Syntax

In ihrer grundlegendsten Form kann eine Dockerfile-Datei sehr einfach sein. Das folgende Beispiel erstellt ein neues Image, das IIS und eine „Hello World“-Website beinhaltet. Dieses Beispiel enthält Kommentare (mit `#` gekennzeichnet), die jeden Schritt erläutern. Die nachfolgenden Abschnitte dieses Artikels gehen ausführlicher auf Dockerfile-Syntaxregeln und -Anweisungen ein.

>[!NOTE]
>Eine dockerfile-Datei muss ohne Erweiterung erstellt werden. Zu diesem Zweck in Windows erstellen Sie die Datei mit dem Editor Ihrer Wahl, und speichern Sie es mit der Notation "Dockerfile" (einschließlich der Anführungszeichen).

```dockerfile
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM microsoft/windowsservercore

# Metadata indicating an image maintainer.
LABEL maintainer="jshelton@contoso.com"

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Weitere Beispiele von dockerfile-Dateien für Windows finden Sie in der [dockerfile-Datei für Windows-Repository](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## <a name="instructions"></a>Anweisungen

Dockerfile-Anweisungen informieren das Docker-Modul die Anweisungen zum Erstellen eines containerimages benötigt. Diese Anweisungen werden nacheinander und in der Reihenfolge ausgeführt. In den folgenden Beispielen werden die am häufigsten verwendeten Anweisungen in der dockerfile-Dateien. Eine vollständige Liste der Dockerfile-Anweisungen finden Sie unter der [Dockerfile-Referenz](https://docs.docker.com/engine/reference/builder/).

### <a name="from"></a>FROM

Die `FROM`-Anweisung legt das Containerimage fest, das während der Erstellung des neuen Images verwendet wird. Beispielsweise wird bei Verwendung von Anweisung `FROM microsoft/windowsservercore` das resultierende Image vom Windows Server Core-Basisbetriebssystemimage abgeleitet und ist davon abhängig. Ist das angegebene Image nicht auf dem System vorhanden, auf dem der „Docker Build“-Prozess ausgeführt wird, versucht das Docker-Modul, das Image von einer öffentlichen oder privaten Imageregistrierung herunterzuladen.

Die FROM-Anweisung Format wird wie folgt aus:

```dockerfile
FROM <image>
```

Hier ist ein Beispiel für die FROM-Befehl aus:

```dockerfile
FROM microsoft/windowsservercore
```

Ausführlichere Informationen finden Sie unter den [FROM-Referenz](https://docs.docker.com/engine/reference/builder/#from).

### <a name="run"></a>RUN

Die `RUN`-Anweisung gibt Befehle an, die ausgeführt und im neuen Containerimage erfasst werden sollen. Diese Befehle können Elemente wie das Installieren von Software sowie das Erstellen von Dateien, Verzeichnissen und Umgebungskonfigurationen enthalten.

Die RUN-Anweisung wird wie folgt aus:

```dockerfile
# exec form

RUN ["<executable>", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Der Unterschied zwischen der EXEC- und Shell-Format ist wie die `RUN` -Anweisung ausgeführt. Bei Verwendung des Exec-Formats wird das angegebene Programm explizit ausgeführt.

Hier ist ein Beispiel für das Exec-Format:

```dockerfile
FROM microsoft/windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

Das resultierende Image führt die `powershell New-Item c:/test` Befehl:

```dockerfile
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

Im Gegensatz dazu wird im folgenden Beispiel wird den gleichen Vorgang im Shell-Format ausgeführt:

```dockerfile
FROM microsoft/windowsservercore

RUN powershell New-Item c:\test
```

Das resultierende Image besteht eine Anweisung ausführen von `cmd /S /C powershell New-Item c:\test`.

```dockerfile
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

### <a name="considerations-for-using-run-with-windows"></a>Überlegungen zur Verwendung mit Windows ausführen

Wenn unter Windows die `RUN`-Anweisung mit dem Exec-Format verwendet wird, müssen umgekehrte Schrägstriche mit Escapezeichen versehen werden.

```dockerfile
RUN ["powershell", "New-Item", "c:\\test"]
```

Wenn das Zielprogramm ein Windows Installer ist, müssen Sie das Setup über Extrahieren der `/x:<directory>` kennzeichnen, bevor das eigentliche (automatisch) Installationsverfahren gestartet werden kann. Sie müssen auch warten, für den Befehl zum Beenden, bevor Sie fortfahren. Andernfalls wird der Vorgang vorzeitig ohne dass etwas installiert wurde. Weitere Informationen finden Sie im folgenden Beispiel.

#### <a name="examples-of-using-run-with-windows"></a>Beispiele für die Verwendung mit Windows ausführen

Im folgende Beispiel dockerfile-Datei verwendet DISM zum Installieren von IIS im containerimage befindet:

```dockerfile
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

In diesem Beispiel wird das verteilbare Paket von Visual Studio installiert. `Start-Process` und die `-Wait` Parameter werden verwendet, um das Installationsprogramm auszuführen. Dadurch wird sichergestellt, dass die Installation abgeschlossen ist, bevor Sie mit der nächsten Anweisung in der Dockerfile fortfahren.

```dockerfile
RUN powershell.exe -Command Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
```

Ausführliche Informationen über die RUN-Anweisung finden Sie unter den [Verweis ausführen](https://docs.docker.com/engine/reference/builder/#run).

### <a name="copy"></a>KOPIEREN

Die `COPY` -Anweisung kopiert Dateien und Verzeichnisse in Dateisystem des Containers. Die Dateien und Verzeichnisse müssen in einem Pfad relativ zur dockerfile-Datei sein.

Die `COPY` Anweisung Format wird wie folgt aus:

```dockerfile
COPY <source> <destination>
```

Wenn die Quelle oder das Ziel Leerzeichen enthält, schließen Sie den Pfad in eckige Klammern und doppelte Anführungszeichen, wie im folgenden Beispiel dargestellt:

```dockerfile
COPY ["<source>", "<destination>"]
```

#### <a name="considerations-for-using-copy-with-windows"></a>Überlegungen zur Verwendung mit Windows kopieren

Unter Windows müssen im Zielformat Schrägstriche verwendet werden. Dies sind z. B. gültige `COPY` Anweisungen:

```dockerfile
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

In der Zwischenzeit funktioniert nicht für das folgende Format umgekehrte Schrägstriche:

```dockerfile
COPY test1.txt c:\temp\
```

#### <a name="examples-of-using-copy-with-windows"></a>Kopieren Sie Beispiele für die Verwendung mit Windows

Das folgende Beispiel fügt den Inhalt des Quellverzeichnisses in ein Verzeichnis namens `sqllite` im containerimage befindet:

```dockerfile
COPY source /sqlite/
```

Im folgende Beispiel werden alle Dateien, die mit Config zu beginnen Hinzufügen der `c:\temp` -Verzeichnis des containerimages:

```dockerfile
COPY config* c:/temp/
```

Ausführlichere Informationen zu den `COPY` -Anweisung finden Sie im [COPY-Referenz](https://docs.docker.com/engine/reference/builder/#copy).

### <a name="add"></a>HINZUFÜGEN

Die ADD-Anweisung ist die COPY-Anweisung, aber mit noch mehr Funktionen. Die `ADD`-Anweisung kann nicht nur Dateien vom Host in das Containerimage kopieren, sondern auch von einem Remotestandort aus mit einer URL-Spezifikation.

Die `ADD` Anweisung Format wird wie folgt aus:

```dockerfile
ADD <source> <destination>
```

Wenn die Quelle oder das Ziel Leerzeichen enthalten, schließen Sie den Pfad in eckige Klammern und doppelte Anführungszeichen:

```dockerfile
ADD ["<source>", "<destination>"]
```

#### <a name="considerations-for-running-add-with-windows"></a>Hinzufügen von Überlegungen für die Ausführung mit Windows

Unter Windows müssen im Zielformat Schrägstriche verwendet werden. Dies sind z. B. gültige `ADD` Anweisungen:

```dockerfile
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

In der Zwischenzeit funktioniert nicht für das folgende Format umgekehrte Schrägstriche:

```dockerfile
ADD test1.txt c:\temp\
```

Außerdem erweitert die `ADD`-Anweisung unter Linux komprimierte Pakete beim Kopieren. Diese Funktionalität ist in Windows nicht verfügbar.

#### <a name="examples-of-using-add-with-windows"></a>Beispiele für die Verwendung mit Windows hinzufügen

Das folgende Beispiel fügt den Inhalt des Quellverzeichnisses in ein Verzeichnis namens `sqllite` im containerimage befindet:

```dockerfile
ADD source /sqlite/
```

Im folgende Beispiel werden alle Dateien, die mit "Config" beginnen hinzugefügt, um die `c:\temp` -Verzeichnis des containerimages.

```dockerfile
ADD config* c:/temp/
```

Im folgende Beispiel wird Python für Windows in Herunterladen der `c:\temp` -Verzeichnis des containerimages.

```dockerfile
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Ausführlichere Informationen zu den `ADD` -Anweisung finden Sie unter den [Verweis hinzufügen](https://docs.docker.com/engine/reference/builder/#add).

### <a name="workdir"></a>WORKDIR

Die `WORKDIR`-Anweisung legt ein Arbeitsverzeichnis für andere Dockerfile-Anweisungen wie z.B. `RUN` und `CMD` sowie das Arbeitsverzeichnis für ausgeführte Instanzen des Containerimages fest.

Die `WORKDIR` Anweisung Format wird wie folgt aus:

```dockerfile
WORKDIR <path to working directory>
```

#### <a name="considerations-for-using-workdir-with-windows"></a>Überlegungen zur Verwendung des WORKDIR mit Windows

Wenn das Arbeitsverzeichnis unter Windows einen umgekehrten Schrägstrich enthält, muss es mit Escapezeichen versehen werden.

```dockerfile
WORKDIR c:\\windows
```

**Beispiele**

```dockerfile
WORKDIR c:\\Apache24\\bin
```

Ausführliche Informationen über die `WORKDIR` -Anweisung finden Sie im [WORKDIR-Referenz](https://docs.docker.com/engine/reference/builder/#workdir).

### <a name="cmd"></a>CMD

Die `CMD`-Anweisung legt den Standardbefehl fest, der bei der Bereitstellung einer Instanz des Containerimages ausgeführt werden soll. Beispielsweise, wenn der Container einen NGINX-Webserver hostet die `CMD` gehören Anweisungen zum Starten des Webservers mit einem Befehl wie `nginx.exe`. Wenn mehrere `CMD`-Anweisungen in einer Dockerfile-Datei angegeben sind, wird nur die letzte ausgewertet.

Die `CMD` Anweisung Format wird wie folgt aus:

```dockerfile
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

#### <a name="considerations-for-using-cmd-with-windows"></a>Überlegungen zur Verwendung des CMD mit Windows

Unter Windows müssen in Dateipfaden, die in der `CMD`-Anweisung angegeben werden, Schrägstriche oder mit Escapezeichen versehene umgekehrte Schrägstriche `\\` verwendet werden. Im folgenden sind gültige `CMD` Anweisungen:

```dockerfile
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```

Das folgende Format ohne den richtigen Schrägstriche funktioniert jedoch nicht:

```dockerfile
CMD c:\Apache24\bin\httpd.exe -w
```

Ausführlichere Informationen zu den `CMD` -Anweisung finden Sie im [CMD-Referenz](https://docs.docker.com/engine/reference/builder/#cmd).

## <a name="escape-character"></a>Escapezeichen

In vielen Fällen müssen eine Dockerfile-Anweisung mehrere Zeilen umfassen. Zu diesem Zweck können Sie ein Escapezeichen verwenden. In einer Dockerfile-Anweisung wird als Escapezeichen standardmäßig ein umgekehrter Schrägstrich verwendet: `\`. Jedoch, da der umgekehrte Schrägstrich auch ein dateipfadtrennzeichen in Windows ist, kann es mehrere Zeilen umfassen verwendet Probleme verursachen. Um dies zu umgehen, können Sie eine Parser-Anweisung verwenden, um die Standard-Escapezeichen ändern. Weitere Informationen zu Parser-Anweisungen finden Sie unter [Parser-Anweisungen](https://docs.docker.com/engine/reference/builder/#parser-directives).

Das folgende Beispiel zeigt eine einzelne RUN-Anweisung, die mehrere Zeilen mit Escapezeichen standardmäßig umfasst:

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Platzieren Sie zum Ändern des Escapezeichens eine Escape-Parser-Anweisung in der ersten Zeile der Dockerfile-Datei. Dies kann im folgenden Beispiel erkannt werden.

>[!NOTE]
>Nur zwei Werte als Escapezeichen verwendet werden können: `\` und `` ` ``.

```dockerfile
# escape=`

FROM microsoft/windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Weitere Informationen zur Escape-Parser-Anweisung finden Sie unter [Escape-Parser-Anweisung](https://docs.docker.com/engine/reference/builder/#escape).

## <a name="powershell-in-dockerfile"></a>PowerShell in Dockerfile

### <a name="powershell-cmdlets"></a>PowerShell-Cmdlets

PowerShell-Cmdlets ausgeführt werden kann, in einer dockerfile-Datei mit den `RUN` Vorgang.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### <a name="rest-calls"></a>REST-Aufrufe

PowerShell `Invoke-WebRequest` Cmdlet kann beim Sammeln von Informationen oder Dateien aus einem Webdienst hilfreich sein. Beispielsweise, wenn Sie ein Image, die Python enthält erstellen, können Sie festlegen `$ProgressPreference` , `SilentlyContinue` , schneller Downloads zu erreichen, wie im folgenden Beispiel gezeigt.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  $ProgressPreference = 'SilentlyContinue'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>`Invoke-WebRequest` funktioniert auch im Nano Server.

Eine weitere Möglichkeit zur Verwendung von PowerShell zum Herunterladen von Dateien während der Imageerstellung ist die Verwendung der .NET WebClient-Bibliothek. Dies kann die Downloadleistung verbessern. Im folgenden Beispiel wird die Python-Software mit der WebClient-Bibliothek heruntergeladen.

```dockerfile
FROM microsoft/windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

>[!NOTE]
>Nano Server unterstützt das WebClient derzeit nicht.

### <a name="powershell-scripts"></a>PowerShell-Skripts

In einigen Fällen kann es hilfreich sein, ein Skript in den Containern, die Sie während des Erstellungsprozesses zu verwenden, und führen Sie das Skript aus innerhalb des Containers.

>[!NOTE]
>Dies beschränkt jegliche Zwischenspeicherung von imageebenen und verringert die Lesbarkeit der dockerfile-Datei.

In diesem Beispiel wird ein Skript vom Buildcomputer mithilfe der `ADD`-Anweisung in den Container kopiert. Dieses Skript wird dann mit der RUN-Anweisung ausgeführt.

```dockerfile
FROM microsoft/windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## <a name="docker-build"></a>Docker build

Sobald eine dockerfile-Datei erstellt und auf dem Datenträger gespeichert wurde, können Sie ausführen `docker build` auf das neue Image zu erstellen. Der `docker build`-Befehl unterstützt verschiedene optionale Parameter und einen Pfad zur Dockerfile-Datei. Eine ausführliche Dokumentation zu Docker Build einschließlich einer Liste aller Buildoptionen Sie, finden Sie die [build-Referenz](https://docs.docker.com/engine/reference/commandline/build/#build).

Das Format der `docker build` Befehl wird wie folgt aus:

```dockerfile
docker build [OPTIONS] PATH
```

Der folgende Befehl wird z. B. ein Bild mit dem Namen "Iis". erstellt.

```dockerfile
docker build -t iis .
```

Wenn der Buildprozess eingeleitet wurde, wird die Ausgabe Status anzugeben und Fehlermeldungen zurückgegeben.

```dockerfile
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM micrsoft/windowsservercore
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

Das Ergebnis ist ein neues containerimage, das in diesem Beispiel mit dem Namen "Iis".

```dockerfile
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## <a name="further-reading-and-references"></a>Weitere Informationen und Referenzen

- [Optimieren von Dockerfiles und Docker-Builds für Windows](optimize-windows-dockerfile.md)
- [Dockerfile-Referenz](https://docs.docker.com/engine/reference/builder/)
