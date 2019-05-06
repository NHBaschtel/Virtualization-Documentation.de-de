---
title: Optimieren von Windows-Dockerfile-Dateien
description: Optimieren Sie Dockerfile-Dateien für Windows-Container.
keywords: Docker, Container
author: cwilhit
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
ms.openlocfilehash: d897560061fae23fda6f88ebdad6dd804da9a8f1
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610340"
---
# <a name="optimize-windows-dockerfiles"></a>Optimieren von Windows-Dockerfile-Dateien

Es gibt viele Möglichkeiten zur Optimierung des Docker Build-Prozesses sowie der resultierenden Docker-Images. In diesem Artikel wird erläutert, die Funktionsweise des Docker Build-Prozesses und optimal Images für Windows-Container erstellen.

## <a name="image-layers-in-docker-build"></a>Imageebenen in Docker build

Bevor Sie Ihre Docker Build optimieren können, müssen Sie wissen, wie Docker build funktioniert. Während des Docker Build-Prozesses wird eine Dockerfile-Datei genutzt, und die ausführbaren Anweisungen werden nacheinander ausgeführt, jede in einem eigenen temporären Container. Das Ergebnis ist eine neue Imageebene für jede ausführbare Anweisung.

Im folgenden Beispiel z. B. dockerfile-Datei verwendet die `windowsservercore` Basisimage des Betriebssystems, IIS installiert und dann eine einfache Website erstellt.

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Sie erwarten, dass dieser dockerfile-Datei ein Bild mit zwei Ebenen, einer für das Container-Betriebssystemimage und eine zweite, die IIS und die Website enthält erzeugen. Allerdings das tatsächliche Image besteht aus vielen Ebenen, und jede Ebene hängt von vorherigen.

Um dies deutlicher zu machen, führen wir die `docker history` Befehl gegen das Bild unserem Beispiel Dockerfile vorgenommen.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

Die Ausgabe zeigt uns, dass dieses Image besteht aus vier Ebenen: der Grundebene und drei zusätzlichen Ebenen, die jede Anweisung in der dockerfile-Datei zugeordnet sind. Die unterste Ebene (`6801d964fda5` in diesem Beispiel) stellt das Basis-Betriebssystemimage dar. Eine Ebene nach oben ist die IIS-Installation. Die nächste Ebene enthält die neue Website usw.

Dockerfile-Dateien können geschrieben werden, um imageebenen zu minimieren, Buildleistung und Eingabehilfen über besseren Lesbarkeit zu optimieren. Schließlich stehen viele Alternativen zur Durchführung einer bestimmten Buildaufgabe zur Verfügung. Grundlegendes zu wie die Dockerfile-Format wirkt sich auf die Build-Dauer und das Bild erstellt wird, verbessert die automatisierungserfahrung.

## <a name="optimize-image-size"></a>Optimieren der Imagegröße

Je nach Ihren Anforderungen Speicherplatz kann Imagegröße ein wichtiger Faktor sein, wenn Sie Docker-containerimages erstellen. Containerimages werden zwischen Registrierungen und Host verschoben, exportiert und importiert und verbrauchen so letztendlich Speicherplatz. In diesem Abschnitt erfahren Sie, wie Sie während des Buildprozesses Docker für Windows-Container Imagegröße zu minimieren.

Weitere Informationen zu bewährten Vorgehensweisen mit Dockerfile finden Sie [bewährte Methoden zum Schreiben von Dockerfiles auf Docker.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Gruppieren von verwandten Aktionen

Da jedes `RUN` -Anweisung erstellt eine neue Ebene in der Container-Image, die Gruppierung von Aktionen in einer `RUN` -Anweisung die Anzahl der Ebenen in einer dockerfile-Datei reduzieren kann. Das Minimieren der Ebenen wirkt sich möglicherweise nicht auf die Imagegröße aus, aber das Gruppieren verwandter Aktionen kann eine Einfluss haben, wie die nachfolgenden Beispiele zeigen.

In diesem Abschnitt Vergleichen wir zwei Beispiel dockerfile-Dateien, die das gleiche tun. Eine dockerfile-Datei verfügt jedoch über eine Anweisung pro Aktion aus, während die andere die zugehörigen Aktivitäten gruppiert hatten.

Die dockerfile-Datei im folgenden aufgehoben wird Python für Windows, Installation und entfernt die heruntergeladenen Setupdatei, sobald die Installation abgeschlossen ist. In dieser dockerfile-Datei, jede Aktion erhält eine eigene `RUN` Anweisung.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

Das resultierende Image besteht aus drei zusätzliche Ebenen, einer für jede `RUN`-Anweisung.

```dockerfile
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

Im zweite Beispiel wird eine dockerfile-Datei, die die exakt gleiche durchgeführt. Jedoch alle zugehörigen Aktionen mit einer einzelnen gruppiert wurde haben `RUN` Anweisung. Jeder Schritt in die `RUN` -Anweisung wird in einer neuen Zeile der dockerfile-Datei, während die ' \\' Zeichen Zeilenumbruch verwendet wird.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

Das resultierende Image besteht nur einer zusätzliche Ebene für die `RUN` Anweisung.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Entfernen überflüssiger Dateien

Wenn vorhanden ist, wird eine Datei in die dockerfile-Datei, z. B. ein Installationsprogramm, die Sie danach müssen nicht verwendet wurde, entfernen Sie darauf, um die Imagegröße zu verringern. Dies muss in dem gleichen Schritt erfolgen, in dem die Datei in die Imageebene kopiert wurde. Dadurch wird verhindert, dass die Datei auf einer niedrigeren Ebene imageebene beibehalten.

Im folgenden Beispiel dockerfile-Datei die Python-Paket ist heruntergeladen, ausgeführt und dann entfernt. Dies alles wird in einem einzigen `RUN`-Vorgang ausgeführt und resultiert in einer einzelnen Imageebene.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>Optimieren der Buildgeschwindigkeit

### <a name="multiple-lines"></a>Mehrere Zeilen

Sie können Vorgänge in mehrere einzelne Anweisungen zur Optimierung des Docker Build-Geschwindigkeit teilen. Mehrere `RUN` -Vorgänge steigern die Effizienz des Zwischenspeicherns, da für jede einzelne Ebenen erstellt werden `RUN` Anweisung. Wenn eine identische Anweisung bereits in einem anderen Docker Build-Vorgang ausgeführt wurde, wird dieser zwischengespeicherte Vorgang (imageebene) wiederverwendet, was verringerte Docker Build-Laufzeit.

Im folgenden Beispiel wird werden sowohl Apache als auch die Visual Studio verteilbaren Pakete heruntergeladen, installiert und dann durch das Entfernen von Dateien, die nicht mehr benötigt werden bereinigt. Dies alles erfolgt mit einem einzigen `RUN` Anweisung. Wenn eine dieser Aktionen aktualisiert wird, werden alle Aktionen erneut ausführen.

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    
  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    
  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force; \
  Remove-Item c:\php.zip
```

Das resultierende Image besteht, zwei Ebenen, eine für das Basisbetriebssystem-Image und enthält alle Vorgänge aus der einzelnen `RUN` Anweisung.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

Hier sind im Vergleich dieselbe Aktionen Aufteilung in drei `RUN` Anweisungen. In diesem Fall wird jede `RUN` -Anweisung in einer containerimageebene zwischengespeichert, und nur solche, die geänderte müssen erneut ausgeführt werden, auf die nachfolgenden Dockerfile-builds.

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
    Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
    Remove-Item c:\apache.zip -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
    start-Process c:\vcredist.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist.exe -Force

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    wget http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
    Remove-Item c:\php.zip -Force
```

Das resultierende Image besteht aus vier Ebenen. eine Ebene für das Basisbetriebssystem-Image und die drei `RUN` Anweisungen. Da jedes `RUN` -Anweisung in einer eigenen Ebene ausgeführt wurde, verwenden alle nachfolgenden Ausführungen dieser dockerfile-Datei oder eines identischen Satzes von Anweisungen in einer anderen Dockerfile zwischengespeicherte imageebenen, die Buildzeit reduziert.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

Wie Sie die Anweisungen bestellen ist wichtig, bei der Arbeit mit Bild Caches, wie Sie im nächsten Abschnitt sehen werden.

### <a name="ordering-instructions"></a>Anordnung von Anweisungen

Eine Dockerfile-Datei wird von oben nach unten verarbeitet und jede Anweisung mit den zwischengespeicherten Ebenen verglichen. Wenn eine Anweisung gefunden wird, der keine zwischengespeicherte Ebene zugeordnet ist, werden diese Anweisung und alle nachfolgenden Anweisungen in neuen Containerimageebenen verarbeitet. Aus diesem Grund ist die Reihenfolge wichtig, in der die Anweisungen platziert werden. Platzieren Sie Anweisungen, die konstant bleiben werden, im oberen Teil der Dockerfile-Datei. Platzieren Sie Anweisungen, die sich ändern könnten, im unteren Teil der Dockerfile-Datei. Dies reduziert die Wahrscheinlichkeit, dass der vorhandene Cache negiert wird.

Die folgenden Beispiele zeigen, wie die Reihenfolge von Dockerfile-Anweisung Zwischenspeichers beeinflussen kann. In diesem einfache Beispiel dockerfile-Datei verfügt über vier nummerierte Ordner.  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

Das resultierende Image besteht aus fünf Ebenen, eine für das Basisbetriebssystem-Image und jedes der `RUN` Anweisungen.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

Dieser nächsten dockerfile-Datei wurde jetzt etwas geändert, mit der dritten `RUN` -Anweisung geändert wird, um eine neue Datei. Wenn Docker Build mit dieser Dockerfile-Datei ausgeführt wird, verwenden die ersten drei Anweisungen, die mit denen im letzten Beispiel identisch sind, zwischengespeicherte Imageebenen. Aber da die geänderte `RUN` -Anweisung nicht zwischengespeichert, wird eine neue Ebene für für die geänderte Anweisung und alle nachfolgenden Anweisungen erstellt.

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Wenn Sie die Image-IDs des neuen Bilds zu, die im ersten Beispiel in diesem Abschnitt vergleichen, werden Sie feststellen, dass die ersten drei Ebenen von unten nach oben werden gemeinsam verwendet, aber die vierte und fünfte eindeutig sind.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>Optische Optimierung

### <a name="instruction-case"></a>/ Kleinschreibung der Anweisung

Dockerfile-Anweisungen wird nicht zwischen Groß-/Kleinschreibung berücksichtigt, aber die Konvention ist die Verwendung von Großbuchstaben. Dies verbessert die Lesbarkeit durch die Unterscheidung zwischen Anweisungsaufruf und anweisungsvorgang. Die folgenden beiden Beispiele vergleichen eine uncapitalized und Großbuchstaben Dockerfile.

Der folgende Code ist ein uncapitalized dockerfile-Datei:

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

Im folgenden finden die gleichen dockerfile-Datei mit groß-:

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Zeilenumbruch

Lange und komplexe Vorgänge in mehrere Zeilen getrennt werden können, indem der umgekehrte Schrägstrich `\` Zeichen. Die folgende Dockerfile-Datei installiert das verteilbare Paket von Visual Studio, entfernt die Dateien des Installationsprogramms und erstellt dann eine Konfigurationsdatei. Alle diese drei Vorgänge werden in einer Zeile angegeben.

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

Der Befehl kann aufzuteilen umgekehrte Schrägstriche also, die jeden Vorgang der einen `RUN` -Anweisung in einer eigenen Zeile angegeben ist.

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Weitere Informationen und Referenzen

[Dockerfile unter Windows](manage-windows-dockerfile.md)

[Bewährte Methoden zum Schreiben von Dockerfiles auf Docker.com](https://docs.docker.com/engine/reference/builder/)
