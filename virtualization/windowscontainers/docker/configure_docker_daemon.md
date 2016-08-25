---
title: Konfigurieren von Docker unter Windows
description: Konfigurieren von Docker unter Windows
keywords: Docker, Container
author: neilpeterson
manager: timlt
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
translationtype: Human Translation
ms.sourcegitcommit: 4dded90462c5438a6836ec32a165a9cc1019d6ec
ms.openlocfilehash: 6ae49d82a89b2f30198de05aa4915726853172f5

---

# Docker-Modul unter Windows

Das Docker-Modul und der Docker-Client sind nicht im Lieferumfang von Windows enthalten und müssen einzeln installiert und konfiguriert werden. Das Docker-Modul kann zudem zahlreiche verschiedene Konfigurationen akzeptieren. Beispielsweise kann konfiguriert werden, wie der Daemon eingehende Anforderungen akzeptiert, und Sie können standardmäßige Netzwerkoptionen sowie Einstellungen für Debugging und Protokolle konfigurieren. Unter Windows können diese Konfigurationen in einer Konfigurationsdatei oder mit dem Windows-Dienststeuerungs-Manager angegeben werden. In diesem Dokument wird beschrieben, wie Sie das Docker-Modul installieren und konfigurieren, und es werden einige Beispiele für gängige Konfigurationen vorgestellt.

## Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Für diese Übung werden beide installiert.

Laden Sie das Docker-Modul herunter.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Erweitern Sie das ZIP-Archiv in „Programme“.

```
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
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

Die bevorzugte Methode zum Konfigurieren des Docker-Moduls ist die Verwendung einer Konfigurationsdatei. Die Konfigurationsdatei finden Sie unter „C:\ProgramData\docker\config\daemon.json“. Wenn diese Datei nicht bereits vorhanden ist, kann sie erstellt werden.

Hinweis: Nicht jede verfügbare Docker-Konfigurationsoption ist für Docker unter Windows anwendbar. Die nachfolgenden Beispiele zeigen für Windows gültige Optionen. Eine vollständige Dokumentation für die Konfiguration des Docker-Moduls (einschließlich Linux) finden Sie unter [Docker-Daemon]( https://docs.docker.com/v1.10/engine/reference/commandline/daemon/).

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

Der Konfigurationsdatei müssen nur die gewünschten Konfigurationsänderungen hinzugefügt werden. In diesem Beispiel wird das Docker-Modul so konfiguriert, dass eingehende Verbindungen über Port 2375 akzeptiert werden. Für alle weiteren Konfigurationsoptionen werden die Standardwerte verwendet.

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

Das Docker-Modul kann auch konfiguriert werden, indem der Docker-Dienst mit `sc config` geändert wird. Bei Verwendung dieser Methode werden die Flags des Docker-Moduls direkt im Docker-Dienst festgelegt.


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

## Allgemeine Konfiguration

Die folgenden Beispiele für Konfigurationsdateien zeigen allgemeine Docker-Konfigurationen. Diese können in einer einzigen Konfigurationsdatei kombiniert werden.

### Erstellen eines Standardnetzwerks 

Verwenden Sie den folgenden Befehl, um das Docker-Modul so zu konfigurieren, dass kein NAT-Standardnetzwerk erstellt wird. Weitere Informationen finden Sie unter [Verwalten von Docker-Netzwerken](../management/container_networking.md).

```none
{
    "bridge" : "none"
}
```

### Festlegen der Docker-Sicherheitsgruppe

Wenn Sie sich beim Docker-Host angemeldet haben und Docker-Befehle lokal ausführen, werden diese Befehle über eine Named Pipe ausgeführt. Standardmäßig können nur Mitglieder der Administratorengruppe über die Named Pipe auf das Docker-Modul zugreifen. Verwenden Sie das Flag `group`, um eine Sicherheitsgruppe mit diesem Zugriff festzulegen.

```none
{
    "group" : "docker"
}
```

## Proxykonfiguration

Erstellen Sie eine Windows-Umgebungsvariable namens `HTTP_PROXY` oder `HTTPS_PROXY` und den Wert der Proxyinformationen, um Proxyinformationen für `docker search` und `docker pull` festzulegen. Sie können dies in PowerShell tun, indem Sie einen Befehl ähnlich dem folgenden verwenden:

```none
[Environment]::SetEnvironmentVariable("HTTP_PROXY”, “http://username:password@proxy:port/”, [EnvironmentVariableTarget]::Machine)
```

Starten Sie den Docker-Dienst neu, sobald die Variable festgelegt wurde.

```none
restart-service docker
```

Weitere Informationen finden Sie unter [Daemon Socket Options](https://docs.docker.com/v1.10/engine/reference/commandline/daemon/#daemon-socket-option) (Daemon-Socketoptionen) auf Docker.com.

## Sammeln von Protokollen
Das Docker-Modul protokolliert in das Windows-„Anwendungsereignisprotokoll“, statt in eine Datei. Diese Protokolle können mithilfe von Windows PowerShell einfach gelesen, sortiert und gefiltert werden.

Beispielsweise werden dadurch die Protokolle des Docker-Moduls der letzten fünf Minuten angezeigt, angefangen mit dem ältesten.
```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-5) | Sort-Object Time 
```

Dies könnte auch einfach in eine CSV-Datei weitergeleitet werden, um dort von einem anderen Tool oder Arbeitsblatt gelesen zu werden.
```
Get-EventLog -LogName Application -Source Docker -After (Get-Date).AddMinutes(-30)  | Sort-Object Time | Export-CSV ~/last30minutes.csv ```



<!--HONumber=Aug16_HO4-->


