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
ms.sourcegitcommit: ea86e2dd88413569e4329ab27a8b6a4d3a7afca9
ms.openlocfilehash: d2fe856b9d00c5f7ac33d683f1c2204dc06d4a11

---

# Docker-Modul unter Windows

Das Docker-Modul und der Docker-Client sind nicht im Lieferumfang von Windows enthalten und müssen einzeln installiert und konfiguriert werden. Das Docker-Modul kann zudem zahlreiche verschiedene Konfigurationen akzeptieren. Beispielsweise kann konfiguriert werden, wie der Daemon eingehende Anforderungen akzeptiert, und Sie können standardmäßige Netzwerkoptionen sowie Einstellungen für Debugging und Protokolle konfigurieren. Unter Windows können diese Konfigurationen in einer Konfigurationsdatei oder mit dem Windows-Dienststeuerungs-Manager angegeben werden. In diesem Dokument wird beschrieben, wie Sie das Docker-Modul installieren und konfigurieren, und es werden einige Beispiele für gängige Konfigurationen vorgestellt.


## Installieren von Docker
Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul (dockerd.exe) und dem Docker-Client (docker.exe). Die einfachste Möglichkeit, alles zu installieren, ist die Verwendung der Schnellstartanleitungen. Sie helfen Ihnen bei der Einrichtung und bei der Ausführung Ihres ersten Containers. 

* [Windows-Container unter Windows Server 2016](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/quick_start/quick_start_windows_server)
* [Windows-Container unter Windows 10](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/quick_start/quick_start_windows_10)


### Manuelle Installation
Befolgen Sie die nachstehenden Schritte, wenn Sie stattdessen eine in der Entwicklung befindliche Version des Docker-Moduls und -Clients verwenden möchten. Hiermit werden das Docker-Modul und der Docker-Client installiert. Fahren Sie andernfalls mit dem nächsten Abschnitt fort.

> Wenn Sie Docker für Windows installiert haben, stellen Sie sicher, dass Sie es vor der Ausführung der folgenden manuellen Installationsschritte entfernen. 

Herunterladen des Docker-Moduls

Sie erhalten die neueste Version immer unter https://master.dockerproject.org. Dieses Beispiel verwendet die neueste Version aus dem Zweig v1.13-development. 

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Erweitern Sie das ZIP-Archiv in „Programme“.

```
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
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

## Konfigurieren von Docker mit einer Konfigurationsdatei

Die bevorzugte Methode zum Konfigurieren des Docker-Moduls ist die Verwendung einer Konfigurationsdatei. Die Konfigurationsdatei finden Sie unter „C:\ProgramData\docker\config\daemon.json“. Wenn diese Datei nicht bereits vorhanden ist, kann sie erstellt werden.

Hinweis: Nicht jede verfügbare Docker-Konfigurationsoption ist für Docker unter Windows anwendbar. Die nachfolgenden Beispiele zeigen für Windows gültige Optionen. Eine vollständige Dokumentation für die Konfiguration des Docker-Moduls finden Sie auf der Docker-Seite unter [Daemon Configuration File (Daemon-Konfigurationsdatei)](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

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

## Konfigurieren von Docker über den Docker-Dienst

Das Docker-Modul kann auch konfiguriert werden, indem der Docker-Dienst mit `sc config` geändert wird. Bei Verwendung dieser Methode werden die Flags des Docker-Moduls direkt im Docker-Dienst festgelegt. Führen Sie den folgenden Befehl in einem Eingabeaufforderungsfenster („cmd.exe“, nicht PowerShell) aus:


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

Hinweis: Sie müssen diesen Befehl nicht ausführen, wenn Ihre Datei „daemon.json“ den Eintrag `"hosts": ["tcp://0.0.0.0:2375"]` bereits enthält.

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
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

Starten Sie den Docker-Dienst neu, sobald die Variable festgelegt wurde.

```none
restart-service docker
```

Weitere Informationen finden Sie unter [Windows Configuration File (Windows-Konfigurationsdatei)](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file) auf Docker.com.




<!--HONumber=Oct16_HO3-->


