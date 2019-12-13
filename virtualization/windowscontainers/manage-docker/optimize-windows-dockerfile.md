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
ms.openlocfilehash: ae633c7ba5d9672335addcc582988fc47c13ed79
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910150"
---
# <a name="optimize-windows-dockerfiles"></a>Optimieren von Windows-Dockerfile-Dateien

Es gibt zahlreiche Möglichkeiten, den docker-Buildprozess und die resultierenden docker-Images zu optimieren. In diesem Artikel wird erläutert, wie der Docker-Buildprozess funktioniert und wie Images für Windows-Container optimal erstellt werden.

## <a name="image-layers-in-docker-build"></a>Bildebenen in docker Build

Bevor Sie Ihren docker-Build optimieren können, müssen Sie wissen, wie docker Build funktioniert. Während des Docker Build-Prozesses wird eine Dockerfile-Datei genutzt, und die ausführbaren Anweisungen werden nacheinander ausgeführt, jede in einem eigenen temporären Container. Das Ergebnis ist eine neue Imageebene für jede ausführbare Anweisung.

Beispielsweise verwendet die folgende dockerfile-Beispieldatei das `mcr.microsoft.com/windows/servercore:ltsc2019` Basis-Betriebssystem Image, installiert IIS und erstellt dann eine einfache Website.

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Sie erwarten möglicherweise, dass diese dockerfile-Datei ein Image mit zwei Ebenen erstellt, eines für das Container-Betriebssystem Image und ein zweites, das IIS und die Website enthält. Das tatsächliche Image weist jedoch viele Ebenen auf, und jede Ebene hängt von der jeweils vorhandenen Ebene ab.

Um dies zu verdeutlichen, führen wir den `docker history` Befehl für das Image aus, das in der dockerfile-Beispieldatei erstellt wurde.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

Die Ausgabe zeigt, dass dieses Bild vier Ebenen hat: die Basisschicht und drei zusätzliche Ebenen, die jeder Anweisung in der dockerfile-Datei zugeordnet werden. Die unterste Ebene (`6801d964fda5` in diesem Beispiel) stellt das Basis-Betriebssystemimage dar. Eine Ebene ist die IIS-Installation. Die nächste Ebene enthält die neue Website usw.

Dockerfiles können erstellt werden, um Bildebenen zu minimieren, die Buildleistung zu optimieren und die Barrierefreiheit durch Lesbarkeit zu optimieren. Schließlich stehen viele Alternativen zur Durchführung einer bestimmten Buildaufgabe zur Verfügung. Wenn Sie verstehen, wie sich das Format der dockerfile-Datei auf die Buildzeit und das erstellte Image auswirkt, verbessert sich die Automatisierung

## <a name="optimize-image-size"></a>Bild Größe optimieren

Abhängig von den Speicherplatzanforderungen kann die Image Größe beim Aufbau von Docker-Container Images ein wichtiger Faktor sein. Containerimages werden zwischen Registrierungen und Host verschoben, exportiert und importiert und verbrauchen so letztendlich Speicherplatz. In diesem Abschnitt erfahren Sie, wie Sie die Image Größe während des docker-Buildprozesses für Windows-Container minimieren.

Weitere Informationen zu bewährten Methoden für dockerfile finden Sie unter [bewährte Methoden zum Schreiben von dockerfiles auf Docker.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Gruppieren von verwandten Aktionen

Da jede `RUN` Anweisung eine neue Ebene im Container Image erstellt, kann die Gruppierung von Aktionen in einer `RUN` Anweisung die Anzahl der Ebenen in einer dockerfile-Datei reduzieren. Das Minimieren der Ebenen wirkt sich möglicherweise nicht auf die Imagegröße aus, aber das Gruppieren verwandter Aktionen kann eine Einfluss haben, wie die nachfolgenden Beispiele zeigen.

In diesem Abschnitt werden zwei docker-Beispieldateien verglichen, die dieselben Aktionen ausführen. Allerdings hat eine dockerfile-Datei eine Anweisung pro Aktion, während bei der anderen die zugehörigen Aktionen gruppiert wurden.

In der folgenden nicht gruppierten dockerfile-Beispieldatei wird python für Windows heruntergeladen, installiert und die heruntergeladene Setup Datei entfernt, sobald die Installation abgeschlossen ist. In dieser dockerfile-Datei erhält jede Aktion eine eigene `RUN` Anweisung.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

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

Das zweite Beispiel ist eine dockerfile-Datei, die genau denselben Vorgang ausführt. Alle zugehörigen Aktionen wurden jedoch unter einer einzigen `RUN` Anweisung gruppiert. Jeder Schritt in der `RUN` Anweisung befindet sich in einer neuen Zeile der dockerfile-Datei, während das Zeichen '\\' für den Zeilenumbruch verwendet wird.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

Das resultierende Bild verfügt nur über eine zusätzliche Ebene für die `RUN` Anweisung.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Entfernen überflüssiger Dateien

Wenn eine Datei in ihrer dockerfile-Datei vorhanden ist, z. b. ein Installer, die Sie nicht benötigen, nachdem Sie verwendet wurde, können Sie Sie entfernen, um die Image Größe zu verringern. Dies muss in dem gleichen Schritt erfolgen, in dem die Datei in die Imageebene kopiert wurde. Dadurch wird verhindert, dass die Datei in einer Bild Ebene auf niedrigerer Ebene beibehalten wird.

In der folgenden dockerfile-Beispieldatei wird das Python-Paket heruntergeladen, ausgeführt und dann entfernt. Dies alles wird in einem einzigen `RUN`-Vorgang ausgeführt und resultiert in einer einzelnen Imageebene.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>Optimieren der Buildgeschwindigkeit

### <a name="multiple-lines"></a>Mehrere Zeilen

Sie können Vorgänge in mehrere einzelne Anweisungen aufteilen, um die Docker-Buildgeschwindigkeit zu optimieren. Mehrere `RUN` Vorgänge erhöhen die Effektivität der Zwischenspeicherung, da für jede `RUN` Anweisung einzelne Ebenen erstellt werden. Wenn eine identische Anweisung bereits in einem anderen docker-Buildvorgang ausgeführt wurde, wird dieser zwischengespeicherte Vorgang (Bild Ebene) wieder verwendet und führt zu einer verringerten docker Build-Laufzeit.

Im folgenden Beispiel werden sowohl das Apache-als auch das Visual Studio-Paket neu verteilt heruntergeladen, installiert und dann bereinigt, indem Dateien entfernt werden, die nicht mehr benötigt werden. Dies erfolgt mit einer einzigen `RUN` Anweisung. Wenn eine dieser Aktionen aktualisiert wird, werden alle Aktionen erneut ausgeführt.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

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

Das resultierende Bild verfügt über zwei Ebenen, eine für das Basis Betriebssystem-Image und eine, die alle Vorgänge der einzelnen `RUN` Anweisung enthält.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

Im Vergleich dazu werden die gleichen Aktionen in drei `RUN` Anweisungen aufgeteilt. In diesem Fall wird jede `RUN` Anweisung in einer Container Image Ebene zwischengespeichert, und nur solche, die geändert wurden, müssen bei nachfolgenden dockerfile-Builds erneut ausgeführt werden.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

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

Das resultierende Bild besteht aus vier Ebenen. eine Ebene für das Basis Betriebssystem-Image und jede der drei `RUN` Anweisungen. Da jede `RUN` Anweisung in einer eigenen Schicht ausgeführt wurde, verwenden alle nachfolgenden Ausführungen dieser dockerfile-Datei oder identische Anweisungen in einer anderen dockerfile-Datei zwischengespeicherte Bildebenen, wodurch die Buildzeit reduziert wird.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

Wie Sie die Anweisungen anordnen, ist wichtig, wenn Sie mit Abbild Caches arbeiten, wie Sie im nächsten Abschnitt sehen werden.

### <a name="ordering-instructions"></a>Bestell Anweisungen

Eine Dockerfile-Datei wird von oben nach unten verarbeitet und jede Anweisung mit den zwischengespeicherten Ebenen verglichen. Wenn eine Anweisung gefunden wird, der keine zwischengespeicherte Ebene zugeordnet ist, werden diese Anweisung und alle nachfolgenden Anweisungen in neuen Containerimageebenen verarbeitet. Aus diesem Grund ist die Reihenfolge wichtig, in der die Anweisungen platziert werden. Platzieren Sie Anweisungen, die konstant bleiben werden, im oberen Teil der Dockerfile-Datei. Platzieren Sie Anweisungen, die sich ändern könnten, im unteren Teil der Dockerfile-Datei. Dies reduziert die Wahrscheinlichkeit, dass der vorhandene Cache negiert wird.

In den folgenden Beispielen wird veranschaulicht, wie die Reihenfolge der dockerfile-Anweisungen Auswirkungen auf das Zwischenspeichern Diese einfache dockerfile-Beispieldatei hat vier nummerierte Ordner.  

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

Das resultierende Image verfügt über fünf Ebenen, eine für das Basis Betriebssystem-Image und jede der `RUN` Anweisungen.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

Diese nächste dockerfile-Datei wurde nun etwas geändert, wobei die dritte `RUN` Anweisung in eine neue Datei geändert wurde. Wenn Docker Build mit dieser Dockerfile-Datei ausgeführt wird, verwenden die ersten drei Anweisungen, die mit denen im letzten Beispiel identisch sind, zwischengespeicherte Imageebenen. Da die geänderte `RUN` Anweisung jedoch nicht zwischengespeichert wird, wird für die geänderte Anweisung und alle nachfolgenden Anweisungen eine neue Ebene erstellt.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Wenn Sie die Image-IDs des neuen Bilds mit der im ersten Beispiel dieses Abschnitts vergleichen, werden Sie feststellen, dass die ersten drei Ebenen von unten nach oben freigegeben sind, aber das vierte und fünfte sind eindeutig.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## <a name="cosmetic-optimization"></a>Kosmetische Optimierung

### <a name="instruction-case"></a>Anweisungs Fall

In den Anweisungen der dockerfile-Datei wird die Groß-/Kleinschreibung nicht beachtet, aber es wird in Großbuchstaben verwendet. Dadurch wird die Lesbarkeit verbessert, indem der Anweisungs-und Anweisungs Vorgang unterschieden wird. Die folgenden beiden Beispiele vergleichen eine nicht groß geschriebene und groß geschriebene dockerfile-Datei.

Im folgenden finden Sie eine nicht groß geschriebene dockerfile-Datei:

```dockerfile
# Sample Dockerfile

from mcr.microsoft.com/windows/servercore:ltsc2019
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

Im folgenden finden Sie eine dockerfile-Datei, die in Großbuchstaben verwendet wird:

```dockerfile
# Sample Dockerfile

FROM mcr.microsoft.com/windows/servercore:ltsc2019
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Zeilen umschluss

Lange und komplexe Vorgänge können durch den umgekehrten Schrägstrich `\` Zeichen auf mehrere Zeilen aufgeteilt werden. Die folgende Dockerfile-Datei installiert das verteilbare Paket von Visual Studio, entfernt die Dateien des Installationsprogramms und erstellt dann eine Konfigurationsdatei. Alle diese drei Vorgänge werden in einer Zeile angegeben.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

Der Befehl kann mit umgekehrten Schrägstrichen untergliedert werden, sodass jeder Vorgang von einer `RUN` Anweisung in einer eigenen Zeile angegeben wird.

```dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Weitere Informationen und Verweise

[Dockerfile unter Windows](manage-windows-dockerfile.md)

[Bewährte Methoden zum Schreiben von dockerfiles auf Docker.com](https://docs.docker.com/engine/reference/builder/)
