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
ms.openlocfilehash: 056ab87189e8e423df5758be0f622a43b92c9056
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882953"
---
# <a name="optimize-windows-dockerfiles"></a>Optimieren von Windows-Dockerfile-Dateien

Es gibt viele Möglichkeiten, den andocker-Buildprozess und die resultierenden Andock Bilder zu optimieren. In diesem Artikel wird erläutert, wie der Andock Erstellungsprozess funktioniert und wie Bilder für Windows-Container optimal erstellt werden.

## <a name="image-layers-in-docker-build"></a>Bildebenen im docker-Build

Bevor Sie Ihren andocker-Build optimieren können, müssen Sie wissen, wie der andocker-Build funktioniert. Während des Docker Build-Prozesses wird eine Dockerfile-Datei genutzt, und die ausführbaren Anweisungen werden nacheinander ausgeführt, jede in einem eigenen temporären Container. Das Ergebnis ist eine neue Imageebene für jede ausführbare Anweisung.

Im folgenden Beispiel Dockerfile wird beispielsweise das `windowsservercore` Basisbetriebssystem-Abbild verwendet, IIS installiert und dann eine einfache Website erstellt.

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Möglicherweise werden Sie davon ausgehen, dass diese Dockerfile ein Bild mit zwei Layern erstellt, eines für das Betriebssystemabbild des Containers und eine Sekunde, die IIS und die Website umfasst. Das tatsächliche Bild weist jedoch viele Ebenen auf, und jede Ebene hängt von der vorhergehenden Schicht ab.

Um dies übersichtlicher zu gestalten, führen wir den `docker history` Befehl für das Bild aus, das von unserem Beispiel Dockerfile erstellt wurde.

```dockerfile
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

Die Ausgabe zeigt an, dass dieses Bild vier Ebenen hat: die Basisebene und drei zusätzliche Ebenen, die jeder Anweisung im Dockerfile zugeordnet sind. Die unterste Ebene (`6801d964fda5` in diesem Beispiel) stellt das Basis-Betriebssystemimage dar. Eine Ebene nach oben ist die IIS-Installation. Die nächste Ebene enthält die neue Website usw.

Dockerfiles können geschrieben werden, um Bildebenen zu minimieren, die Erstellungs Leistung zu optimieren und die Barrierefreiheit durch Lesbarkeit zu optimieren. Schließlich stehen viele Alternativen zur Durchführung einer bestimmten Buildaufgabe zur Verfügung. Wenn Sie wissen, wie sich das Format des Dockerfile auf die Erstellungszeit und das erstellte Bild auswirkt, verbessert dies die Automatisierungs Funktionalität.

## <a name="optimize-image-size"></a>Optimieren der Bildgröße

Je nach Platzbedarf kann die Bildgröße ein wichtiger Faktor beim Erstellen von Andockcontainer Bildern sein. Containerimages werden zwischen Registrierungen und Host verschoben, exportiert und importiert und verbrauchen so letztendlich Speicherplatz. In diesem Abschnitt erfahren Sie, wie Sie die Bildgröße während des Andock Vorgangs für Windows-Container minimieren.

Weitere Informationen zu den bewährten Methoden von Dockerfile finden Sie unter [bewährte Methoden zum Schreiben von Dockerfiles auf Docker.com](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/).

### <a name="group-related-actions"></a>Gruppieren von verwandten Aktionen

Da jede `RUN` Anweisung eine neue Ebene im Container Bild erstellt, kann durch Gruppieren von Aktionen `RUN` in einer Anweisung die Anzahl der Ebenen in einem Dockerfile verringert werden. Das Minimieren der Ebenen wirkt sich möglicherweise nicht auf die Imagegröße aus, aber das Gruppieren verwandter Aktionen kann eine Einfluss haben, wie die nachfolgenden Beispiele zeigen.

In diesem Abschnitt vergleichen wir zwei Beispiel Dockerfiles, die die gleichen Aufgaben ausführen. Ein Dockerfile verfügt jedoch über eine Anweisung pro Aktion, während die anderen zugehörigen Aktionen zusammen gruppiert wurden.

Das folgende, nicht gruppierte Beispiel Dockerfile downloadet python für Windows, installiert es und entfernt die heruntergeladene Setup-Datei, sobald die Installation abgeschlossen ist. In diesem Dockerfile erhält jede Aktion eine eigene `RUN` Anweisung.

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

Das zweite Beispiel ist ein Dockerfile, das exakt denselben Vorgang ausführt. Allerdings wurden alle zugehörigen Aktionen unter einer einzelnen `RUN` Anweisung gruppiert. Jeder Schritt in der `RUN` Anweisung befindet sich in einer neuen Zeile des Dockerfile, während das "\ \"-Zeichen zum Zeilenumbruch verwendet wird.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

Das resultierende Bild hat nur eine zusätzliche Ebene für `RUN` die Anweisung.

```dockerfile
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB
```

### <a name="remove-excess-files"></a>Entfernen überflüssiger Dateien

Wenn sich in Ihrem Dockerfile eine Datei befindet, beispielsweise ein Installationsprogramm, die Sie nach der Verwendung nicht benötigen, können Sie Sie entfernen, um die Bildgröße zu verringern. Dies muss in dem gleichen Schritt erfolgen, in dem die Datei in die Imageebene kopiert wurde. Dadurch wird verhindert, dass die Datei in einer Bildebene auf niedrigerer Ebene beibehalten wird.

Im folgenden Beispiel Dockerfile wird das Python-Paket heruntergeladen, ausgeführt und dann entfernt. Dies alles wird in einem einzigen `RUN`-Vorgang ausgeführt und resultiert in einer einzelnen Imageebene.

```dockerfile
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## <a name="optimize-build-speed"></a>Optimieren der Build-Geschwindigkeit

### <a name="multiple-lines"></a>Mehrere Zeilen

Sie können Vorgänge in mehrere einzelne Anweisungen aufteilen, um die Andock Geschwindigkeit zu optimieren. Mehrere `RUN` Vorgänge erhöhen die Effektivität der Zwischenspeicherung, da für `RUN` jede Anweisung einzelne Ebenen erstellt werden. Wenn eine identische Anweisung bereits in einem anderen Andock Erstellungsvorgang ausgeführt wurde, wird dieser zwischengespeicherte Vorgang (Bildebene) wieder verwendet, was zu einer verminderten Andock Erstellungs Laufzeit führt.

Im folgenden Beispiel werden sowohl der Apache als auch die Visual Studio-Pakete neu verteilt, heruntergeladen, installiert und anschließend bereinigt, indem Dateien entfernt werden, die nicht mehr benötigt werden. Dies erfolgt mit einer einzelnen `RUN` Anweisung. Wenn eine dieser Aktionen aktualisiert wird, werden alle Aktionen erneut ausgeführt.

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

Das resultierende Bild hat zwei Ebenen, eines für das Basisbetriebssystem-Bild und eines, das alle Vorgänge aus der `RUN` einzelnen Anweisung enthält.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

Im Vergleich dazu sind die gleichen Aktionen in drei `RUN` Anweisungen aufgeteilt. In diesem Fall wird jede `RUN` Anweisung in einem Container Bild-Layer zwischengespeichert, und nur diejenigen, die sich geändert haben, müssen bei nachfolgenden Dockerfile-Builds erneut ausgeführt werden.

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

Das resultierende Bild besteht aus vier Ebenen; eine Ebene für das Basisbetriebssystem-Abbild und jede der drei `RUN` Anweisungen. Da jede `RUN` Anweisung in einer eigenen Ebene ausgeführt wird, verwenden alle nachfolgenden Ausführungen dieser Dockerfile oder identischen Anweisungen in einem anderen Dockerfile zwischengespeicherte Bildebenen, wodurch die Erstellungszeit verkürzt wird.

```dockerfile
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

Die Reihenfolge der Anweisungen ist wichtig, wenn Sie mit Bild Caches arbeiten, wie im nächsten Abschnitt zu sehen ist.

### <a name="ordering-instructions"></a>Bestellhinweise

Eine Dockerfile-Datei wird von oben nach unten verarbeitet und jede Anweisung mit den zwischengespeicherten Ebenen verglichen. Wenn eine Anweisung gefunden wird, der keine zwischengespeicherte Ebene zugeordnet ist, werden diese Anweisung und alle nachfolgenden Anweisungen in neuen Containerimageebenen verarbeitet. Aus diesem Grund ist die Reihenfolge wichtig, in der die Anweisungen platziert werden. Platzieren Sie Anweisungen, die konstant bleiben werden, im oberen Teil der Dockerfile-Datei. Platzieren Sie Anweisungen, die sich ändern könnten, im unteren Teil der Dockerfile-Datei. Dies reduziert die Wahrscheinlichkeit, dass der vorhandene Cache negiert wird.

In den folgenden Beispielen wird gezeigt, wie sich die Dockerfile-Anweisungsreihenfolge auf die Zwischenspeicherung auswirkt. Dieses einfache Beispiel Dockerfile hat vier nummerierte Ordner.  

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```

Das resultierende Bild weist fünf Ebenen auf, eines für das Basisbetriebssystem-Bild und `RUN` jede der Anweisungen.

```dockerfile
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B
```

Dieser nächste Dockerfile wurde nun leicht geändert, wobei die dritte `RUN` Anweisung in eine neue Datei geändert wurde. Wenn Docker Build mit dieser Dockerfile-Datei ausgeführt wird, verwenden die ersten drei Anweisungen, die mit denen im letzten Beispiel identisch sind, zwischengespeicherte Imageebenen. Da die geänderte `RUN` Anweisung aber nicht zwischengespeichert wird, wird für die geänderte Anweisung und alle nachfolgenden Anweisungen ein neuer Layer erstellt.

```dockerfile
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Wenn Sie die Bild-IDs des neuen Bilds mit dem im ersten Beispiel dieses Abschnitts vergleichen, werden Sie feststellen, dass die ersten drei Ebenen von unten nach oben freigegeben sind, die vierte und die fünfte jedoch eindeutig sind.

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

Bei Dockerfile-Anweisungen wird die Groß-/Kleinschreibung nicht unterschieden, aber die Konvention ist in Großbuchstaben zu verwenden. Dies verbessert die Lesbarkeit, da zwischen dem Anweisungsaufruf und dem Anweisungs Vorgang unterschieden wird. Die folgenden beiden Beispiele vergleichen eine nicht aktivierte und groß geschriebene Dockerfile.

Der folgende Code ist ein nicht aktivierter Dockerfile:

```dockerfile
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```

Im folgenden ist der gleiche Dockerfile in Großbuchstaben zu verwenden:

```dockerfile
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### <a name="line-wrapping"></a>Zeilenumbruch

Lange und komplexe Vorgänge können mit dem umgekehrten Schrägstrich `\` auf mehrere Zeilen aufgeteilt werden. Die folgende Dockerfile-Datei installiert das verteilbare Paket von Visual Studio, entfernt die Dateien des Installationsprogramms und erstellt dann eine Konfigurationsdatei. Alle diese drei Vorgänge werden in einer Zeile angegeben.

```dockerfile
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```

Der Befehl kann mit umgekehrten Schrägstrichen aufgeteilt werden, damit jeder Vorgang aus der `RUN` einen Anweisung in einer eigenen Zeile angegeben wird.

```dockerfile
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## <a name="further-reading-and-references"></a>Weitere Informationen und Verweise

[Dockerfile unter Windows](manage-windows-dockerfile.md)

[Bewährte Methoden zum Schreiben von Dockerfiles auf Docker.com](https://docs.docker.com/engine/reference/builder/)
