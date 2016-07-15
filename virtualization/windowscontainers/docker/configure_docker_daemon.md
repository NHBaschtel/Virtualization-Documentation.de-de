---
title: Konfigurieren von Docker unter Windows
description: Konfigurieren von Docker unter Windows
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
translationtype: Human Translation
ms.sourcegitcommit: 2d6f2c24624883457302c925c2bb47e6c867b730
ms.openlocfilehash: 533f3a3277e3d9654f0d425c9c0f442c93e2d24a

---

# Docker-Daemon unter Windows

Das Docker-Modul ist nicht im Lieferumfang von Windows enthalten und muss einzeln installiert und konfiguriert werden. Docker-Daemon kann zahlreiche verschiedene Konfigurationen akzeptieren. Beispielsweise kann konfiguriert werden, wie der Daemon eingehende Anforderungen akzeptiert, und Sie können standardmäßige Netzwerkoptionen sowie Einstellungen für Debugging und Protokolle konfigurieren. Unter Windows können diese Konfigurationen in einer Konfigurationsdatei oder mit dem Windows-Dienststeuerungs-Manager angegeben werden. In diesem Dokument wird beschrieben, wie Sie den Docker-Daemon installieren und konfigurieren, und es werden einige Beispiele für gängige Konfigurationen vorgestellt.

## Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Für diese Übung werden beide installiert.

Erstellen Sie einen Ordner für die ausführbaren Docker-Dateien.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Laden Sie den Docker-Daemon herunter.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Laden Sie den Docker-Client herunter.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
```

Fügen Sie das Docker-Verzeichnis dem Systempfad hinzu. Starten Sie anschließend die PowerShell-Sitzung neu, damit der geänderte Pfad erkannt wird.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Führen Sie den folgenden Befehl aus, um Docker als Windows-Dienst zu installieren.

```none
dockerd --register-service
```

Nach Abschluss der Installation kann der Dienst gestartet werden.

```none
Start-Service Docker
```

Bevor Docker verwendet werden kann, müssen Containerimages installiert werden. Weitere Informationen finden Sie unter [Verwalten von Containerimages](../management/manage_images.md).

## Docker-Konfigurationsdatei

Die bevorzugte Methode zum Konfigurieren des Docker-Daemons ist die Verwendung einer Konfigurationsdatei. Die Konfigurationsdatei finden Sie unter „C:\ProgramData\docker\config\daemon.json“. Wenn diese Datei nicht bereits vorhanden ist, kann sie erstellt werden.

Hinweis: Nicht jede verfügbare Docker-Konfigurationsoption ist für Docker unter Windows anwendbar. Die nachfolgenden Beispiele zeigen für Windows gültige Optionen. Eine vollständige Dokumentation für die Konfiguration des Docker-Daemons (einschließlich Linux) finden Sie unter [Docker-Daemon]( https://docs.docker.com/v1.10/engine/reference/commandline/daemon/).

```none
{
    "authorization-plugins": [],
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "", 
    "mtu": 0,
    "pidfile": "",
    "graph": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "insecure-registries": [],
    "disable-legacy-registry": false
}
```

Der Konfigurationsdatei müssen nur die gewünschten Konfigurationsänderungen hinzugefügt werden. In diesem Beispiel wird der Docker-Daemon so konfiguriert, dass eingehende Verbindungen über Port 2375 akzeptiert werden. Für alle weiteren Konfigurationsoptionen werden die Standardwerte verwendet.

```none
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Ähnlich wird in diesem Beispiel der Docker-Daemon so konfiguriert, dass nur sichere Verbindungen über Port 2376 akzeptiert werden.

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```



## Dienststeuerungs-Manager

Der Docker-Daemon kann auch konfiguriert werden, indem der Docker-Dienst mit `sc config` geändert wird. Bei Verwendung dieser Methode werden die Docker-Daemon-Flags direkt im Docker-Dienst festgelegt.


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

## Allgemeine Konfiguration

Die folgenden Beispiele für Konfigurationsdateien zeigen allgemeine Docker-Konfigurationen. Diese können in einer einzigen Konfigurationsdatei kombiniert werden.

### Erstellen eines Standardnetzwerks 

Verwenden Sie den folgenden Befehl, um den Docker-Daemon so zu konfigurieren, dass kein NAT-Standardnetzwerk erstellt wird. Weitere Informationen finden Sie unter [Verwalten von Docker-Netzwerken](../management/container_networking.md).

```none
{
    "bridge" : "none"
}
```

### Festlegen der Docker-Sicherheitsgruppe

Wenn Sie sich beim Docker-Host angemeldet haben und Docker-Befehle lokal ausführen, werden diese Befehle über eine Named Pipe ausgeführt. Standardmäßig können nur Mitglieder der Administratorengruppe über die Named Pipe auf den Docker-Daemon zugreifen. Verwenden Sie das Flag `group`, um eine Sicherheitsgruppe mit diesem Zugriff festzulegen.

```none
{
    "group" : "docker"
}
```


## Sammeln von Protokollen
Der Docker-Daemon protokolliert in das Windows-„Anwendungsereignisprotokoll“, statt in eine Datei. Diese Protokolle können mithilfe von Windows PowerShell einfach gelesen, sortiert und gefiltert werden.

Beispielsweise zeigt dies die Docker-Daemon-Protokolle der letzten fünf Minuten an, angefangen mit dem ältesten.
```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Dies könnte auch einfach in eine CSV-Datei weitergeleitet werden, um dort von einem anderen Tool oder Arbeitsblatt gelesen zu werden.
```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.csv ```



<!--HONumber=Jul16_HO1-->


