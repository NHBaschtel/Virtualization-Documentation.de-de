---
title: "Problembehandlung für Windows-Container"
description: "Problembehandlung, Tipps, automatisierte Skripts sowie protokollierte Informationen für Windows-Container und Docker"
keywords: Docker, Container, Problembehandlung, Protokolle
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ebd79cd3-5fdd-458d-8dc8-fc96408958b5
translationtype: Human Translation
ms.sourcegitcommit: c65353d0b6dff233819dcc8f4f92eb186bf3b8fc
ms.openlocfilehash: 9f28c35c6eaddd8bcf3883863b63251378f845a7

---

# Problembehandlung

Haben Sie Probleme beim Einrichten Ihres Computers oder beim Ausführen eines Containers? Wir haben ein PowerShell-Skript entwickelt, das nach häufigen Problemen sucht. Bitte probieren Sie es erst aus, um zu sehen, was es findet, und geben Sie Ihre Ergebnisse frei.

```PowerShell
Invoke-WebRequest https://aka.ms/Debug-ContainerHost.ps1 -UseBasicParsing | Invoke-Expression
```
Eine Liste aller Tests, die das Skript ausführt sowie allgemeine Lösungen, finden Sie in der [Readme-Datei](https://github.com/Microsoft/Virtualization-Documentation/blob/live/windows-server-container-tools/Debug-ContainerHost/README.md) des Skripts.

Wenn das nicht hilft, suchen Sie die Quelle des Problems, posten Sie die Ausgabe Ihres Skripts im [Containerforum](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers). Dies ist der beste Ort, um Hilfe von der Community zu erhalten, zu der auch Windows Insiders und Entwickler gehören.


## Suchen von Protokollen
Es gibt mehrere Dienste, die zum Verwalten von Windows-Containern verwendet werden. Im nächsten Abschnitt wird gezeigt, wie Sie Protokolle für jeden Dienst erhalten.

### Docker-Modul
Das Docker-Modul protokolliert in das Windows-„Anwendungsereignisprotokoll“, statt in eine Datei. Diese Protokolle können mithilfe von Windows PowerShell einfach gelesen, sortiert und gefiltert werden.

Beispielsweise werden dadurch die Protokolle des Docker-Moduls der letzten fünf Minuten angezeigt, angefangen mit dem ältesten.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Dies könnte auch einfach in eine CSV-Datei weitergeleitet werden, um dort von einem anderen Tool oder Arbeitsblatt gelesen zu werden.

```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.CSV
```

#### Aktivieren der Debugprotokollierung
Sie können auch die Protokollierung auf Debugebene im Docker-Modul aktivieren. Dies kann für die Problembehandlung hilfreich sein, wenn reguläre Protokolle nicht genügend Informationen enthalten.

Öffnen Sie zunächst eine Eingabeaufforderung mit erhöhten Rechten, führen Sie dann `sc.exe qc docker` aus, um die aktuelle Befehlszeile für den Docker-Dienst zu erhalten.
Beispiel:
```none
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

Führen Sie anschließend `sc.exe config docker binpath= `, gefolgt von der neuen Zeichenfolge, aus. Beispiel: 
```none
sc.exe config docker binpath= "\"C:\Program Files\Docker\dockerd.exe\" --run-service -D"
```


Starten Sie nun den Docker-Dienst neu.
```none
sc.exe stop docker
sc.exe start docker
```

Dadurch wird viel mehr im Anwendungsereignisprotokoll protokolliert. Deshalb ist es von Vorteil, die Option `-D` zu entfernen, wenn Sie mit der Problembehandlung fertig sind. Führen Sie die gleichen Schritte wie oben aus, jedoch ohne `-D`, und starten Sie den Dienst neu, um die Debugprotokollierung zu deaktivieren.


### Hostcontainerdienst
Das Docker-Modul ist von einem bestimmten Windows-spezifischen Hostcontainerdienst abhängig. Dieser verfügt über separate Protokolle: 
- Microsoft-Windows-Hyper-V-Compute-Admin
- Microsoft-Windows-Hyper-V-Compute-Operational

Sie werden in der Ereignisansicht angezeigt und können auch mit PowerShell abgefragt werden.

Beispiel:
```PowerShell
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Admin
Get-WinEvent -LogName Microsoft-Windows-Hyper-V-Compute-Operational 
```




<!--HONumber=Jan17_HO4-->


