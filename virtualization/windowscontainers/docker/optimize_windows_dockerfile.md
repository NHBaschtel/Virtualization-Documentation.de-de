---
title: Optimieren von Windows-Dockerfile-Dateien
description: "Optimieren Sie Dockerfile-Dateien für Windows-Container."
keywords: Docker, Container
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb2848ca-683e-4361-a750-0d1d14ec8031
translationtype: Human Translation
ms.sourcegitcommit: 7ebd83d5d3a098fc8760f5dfba7e350c3f167232
ms.openlocfilehash: 19a363aa013b51e0c80d56572de77e94f27e546f

---
# Optimieren von Windows-Dockerfile-Dateien

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

Mehrere Methoden können zur Optimierung des Docker Build-Prozesses sowie der resultierenden Docker-Images verwendet werden. Dieses Dokument beschreibt ausführlich den Ablauf des Docker Build-Prozesses und zeigt verschiedene Taktiken, die für eine optimale Imageerstellung mit Windows-Containern verwendet werden können.

## Docker Build

### Imageebenen

Unverzichtbare Voraussetzung zur Untersuchung der Docker Build-Optimierung ist ein grundlegendes Verständnis der Funktionsweise von Docker Build. Während des Docker Build-Prozesses wird eine Dockerfile-Datei genutzt, und die ausführbaren Anweisungen werden nacheinander ausgeführt, jede in einem eigenen temporären Container. Das Ergebnis ist eine neue Imageebene für jede ausführbare Anweisung. 

Betrachten Sie die folgende Dockerfile-Datei. In diesem Beispiel wird das `windowsservercore`-Basis-Betriebssystemimage verwendet, IIS installiert und eine einfache Website erstellt.

```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

Sie erwarten vielleicht von dieser Dockerfile-Datei ausgehend, dass das resultierende Image aus zwei Ebenen besteht, eine für das Container-Betriebssystemimage und eine zweite, die IIS und die Website enthält, jedoch ist dies nicht der Fall. Das neue Image besteht aus vielen Ebenen, wobei jede jeweils von der vorhergehenden abhängt. Um dies zu visualisieren, kann der `docker history`-Befehl auf das neue Image angewandt werden. Dies zeigt, dass das Image aus vier Ebenen besteht, der Basis und drei zusätzlichen Ebenen, eine für jede Anweisung in der Dockerfile-Datei.

```none
docker history iis

IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
f4caf476e909        16 seconds ago       cmd /S /C REM (nop) CMD ["cmd"]                 41.84 kB
f0e017e5b088        21 seconds ago       cmd /S /C echo "Hello World - Dockerfile" > c   6.816 MB
88438e174b7c        About a minute ago   cmd /S /C dism /online /enable-feature /all /   162.7 MB
6801d964fda5        4 months ago                                                         0 B
```

Jede dieser Ebenen kann einer Anweisung aus der Dockerfile-Datei zugeordnet werden. Die unterste Ebene (`6801d964fda5` in diesem Beispiel) stellt das Basis-Betriebssystemimage dar. Eine Ebene darüber ist die IIS-Installation sichtbar. Die nächste Ebene enthält die neue Website usw.

Dockerfile-Dateien können geschrieben werden, um Imageebenen zu minimieren und sowohl die Buildleistung als auch optische Aspekte wie die Lesbarkeit zu optimieren. Schließlich stehen viele Alternativen zur Durchführung einer bestimmten Buildaufgabe zur Verfügung. Fundiertes Wissen darüber, wie sich das Format einer Dockerfile-Datei auf die erforderliche Zeit für den Build und das resultierende Image auswirkt, verbessert die Automatisierungserfahrung. 

## Optimieren der Imagegröße

Wenn Sie Docker-Containerimages erstellen, kann die Imagegröße ein wichtiger Faktor sein. Containerimages werden zwischen Registrierungen und Host verschoben, exportiert und importiert und verbrauchen so letztendlich Speicherplatz. Verschiedene Taktiken können während des Docker Build-Prozesses verwendet werden, um die Imagegröße zu minimieren. In diesem Abschnitt werden einige dieser Taktiken speziell für Windows-Container ausführlich dargestellt. 

Weitere Informationen zu bewährten Vorgehensweisen mit Dockerfile finden Sie unter [Best practices for writing Dockerfiles on Docker.com]( https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/) (Bewährte Vorgehensweisen für das Schreiben von Dockerfile-Dateien auf Docker.com).

### Gruppieren von verwandten Aktionen

Da jede `RUN`-Anweisung eine neue Ebene in dem Containerimage erstellt, kann die Gruppierung von Aktionen in einer `RUN`-Anweisung die Anzahl der Ebenen reduzieren. Während das Minimieren der Ebenen sich möglicherweise nicht auf die Imagegröße auswirkt, ist dies mit dem Gruppieren von verwandten Aktionen möglich, wie die nachfolgenden Beispiele zeigen.

Die folgenden beiden Beispiele veranschaulichen den gleichen Vorgang, was zu Containerimages identischer Funktionalität führt, die beiden Dockerfile-Dateien erstellten sie jedoch unterschiedlich. Die resultierenden Images werden auch verglichen.  

Im nachfolgenden ersten Beispiel wird Python für Windows heruntergeladen, installiert und aufgeräumt (durch das Löschen der heruntergeladenen Setupdatei). Für jede dieser Aktionen wird eine eigene `RUN`-Anweisung ausgeführt.

```none
FROM windowsservercore

RUN powershell.exe -Command Invoke-WebRequest "https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe" -OutFile c:\python-3.5.1.exe
RUN powershell.exe -Command Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait
RUN powershell.exe -Command Remove-Item c:\python-3.5.1.exe -Force
```

Das resultierende Image besteht aus drei zusätzliche Ebenen, einer für jede `RUN`-Anweisung.

```none
docker history doc-example-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
a395ca26777f        15 seconds ago      cmd /S /C powershell.exe -Command Remove-Item   24.56 MB
6c137f466d28        28 seconds ago      cmd /S /C powershell.exe -Command Start-Proce   178.6 MB
957147160e8d        3 minutes ago       cmd /S /C powershell.exe -Command Invoke-WebR   125.7 MB
```

Zum Vergleich sehen Sie hier den gleichen Vorgang, wobei jedoch alle Schritte mit der gleichen `RUN`-Anweisung ausgeführt werden. Beachten Sie, dass jeder Schritt in der `RUN`-Anweisung sich in einer neuen Zeile der Dockerfile-Datei befindet, wobei der Zeilenumbruch mit dem Zeichen '\'erfolgt. 

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

Das resultierende Image besteht hier aus einer zusätzlichen Ebene für die `RUN`-Anweisung.

```none
docker history doc-example-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
69e44f37c748        54 seconds ago      cmd /S /C powershell.exe -Command   $ErrorAct   216.3 MB                
```

### Entfernen überflüssiger Dateien

Wenn eine Datei, z. B. ein Installationsprogramm, nach der Verwendung nicht mehr erforderlich ist, entfernen Sie die Datei, um die Imagegröße zu verringern. Dies muss in dem gleichen Schritt erfolgen, in dem die Datei in die Imageebene kopiert wurde. So wird verhindert, dass die Datei auf einer niedrigeren Imageebene beibehalten wird.

In diesem Beispiel wird das Python-Paket heruntergeladen, ausgeführt und dann die ausführbare Datei entfernt. Dies alles wird in einem einzigen `RUN`-Vorgang ausgeführt und resultiert in einer einzelnen Imageebene.

```none
FROM windowsservercore

RUN powershell.exe -Command \
  $ErrorActionPreference = 'Stop'; \
  Invoke-WebRequest https://www.python.org/ftp/python/3.5.1/python-3.5.1.exe -OutFile c:\python-3.5.1.exe ; \
  Start-Process c:\python-3.5.1.exe -ArgumentList '/quiet InstallAllUsers=1 PrependPath=1' -Wait ; \
  Remove-Item c:\python-3.5.1.exe -Force
```

## Optimieren der Buildgeschwindigkeit

### Mehrere Zeilen

Bei der Optimierung der Docker Build-Geschwindigkeit kann es vorteilhaft sein, Vorgänge in mehrere einzelne Anweisungen aufzuteilen. Mehrere `RUN`-Vorgänge steigern die Effizienz des Zwischenspeicherns. Für jede `RUN`-Anweisung wird eine individuelle Ebene erstellt, d. h. wenn ein identischer Schritt bereits in einem anderen Docker Build-Vorgang ausgeführt wurde, wird dieser zwischengespeicherte Vorgang (Imageebene) erneut verwendet. Das Ergebnis ist eine verringerte Docker Build-Laufzeit.

Im folgenden Beispiel werden sowohl Apache als auch die verteilbaren Pakete von Visual Studio heruntergeladen, installiert und dann die nicht benötigten Dateien bereinigt. Dies alles erfolgt mit einer einzigen `RUN`-Anweisung. Wenn eine dieser Aktionen aktualisiert wird, werden alle Aktionen erneut ausgeführt.

```none
FROM windowsservercore

RUN powershell -Command \
    
  # Download software ; \
    
  wget https://www.apachelounge.com/download/VC11/binaries/httpd-2.4.18-win32-VC11.zip -OutFile c:\apache.zip ; \
  wget "https://download.microsoft.com/download/1/6/B/16B06F60-3B20-4FF2-B699-5E9B7962F9AE/VSU_4/vcredist_x86.exe" -OutFile c:\vcredist.exe ; \
  wget -Uri http://windows.php.net/downloads/releases/php-5.5.33-Win32-VC11-x86.zip -OutFile c:\php.zip ; \
    
  # Install Software ; \
    
  Expand-Archive -Path c:\php.zip -DestinationPath c:\php ; \
  Expand-Archive -Path c:\apache.zip -DestinationPath c:\ ; \
  start-Process c:\vcredistexe -ArgumentList '/quiet' -Wait ; \
    
  # Remove unneeded files ; \
     
  Remove-Item c:\apache.zip -Force; \
  Remove-Item c:\vcredist.exe -Force
```

Das resultierende Image besteht aus zwei Ebenen, einer für das grundlegende Betriebssystemimage und der zweiten, die alle Vorgänge aus der einzelnen `RUN`-Anweisung enthält.

```none
docker history doc-sample-1

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
9bdf3a21fd41        8 minutes ago       cmd /S /C powershell -Command     Invoke-WebR   205.8 MB
6801d964fda5        5 months ago                                                        0 B
```

Zum Vergleich sehen Sie hier die gleichen Aktionen auf drei `RUN`-Anweisungen verteilt. In diesem Fall wird jede `RUN`-Anweisung in einer Containerimageebene zwischengespeichert, und nur solche, die geändert wurden, müssen in nachfolgenden Dockerfile-Builds neu ausgeführt werden.

```none
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

Das resultierende Image besteht aus vier Ebenen, einer für das Basisbetriebssystem-Image und dann einer für jede `RUN`-Anweisung. Da jede `RUN`-Anweisung in einer eigenen Ebene ausgeführt wurde, verwenden alle nachfolgenden Ausführungen dieser Dockerfile-Datei oder eines identischen Satzes von Anweisungen in einer anderen Dockerfile-Datei die zwischengespeicherte Imageebene, wodurch die Buildzeit reduziert wird. Die Anweisungsreihenfolge ist wichtig beim Arbeiten mit Imagecache. Weitere Details finden Sie im nächsten Abschnitt dieses Dokuments.

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ddf43b1f3751        6 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   127.2 MB
d43abb81204a        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   66.46 MB
7a21073861a1        7 days ago          cmd /S /C powershell -Command  Sleep 2 ;  Inv   115.8 MB
6801d964fda5        5 months ago
```

### Anordnung von Anweisungen

Eine Dockerfile-Datei wird von oben nach unten verarbeitet und jede Anweisung mit den zwischengespeicherten Ebenen verglichen. Wenn eine Anweisung gefunden wird, der keine zwischengespeicherte Ebene zugeordnet ist, werden diese Anweisung und alle nachfolgenden Anweisungen in neuen Containerimageebenen verarbeitet. Aus diesem Grund ist die Reihenfolge wichtig, in der die Anweisungen platziert werden. Platzieren Sie Anweisungen, die konstant bleiben werden, im oberen Teil der Dockerfile-Datei. Platzieren Sie Anweisungen, die sich ändern könnten, im unteren Teil der Dockerfile-Datei. Dies reduziert die Wahrscheinlichkeit, dass der vorhandene Cache negiert wird.

Dieses Beispiel zeigt, wie die Reihenfolge von Dockerfile-Anweisungen die Wirksamkeit des Zwischenspeichers beeinflussen kann. In dieser einfachen Dockerfile-Datei werden vier nummerierte Ordner erstellt.  

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-3
RUN mkdir test-4
```
Das resultierende Image besteht aus fünf Ebenen, einer für das Basisbetriebssystem-Image und einer für jede `RUN`-Anweisung.

```none
docker history doc-sample-1

IMAGE               CREATED              CREATED BY               SIZE                COMMENT
afba1a3def0a        38 seconds ago       cmd /S /C mkdir test-4   42.46 MB
86f1fe772d5c        49 seconds ago       cmd /S /C mkdir test-3   42.35 MB
68fda53ce682        About a minute ago   cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        About a minute ago   cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                  0 B    
```

Die Dockerfile-Datei wurde jetzt etwas geändert. Beachten Sie, dass die dritte `RUN`-Anweisung geändert wurde. Wenn Docker Build mit dieser Dockerfile-Datei ausgeführt wird, verwenden die ersten drei Anweisungen, die mit denen im letzten Beispiel identisch sind, zwischengespeicherte Imageebenen. Aber da die geänderte `RUN`-Anweisung nicht zwischengespeichert wurde, wird eine neue Ebene für sich selbst und alle nachfolgenden Anweisungen erstellt.

```none
FROM windowsservercore

RUN mkdir test-1
RUN mkdir test-2
RUN mkdir test-5
RUN mkdir test-4
```

Der Vergleich der Image-IDs des neuen Images mit denen im letzten Beispiel zeigt, dass die ersten drei Ebenen (von unten nach oben) gemeinsam verwendet werden, aber die vierte und fünfte individuell sind.

```none
docker history doc-sample-2

IMAGE               CREATED             CREATED BY               SIZE                COMMENT
c92cc95632fb        28 seconds ago      cmd /S /C mkdir test-4   5.644 MB
2f05e6f5c523        37 seconds ago      cmd /S /C mkdir test-5   5.01 MB
68fda53ce682        3 minutes ago       cmd /S /C mkdir test-2   6.745 MB
5e5aa8ba1bc2        4 minutes ago       cmd /S /C mkdir test-1   7.12 MB
6801d964fda5        5 months ago                                 0 B
```

## Optische Optimierung

### Groß-/Kleinschreibung der Anweisung

Bei Dockerfile-Anweisungen wird nicht zwischen Groß- und Kleinschreibung unterschieden, üblicherweise werden jedoch Großbuchstaben verwendet. Dies verbessert die Lesbarkeit durch die Unterscheidung zwischen Anweisungsaufruf und Anweisungsvorgang. Die folgenden beiden Beispiele veranschaulichen dieses Konzept. 

Kleinschreibung:
```none
# Sample Dockerfile

from windowsservercore
run dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
run echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
cmd [ "cmd" ]
```
Großschreibung: 
```none
# Sample Dockerfile

FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
CMD [ "cmd" ]
```

### Zeilenumbruch

Lange und komplexe Vorgänge können mit dem umgekehrten Schrägstrich (`\`) auf mehrere Zeilen verteilt werden. Die folgende Dockerfile-Datei installiert das verteilbare Paket von Visual Studio, entfernt die Dateien des Installationsprogramms und erstellt dann eine Konfigurationsdatei. Alle diese drei Vorgänge werden in einer Zeile angegeben.

```none
FROM windowsservercore

RUN powershell -Command c:\vcredist_x86.exe /quiet ; Remove-Item c:\vcredist_x86.exe -Force ; New-Item c:\config.ini
```
Der Befehl kann derart neu geschrieben werden, dass jeder Vorgang der einen `RUN`-Anweisung in einer eigenen Zeile angegeben ist. 

```none
FROM windowsservercore

RUN powershell -Command \
    $ErrorActionPreference = 'Stop'; \
    start-Process c:\vcredist_x86.exe -ArgumentList '/quiet' -Wait ; \
    Remove-Item c:\vcredist_x86.exe -Force ; \
    New-Item c:\config.ini
```

## Weitere Informationen und Referenzen

[Dockerfile on Windows] (Dockerfile unter Windows) (./manage_windows_dockerfile.md)

[Bewährte Methoden zum Schreiben von Dockerfile-Dateien auf Docker.com](https://docs.docker.com/engine/reference/builder/)



<!--HONumber=Aug16_HO5-->


