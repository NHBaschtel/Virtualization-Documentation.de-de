---
title: Konfigurieren von Docker unter Windows
description: Konfigurieren von Docker unter Windows
keywords: Docker, Container
author: PatrickLang
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: ccc45d47fc9f17c10b149bc647463824e1ecbc9e
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/08/2017
---
# <a name="docker-engine-on-windows"></a>Docker-Modul unter Windows

Das Docker-Modul und der Docker-Client sind nicht im Lieferumfang von Windows enthalten und müssen einzeln installiert und konfiguriert werden. Das Docker-Modul kann zudem zahlreiche verschiedene Konfigurationen akzeptieren. Beispielsweise kann konfiguriert werden, wie der Daemon eingehende Anforderungen akzeptiert, und Sie können standardmäßige Netzwerkoptionen sowie Einstellungen für Debugging und Protokolle konfigurieren. Unter Windows können diese Konfigurationen in einer Konfigurationsdatei oder mit dem Windows-Dienststeuerungs-Manager angegeben werden. In diesem Dokument wird beschrieben, wie Sie das Docker-Modul installieren und konfigurieren, und es werden einige Beispiele für gängige Konfigurationen vorgestellt.


## <a name="install-docker"></a>Installieren von Docker
Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul (dockerd.exe) und dem Docker-Client (docker.exe). Die einfachste Möglichkeit, alles zu installieren, ist die Verwendung der Schnellstartanleitungen. Sie helfen Ihnen bei der Einrichtung und bei der Ausführung Ihres ersten Containers. 

* [Windows-Container unter Windows Server 2016](../quick-start/quick-start-windows-server.md)
* [Windows-Container unter Windows10](../quick-start/quick-start-windows-10.md)


### <a name="manual-installation"></a>Manuelle Installation
Befolgen Sie die nachstehenden Schritte, wenn Sie stattdessen eine in der Entwicklung befindliche Version des Docker-Moduls und -Clients verwenden möchten. Damit werden das Docker-Modul und der Docker-Client installiert. Wenn Sie als Entwickler neue Features testen oder einen Windows-Insider-Build verwenden, müssen Sie möglicherweise eine in der Entwicklung befindliche Version von Docker verwenden. Führen Sie andernfalls die Schritte im obigen Abschnitt „Installieren von Docker“ aus, um die neuesten Versionen zu erhalten.

> Falls Sie Docker für Windows installiert haben, entfernen Sie es vor der Ausführung der folgenden manuellen Installationsschritte. 

Herunterladen des Docker-Moduls

Sie erhalten die neueste Version immer unter https://master.dockerproject.org. In diesem Beispiel wird die aktuelle Version aus dem Master Branch verwendet. 

```powershell
$version = (Invoke-WebRequest -UseBasicParsing https://raw.githubusercontent.com/docker/docker/master/VERSION).Content.Trim()
Invoke-WebRequest "https://master.dockerproject.org/windows/x86_64/docker-$($version).zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Extrahieren Sie das ZIP-Archiv in „Programme“.

```powershell
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Fügen Sie das Docker-Verzeichnis dem Systempfad hinzu. Starten Sie anschließend die PowerShell-Sitzung neu, damit der geänderte Pfad erkannt wird.

```powershell
# Add path to this PowerShell session immediately
$env:path += ";$env:ProgramFiles\Docker"

# For persistent use after a reboot
$existingMachinePath = [Environment]::GetEnvironmentVariable("Path",[System.EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("Path", $existingMachinePath + ";$env:ProgramFiles\Docker", [EnvironmentVariableTarget]::Machine)
```

Führen Sie den folgenden Befehl aus, um Docker als Windows-Dienst zu installieren.

```
dockerd --register-service
```

Nach Abschluss der Installation kann der Dienst gestartet werden.

```powershell
Start-Service Docker
```

Bevor Docker verwendet werden kann, müssen Containerimages installiert werden. Weitere Informationen finden Sie unter [Schnellstarthandbuch für die Verwendung von Images](../quick-start/quick-start-images.md).

## <a name="configure-docker-with-configuration-file"></a>Konfigurieren von Docker mit einer Konfigurationsdatei

Die bevorzugte Methode zum Konfigurieren des Docker-Moduls in Windows ist die Verwendung einer Konfigurationsdatei. Die Konfigurationsdatei finden Sie unter „C:\ProgramData\Docker\config\daemon.json”. Wenn diese Datei nicht bereits vorhanden ist, kann sie erstellt werden.

Hinweis: Nicht jede verfügbare Docker-Konfigurationsoption ist für Docker unter Windows anwendbar. Die nachfolgenden Beispiele zeigen für Windows gültige Optionen. Eine vollständige Dokumentation für die Konfiguration des Docker-Moduls finden Sie auf der Docker-Seite unter [Daemon Configuration File (Daemon-Konfigurationsdatei)](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

```
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

Der Konfigurationsdatei müssen nur die gewünschten Konfigurationsänderungen hinzugefügt werden. In diesem Beispiel wird das Docker-Modul so konfiguriert, dass eingehende Verbindungen über Port2375 akzeptiert werden. Für alle weiteren Konfigurationsoptionen werden die Standardwerte verwendet.

```
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Entsprechend konfiguriert dieses Beispiel den Docker-Daemon, um Images und Container in einem alternativen Pfad zu speichern. Wenn nicht angegeben, ist die Standardeinstellung c:\programdata\docker.

```
{    
    "graph": "d:\\docker"
}
```

Ähnlich wird in diesem Beispiel der Docker-Daemon so konfiguriert, dass nur sichere Verbindungen über Port 2376 akzeptiert werden.

```
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Konfigurieren von Docker über den Docker-Dienst

Das Docker-Modul kann auch konfiguriert werden, indem der Docker-Dienst mit `sc config` geändert wird. Bei Verwendung dieser Methode werden die Flags des Docker-Moduls direkt im Docker-Dienst festgelegt. Führen Sie den folgenden Befehl in einem Eingabeaufforderungsfenster („cmd.exe“, nicht PowerShell) aus:


```
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

Hinweis: Sie müssen diesen Befehl nicht ausführen, wenn Ihre Datei „daemon.json“ den Eintrag `"hosts": ["tcp://0.0.0.0:2375"]` bereits enthält.

## <a name="common-configuration"></a>Allgemeine Konfiguration

Die folgenden Beispiele für Konfigurationsdateien zeigen allgemeine Docker-Konfigurationen. Diese können in einer einzigen Konfigurationsdatei kombiniert werden.

### <a name="default-network-creation"></a>Erstellen eines Standardnetzwerks 

Verwenden Sie den folgenden Befehl, um das Docker-Modul so zu konfigurieren, dass kein NAT-Standardnetzwerk erstellt wird. Weitere Informationen finden Sie unter [Verwalten von Docker-Netzwerken](../manage-containers/container-networking.md).

```
{
    "bridge" : "none"
}
```

### <a name="set-docker-security-group"></a>Festlegen der Docker-Sicherheitsgruppe

Wenn Sie sich beim Docker-Host angemeldet haben und Docker-Befehle lokal ausführen, werden diese Befehle über eine Named Pipe ausgeführt. Standardmäßig können nur Mitglieder der Administratorengruppe über die Named Pipe auf das Docker-Modul zugreifen. Verwenden Sie das Flag `group`, um eine Sicherheitsgruppe mit diesem Zugriff festzulegen.

```
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>Proxykonfiguration

Erstellen Sie eine Windows-Umgebungsvariable namens `HTTP_PROXY` oder `HTTPS_PROXY` und den Wert der Proxyinformationen, um Proxyinformationen für `docker search` und `docker pull` festzulegen. Sie können dies in PowerShell tun, indem Sie einen Befehl ähnlich dem folgenden verwenden:

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

Starten Sie den Docker-Dienst neu, sobald die Variable festgelegt wurde.

```powershell
Restart-Service docker
```

Weitere Informationen finden Sie unter [Windows Configuration File (Windows-Konfigurationsdatei)](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file) auf Docker.com.

