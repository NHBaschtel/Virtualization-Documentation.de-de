---
title: Dockerfile und Windows-Container
description: "Erstellen Sie Dockerfile-Dateien für Windows-Container."
keywords: Docker, Container
author: PatrickLang
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 75fed138-9239-4da9-bce4-4f2e2ad469a1
translationtype: Human Translation
ms.sourcegitcommit: 31515396358c124212b53540af8a0dcdad3580e4
ms.openlocfilehash: 20dcc6d263488673bf0a025058c3dee8d30168a2

---

# Dockerfile unter Windows

Das Docker-Modul umfasst Tools zum Automatisieren der Erstellung von Containerimages. Containerimages können zwar manuell mit dem `docker commit`-Befehl erstellt werden, doch die Übernahme eines automatisierten Imageerstellungsprozesses bietet viele Vorteile, unter anderem:

- Speichern von Containerimages als Code.
- Schnelle und präzise Neuerstellung von Containerimages für Wartung und Aktualisierung.
- Fortlaufende Integration von Containerimages in den Entwicklungszyklus.

Die Docker-Komponenten, die diese Automatisierung steuern, sind die Dockerfile-Datei und der `docker build`-Befehl.

- **Dockerfile**: eine Textdatei mit Anweisungen, die erforderlich sind, um ein neues Containerimage zu erstellen. Diese Anweisungen beinhalten die Identifikation eines vorhandenen Images, das als Basis verwendet werden kann, Befehle, die während des Erstellungsprozesses ausgeführt werden, und einen Befehl, der bei der Bereitstellung neuer Instanzen des Containerimages ausgeführt wird.
- **Docker Build** – der Docker-Modulbefehl, der eine Dockerfile-Datei nutzt und den Imageerstellungsprozess auslöst.

Dieses Dokument stellt die Verwendung einer Dockerfile-Datei mit Windows-Containern vor, erläutert die Syntax und zeigt häufig verwendete Dockerfile-Anweisungen im Detail. 

Das Konzept der Containerimages und Containerimageebenen ist ein zentrales Thema dieses Dokuments. Weitere Informationen zu Images und Imageebenen finden Sie unter [Containerimages](../management/manage_images.md). 

Weitere Informationen zu Dockerfile-Dateien finden Sie in der [Referenz zu Dockerfiles auf docker.com]( https://docs.docker.com/engine/reference/builder/).

## Einführung zu Dockerfile

### Grundlegende Syntax

In ihrer grundlegendsten Form kann eine Dockerfile-Datei sehr einfach sein. Das folgende Beispiel erstellt ein neues Image, das IIS und eine „Hello World“-Website beinhaltet. Dieses Beispiel enthält Kommentare (mit `#` gekennzeichnet), die jeden Schritt erläutern. Nachfolgende Abschnitte dieses Artikels gehen ausführlicher auf Dockerfile-Syntaxregeln und -Anweisungen ein.

```none
# Sample Dockerfile

# Indicates that the windowsservercore image will be used as the base image.
FROM microsoft/windowsservercore

# Metadata indicating an image maintainer.
MAINTAINER jshelton@contoso.com

# Uses dism.exe to install the IIS role.
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart

# Creates an HTML file and adds content to this file.
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html

# Sets a command or process that will run each time a container is run from the new image.
CMD [ "cmd" ]
```

Weitere Beispiele von Dockerfile-Dateien für Windows finden Sie im [Repository zu Dockerfile für Windows] (https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples).

## Anweisungen

Dockerfile-Anweisungen informieren das Docker-Modul über die erforderlichen Schritte zum Erstellen eines Containerimages. Diese Anweisungen werden der Reihe nach einzeln ausgeführt. Hier sehen Sie die Details zu einigen grundlegenden Dockerfile-Anweisungen. Eine vollständige Liste der Dockerfile-Anweisungen finden Sie unter [Dockerfile-Referenz auf Docker.com] (https://docs.docker.com/engine/reference/builder/).

### FROM

Die `FROM`-Anweisung legt das Containerimage fest, das während der Erstellung des neuen Images verwendet wird. Beispielsweise wird bei Verwendung von Anweisung `FROM windowsservercore` das resultierende Image vom Windows Server Core-Basisbetriebssystemimage abgeleitet und ist davon abhängig. Ist das angegebene Image nicht auf dem System vorhanden, auf dem der „Docker Build“-Prozess ausgeführt wird, versucht das Docker-Modul, das Image von einer öffentlichen oder privaten Imageregistrierung herunterzuladen.

**Format**

Die FROM-Anweisung weist dieses Format auf: 

```
FROM <image>
```

**Beispiel**

```
FROM windowsservercore
```

Ausführliche Informationen über die FROM-Anweisung finden Sie in der [FROM-Referenz auf Docker.com]( https://docs.docker.com/engine/reference/builder/#from). 

### RUN

Die `RUN`-Anweisung gibt Befehle an, die ausgeführt und im neuen Containerimage erfasst werden sollen. Diese Befehle können Elemente wie das Installieren von Software sowie das Erstellen von Dateien, Verzeichnissen und Umgebungskonfigurationen enthalten.

**Format**

Die RUN-Anweisung weist dieses Format auf: 

```none
# exec form

RUN ["<executable", "<param 1>", "<param 2>"]

# shell form

RUN <command>
```

Das Exec- und Shell-Format unterscheiden sich darin, wie die `RUN`-Anweisung ausgeführt wird. Bei Verwendung des Exec-Formats wird das angegebene Programm explizit ausgeführt. 

Im folgenden Beispiel wird das Exec-Format verwendet.

```none
FROM windowsservercore

RUN ["powershell", "New-Item", "c:/test"]
```

Am resultierenden Image ist zu erkennen, dass der Befehl `powershell New-Item c:/test` ausgeführt wurde.

```none
docker history doc-exe-method

IMAGE               CREATED             CREATED BY                    SIZE                COMMENT
b3452b13e472        2 minutes ago       powershell New-Item c:/test   30.76 MB
```

Im Gegensatz dazu wird im folgenden Beispiel der gleiche Vorgang mit dem Shell-Format ausgeführt.

```none
FROM windowsservercore

RUN powershell New-Item c:\test
```

Dies führt zur Ausführungsanweisung `cmd /S /C powershell New-Item c:\test`. 

```none
docker history doc-shell-method

IMAGE               CREATED             CREATED BY                              SIZE                COMMENT
062a543374fc        19 seconds ago      cmd /S /C powershell New-Item c:\test   30.76 MB
```

**Bei Windows zu berücksichtigende Aspekte**
 
Wenn unter Windows die `RUN`-Anweisung mit dem Exec-Format verwendet wird, müssen umgekehrte Schrägstriche mit Escapezeichen versehen werden.

```none
RUN ["powershell", "New-Item", "c:\\test"]
```

Wenn das Zielprogramm ein Windows Installer ist, wird ein zusätzlicher Schritt benötigt, bevor das eigentliche (im Hintergrund ausgeführte) Installationsverfahren gestartet werden kann: Extraktion des Setups mittels des `/x:<directory>`-Flags. Sie müssen die Beendigung der Befehlsausführung abwarten. Andernfalls endet der Vorgang vorzeitig, ohne dass etwas installiert wurde. Weitere Informationen finden Sie im folgenden Beispiel.

**Beispiele**

Dieses Beispiel verwendet DISM zum Installieren von IIS im Containerimage.
```none
RUN dism.exe /online /enable-feature /all /featurename:iis-webserver /NoRestart
```

In diesem Beispiel wird das verteilbare Paket von Visual Studio installiert. Wie Sie sehen, werden `Start-Process` und der Parameter `-Wait` verwendet, um das Installationsprogramm auszuführen. Dadurch wird gewährleistet, dass die Installation abgeschlossen wird, bevor in der Docker-Datei mit dem nächsten Schritt begonnen wird.

```none
RUN Start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait
``` 

Ausführliche Informationen über die RUN-Anweisung finden Sie in der [RUN-Referenz auf Docker.com]( https://docs.docker.com/engine/reference/builder/#run). 

### KOPIEREN

Die `COPY`-Anweisung kopiert Dateien und Verzeichnisse in das Dateisystem des Containers. Die Dateien und Verzeichnisse müssen sich in einem zur Dockerfile-Datei relativen Pfad befinden.

**Format**

Die `COPY`-Anweisung weist dieses Format auf: 

```none
COPY <source> <destination>
``` 

Wenn die Quelle oder das Ziel Leerzeichen enthalten, schließen Sie den Pfad in eckige Klammern und doppelte Anführungszeichen ein.
 
```none
COPY ["<source>", "<destination>"]
```

**Bei Windows zu berücksichtigende Aspekte**
 
Unter Windows müssen im Zielformat Schrägstriche verwendet werden. Dies sind z. B. gültige `COPY`-Anweisungen.

```none
COPY test1.txt /temp/
COPY test1.txt c:/temp/
```

Allerdings wird Folgendes nicht funktionieren.

```none
COPY test1.txt c:\temp\
```

**Beispiele**

In diesem Beispiel wird der Inhalt des Quellverzeichnisses in ein Verzeichnis namens `sqllite` kopiert, das sich im Containerimage befindet.
```none
COPY source /sqlite/
```

In diesem Beispiel werden alle Dateien, die mit „config“ beginnen, dem `c:\temp`-Verzeichnis des Containerimages hinzugefügt.
```none
COPY config* c:/temp/
```

Ausführliche Informationen zur `COPY`-Anweisung finden Sie in der [COPY-Referenz auf Docker.com]( https://docs.docker.com/engine/reference/builder/#copy).

### HINZUFÜGEN

Die ADD-Anweisung ähnelt stark der COPY-Anweisung, bietet jedoch zusätzliche Möglichkeiten. Die `ADD`-Anweisung kann nicht nur Dateien vom Host in das Containerimage kopieren, sondern auch von einem Remotestandort aus mit einer URL-Spezifikation.

**Format**

Die `ADD`-Anweisung weist dieses Format auf: 

```none
ADD <source> <destination>
``` 

Wenn die Quelle oder das Ziel Leerzeichen enthalten, schließen Sie den Pfad in eckige Klammern und doppelte Anführungszeichen ein.
 
```none
ADD ["<source>", "<destination>"]
```

**Bei Windows zu berücksichtigende Aspekte**
 
Unter Windows müssen im Zielformat Schrägstriche verwendet werden. Dies sind z. B. gültige `ADD`-Anweisungen.

```none
ADD test1.txt /temp/
ADD test1.txt c:/temp/
```

Allerdings wird Folgendes nicht funktionieren.

```none
ADD test1.txt c:\temp\
```

Außerdem erweitert die `ADD`-Anweisung unter Linux komprimierte Pakete beim Kopieren. Diese Funktionalität ist in Windows nicht verfügbar.

**Beispiele**

In diesem Beispiel wird der Inhalt des Quellverzeichnisses in ein Verzeichnis namens `sqllite` kopiert, das sich im Containerimage befindet.
```none
ADD source /sqlite/
```

In diesem Beispiel werden alle Dateien, die mit „config“ beginnen, dem `c:\temp`-Verzeichnis des Containerimages hinzugefügt.
```none
ADD config* c:/temp/
```

In diesem Beispiel wird Python für Windows in das `c:\temp`-Verzeichnis des Containerimages heruntergeladen.
```none
ADD https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe /temp/python-3.5.1.exe
```

Ausführliche Informationen über die `ADD`-Anweisung finden Sie in der [ADD-Referenz auf Docker.com]( https://docs.docker.com/engine/reference/builder/#add). 

### WORKDIR

Die `WORKDIR`-Anweisung legt ein Arbeitsverzeichnis für andere Dockerfile-Anweisungen wie z.B. `RUN` und `CMD` sowie das Arbeitsverzeichnis für ausgeführte Instanzen des Containerimages fest.

**Format**

Die `WORKDIR`-Anweisung weist dieses Format auf: 

```none
WORKDIR <path to working directory>
``` 

**Bei Windows zu berücksichtigende Aspekte**

Wenn das Arbeitsverzeichnis unter Windows einen umgekehrten Schrägstrich enthält, muss es mit Escapezeichen versehen werden.

```none
WORKDIR c:\\windows
```

**Beispiele**

```none
WORKDIR c:\\Apache24\\bin
```

Ausführliche Informationen über die `WORKDIR`-Anweisung finden Sie in der [WORKDIR-Referenz auf Docker.com]( https://docs.docker.com/engine/reference/builder/#workdir). 

### CMD

Die `CMD`-Anweisung legt den Standardbefehl fest, der bei der Bereitstellung einer Instanz des Containerimages ausgeführt werden soll. Wenn der Container beispielsweise einen NGINX-Webserver hostet, könnte `CMD` Anweisungen zum Starten des Webservers enthalten, z. B. `nginx.exe`. Wenn mehrere `CMD`-Anweisungen in einer Dockerfile-Datei angegeben sind, wird nur die letzte ausgewertet.

**Format**

Die `CMD`-Anweisung weist dieses Format auf: 

```none
# exec form

CMD ["<executable", "<param>"]

# shell form

CMD <command>
```

**Bei Windows zu berücksichtigende Aspekte**

Unter Windows müssen in Dateipfaden, die in der `CMD`-Anweisung angegeben werden, Schrägstriche oder mit Escapezeichen versehene umgekehrte Schrägstriche `\\` verwendet werden. Dies sind z. B. gültige `CMD`-Anweisungen.

```none
# exec form

CMD ["c:\\Apache24\\bin\\httpd.exe", "-w"]

# shell form

CMD c:\\Apache24\\bin\\httpd.exe -w
```
Allerdings wird Folgendes nicht funktionieren.

```none
CMD c:\Apache24\bin\httpd.exe -w
```

Ausführliche Informationen über die `CMD`-Anweisung finden Sie in der [CMD-Referenz auf Docker.com]( https://docs.docker.com/engine/reference/builder/#cmd). 

## Escapezeichen

Eine Dockerfile-Anweisung muss häufig mehrere Zeilen umfassen. Dazu wird das Escapezeichen verwendet. In einer Dockerfile-Anweisung wird als Escapezeichen standardmäßig ein umgekehrter Schrägstrich verwendet: `\`. Da der umgekehrte Schrägstrich unter Windows auch ein Dateipfadtrennzeichen ist, kann dies problematisch sein. Zum Ändern des Standardescapezeichens kann eine Parser-Anweisung verwendet werden. Weitere Informationen zu Parser-Anweisungen finden Sie im Artikel zu Parser-Anweisungen auf [Docker.com]( https://docs.docker.com/engine/reference/builder/#parser-directives).

Das folgende Beispiel zeigt eine einzelne mehrere Zeilen umfassende RUN-Anweisung, für die das Standardescapezeichen verwendet wird.

```none
FROM windowsservercore

RUN powershell.exe -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
    Remove-Item c:\python-3.5.1.exe -Force
```

Platzieren Sie zum Ändern des Escapezeichens eine Escape-Parser-Anweisung in der ersten Zeile der Dockerfile-Datei. Dies wird im folgenden Beispiel veranschaulicht.

> Beachten Sie, dass nur zwei Werte als Escapezeichen verwendet werden können: `\` und `` ` ``.

```none
# escape=`

FROM windowsservercore

RUN powershell.exe -Command `
    $ErrorActionPreference = 'Stop'; `
    wget https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; `
    Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; `
    Remove-Item c:\python-3.5.1.exe -Force
```

Weitere Informationen zur Escape-Parser-Anweisung finden Sie im Artikel zur Escape-Parser-Anweisung auf [Docker.com]( https://docs.docker.com/engine/reference/builder/#escape).

## PowerShell in Dockerfile

### PowerShell-Befehle

PowerShell-Befehle können mit dem `RUN`-Vorgang in einer Dockerfile-Datei ausgeführt werden. 

```none
FROM windowsservercore

RUN powershell -command Expand-Archive -Path c:\apache.zip -DestinationPath c:\
```

### REST-Aufrufe

PowerShell und der `Invoke-WebRequest`-Befehl können beim Sammeln von Informationen oder Dateien von einem Webdienst hilfreich sein. Beispielsweise könnten Sie das folgende Beispiel verwenden, wenn Sie ein Image erstellen, dass Python enthält.

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> Invoke-WebRequest wird in Nano Server derzeit nicht unterstützt.

Eine weitere Möglichkeit zur Verwendung von PowerShell zum Herunterladen von Dateien während des Prozesses der Imageerstellung ist die Verwendung der .NET WebClient-Bibliothek. Dies kann die Downloadleistung verbessern. Im folgenden Beispiel wird die Python-Software mit der WebClient-Bibliothek heruntergeladen.

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  (New-Object System.Net.WebClient).DownloadFile('https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe','c:\python-3.5.1.exe') ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

> WebClient wird in Nano Server derzeit nicht unterstützt.

### PowerShell-Skripts

In einigen Fällen kann es hilfreich sein, ein Skript in die Container zu kopieren, das während des Prozesses der Imageerstellung verwendet wird, und dann aus dem Container heraus auszuführen. Hinweis: Dies beschränkt jegliche Zwischenspeicherung von Imageebenen und verringert die Lesbarkeit der Dockerfile-Datei.

In diesem Beispiel wird ein Skript vom Buildcomputer mithilfe der `ADD`-Anweisung in den Container kopiert. Dieses Skript wird dann mit der RUN-Anweisung ausgeführt.

```
FROM windowsservercore
ADD script.ps1 /windows/temp/script.ps1
RUN powershell.exe -executionpolicy bypass c:\windows\temp\script.ps1
```

## Docker Build 

Sobald Sie eine Dockerfile-Datei erstellt und auf der Festplatte gespeichert haben, kann `docker build` ausgeführt werden, um das neue Image zu erstellen. Der `docker build`-Befehl unterstützt verschiedene optionale Parameter und einen Pfad zur Dockerfile-Datei. Eine vollständige Dokumentation zu Docker Build einschließlich einer Liste aller Buildoptionen finden Sie in der [Build-Referenz auf Docker.com](https://docs.docker.com/engine/reference/commandline/build/#build).

```none
Docker build [OPTIONS] PATH
```
Der folgende Befehl erstellt z. B. ein Image mit dem Namen „iis“.

```none
docker build -t iis .
```

Wenn der Buildprozess eingeleitet wurde, wird der Status ausgegeben, und ggf. werden Fehlermeldungen zurückgegeben.

```none
C:\> docker build -t iis .

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM windowsservercore
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

Das Ergebnis ist ein neues Containerimage, das in diesem Beispiel den Namen „iis“ trägt.

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
iis                 latest              e2aafdfbe392        About a minute ago   207.8 MB
windowsservercore   latest              6801d964fda5        4 months ago         0 B
```

## Weitere Informationen und Referenzen

[Optimize Windows Dockerfiles (Optimieren von Windows-Dockerfile-Dateien und Docker Build)] (./optimize_windows_dockerfile.md)

[Dockerfile-Referenz auf Docker.com](https://docs.docker.com/engine/reference/builder/)



<!--HONumber=Nov16_HO1-->


