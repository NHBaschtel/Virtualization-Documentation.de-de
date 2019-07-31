---
title: Problembehandlung für Windows-Container
description: Problembehandlung, Tipps, automatisierte Skripts sowie protokollierte Informationen für Windows-Container und Docker
keywords: Docker, Container, Problembehandlung, Protokolle
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
ms.openlocfilehash: 652b1a8e0ab12ac67dd2754051e36c523e3de509
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882943"
---
# <a name="troubleshooting"></a>Problembehandlung

Haben Sie Probleme beim Einrichten Ihres Computers oder beim Ausführen eines Containers? Wir haben ein PowerShell-Skript entwickelt, das nach häufigen Problemen sucht. Bitte probieren Sie es erst aus, um zu sehen, was es findet, und geben Sie Ihre Ergebnisse frei.

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
Eine Liste aller Tests, die das Skript ausführt sowie allgemeine Lösungen, finden Sie in der [Readme-Datei](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md) des Skripts.

Wenn das nicht hilft, suchen Sie die Quelle des Problems, posten Sie die Ausgabe Ihres Skripts im [Containerforum](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers). Dies ist der beste Ort, um Hilfe von der Community zu erhalten, zu der auch Windows Insiders und Entwickler gehören.


## <a name="finding-logs"></a>Suchen von Protokollen
Es gibt mehrere Dienste, die zum Verwalten von Windows-Containern verwendet werden. Im nächsten Abschnitt wird gezeigt, wie Sie Protokolle für jeden Dienst erhalten.

# <a name="docker-engine"></a>Docker-Modul
Das Docker-Modul protokolliert in das Windows-„Anwendungsereignisprotokoll“, statt in eine Datei. Diese Protokolle können mithilfe von Windows PowerShell einfach gelesen, sortiert und gefiltert werden.

Beispielsweise werden dadurch die Protokolle des Docker-Moduls der letzten fünf Minuten angezeigt, angefangen mit dem ältesten.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Dies könnte auch einfach in eine CSV-Datei weitergeleitet werden, um dort von einem anderen Tool oder Arbeitsblatt gelesen zu werden.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

## <a name="enabling-debug-logging"></a>Aktivieren der Debugprotokollierung
Sie können auch die Protokollierung auf Debugebene im Docker-Modul aktivieren. Dies kann für die Problembehandlung hilfreich sein, wenn reguläre Protokolle nicht genügend Informationen enthalten.

Öffnen Sie zunächst eine Eingabeaufforderung mit erhöhten Rechten, führen Sie dann `sc.exe qc docker` aus, um die aktuelle Befehlszeile für den Docker-Dienst zu erhalten.
Beispiel:
```
C:\> sc.exe qc docker
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: docker
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : "C:\Program Files\Docker\dockerd.exe" --run-service
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Docker Engine
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

Modifizieren Sie den aktuellen `BINARY_PATH_NAME`:
- Fügen Sie -D am Ende hinzu
- Fügen Sie für " das Escapezeichen \ hinzu
- Schließen Sie den gesamten Befehl in Anführungszeichen (") ein

Führen Sie anschließend `sc.exe config docker binpath=`, gefolgt von der neuen Zeichenfolge, aus. Beispiel: 
```
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


Starten Sie nun den Docker-Dienst neu.
```
sc.exe stop docker
sc.exe start docker
```

Dadurch wird viel mehr im Anwendungsereignisprotokoll protokolliert. Deshalb ist es von Vorteil, die Option `-D` zu entfernen, wenn Sie mit der Problembehandlung fertig sind. Führen Sie die gleichen Schritte wie oben aus, jedoch ohne `-D`, und starten Sie den Dienst neu, um die Debugprotokollierung zu deaktivieren.

Eine andere Möglichkeit ist, den Docker Daemon im Debugmodus mithilfe einer PowerShell-Eingabeaufforderung mit erhöhten Rechten auszuführen, die die Ausgabe direkt in einer Datei aufzeichnen.
```PowerShell
sc.exe stop docker
<path\to\>dockerd.exe -D > daemon.log 2>&1
```

## <a name="obtaining-stack-dump"></a>Abrufen des Stapel Dumps

Im Allgemeinen ist dies nur nützlich, wenn Sie von Microsoft-Support oder andocker-Entwicklern ausdrücklich angefordert werden. Es kann verwendet werden, um die Diagnose einer Situation zu unterstützen, in der Andockfenster anscheinend aufgehängt wurde. 

Herunterladen von [Docker-Signal.exe](https://github.com/jhowardmsft/docker-signal).

Syntax:
```PowerShell
Get-Process dockerd
# Note the process ID in the `Id` column
docker-signal -pid=<id>
```

Die Ausgabedatei befindet sich im Daten-Root-Verzeichnis, in dem docker ausgeführt wird. Das Standardverzeichnis ist `C:\ProgramData\Docker`. Das aktuelle Verzeichnis kann durch Ausführen von `docker info -f "{{.DockerRootDir}}"` bestätigt werden.

Die Datei ist `goroutine-stacks-<timestamp>.log`.

Beachten Sie `goroutine-stacks*.log` , dass keine persönlichen Informationen enthalten sind.


# <a name="host-compute-service"></a>Hostcomputedienst
Das Docker-Modul ist von einem Windows-spezifischen Hostcomputedienst abhängig. Dieser verfügt über separate Protokolle: 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

Sie werden in der Ereignisansicht angezeigt und können auch mit PowerShell abgefragt werden.

Beispiel:
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```

## <a name="capturing-hcs-analyticdebug-logs"></a>Erfassen von HCS Analyse-/Debug-Protokollen

Um Analyse-/Debug-Protokolle für Hyper-V-Compute zu aktivieren und auf `hcslog.evtx` zu speichern.

```PowerShell
# Enable the analytic logs
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:true /q:true

# <reproduce your issue>

# Export to an evtx
wevtutil.exe epl Microsoft-Windows-Hyper-V-Compute-Analytic <hcslog.evtx>

# Disable
wevtutil.exe sl Microsoft-Windows-Hyper-V-Compute-Analytic /e:false /q:true
```

## <a name="capturing-hcs-verbose-tracing"></a>Aufzeichnen ausführlicher HCS-Protokollierung

In der Regel ist sie nur dann nützlich, wenn sie vom Microsoft Support angefordert wird. 

[HcsTraceProfile.wprp](https://gist.github.com/jhowardmsft/71b37956df0b4248087c3849b97d8a71) herunterladen

```PowerShell
# Enable tracing
wpr.exe -start HcsTraceProfile.wprp!HcsArgon -filemode

# <reproduce your issue>

# Capture to HcsTrace.etl
wpr.exe -stop HcsTrace.etl "some description"
```

Bereitstellen von `HcsTrace.etl` an die Supportkontaktperson.
