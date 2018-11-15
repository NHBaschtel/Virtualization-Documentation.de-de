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
ms.openlocfilehash: 3b592620f4667450c2454f8760b7f3c844c7e2ab
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948049"
---
# <a name="docker-engine-on-windows"></a>Docker-Modul unter Windows

Docker-Moduls und -Client sind nicht in Windows enthalten, und müssen installiert und einzeln konfiguriert werden. Das Docker-Modul kann zudem zahlreiche verschiedene Konfigurationen akzeptieren. Beispielsweise kann konfiguriert werden, wie der Daemon eingehende Anforderungen akzeptiert, und Sie können standardmäßige Netzwerkoptionen sowie Einstellungen für Debugging und Protokolle konfigurieren. Unter Windows können diese Konfigurationen in einer Konfigurationsdatei oder mit dem Windows-Dienststeuerungs-Manager angegeben werden. Dieses Dokument enthält Informationen zum Installieren und konfigurieren das Docker-Modul und bietet auch einige Beispiele für häufig verwendete Konfigurationen.


## <a name="install-docker"></a>Installieren von Docker
Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul (dockerd.exe) und dem Docker-Client (docker.exe). Die einfachste Möglichkeit, alles zu installieren, ist die Verwendung der Schnellstartanleitungen. Sie Hilfe erhalten Sie alles eingerichtet und Ausführung Ihres ersten Containers. 

* [Windows-Container unter Windows Server 2016](../quick-start/quick-start-windows-server.md)
* [Windows-Container unter Windows10](../quick-start/quick-start-windows-10.md)

Informationen über die skriptbasierte Installationen finden Sie unter [Use a script to install Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Bevor Docker verwendet werden kann müssen containerimages installiert werden. Weitere Informationen finden Sie unter [Schnellstarthandbuch für die Verwendung von Images](../quick-start/quick-start-images.md).

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
    "data-root": "",
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
    "data-root": "d:\\docker"
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

Verwenden Sie den folgenden Befehl, um das Docker-Modul so zu konfigurieren, dass kein NAT-Standardnetzwerk erstellt wird. Weitere Informationen finden Sie unter [Verwalten von Docker-Netzwerken](../container-networking/network-drivers-topologies.md).

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

## <a name="uninstall-docker"></a>Deinstallieren von Docker
*Verwenden Sie die Schritte in diesem Abschnitt, um Docker zu deinstallieren und eine vollständige Bereinigung der Docker-Systemkomponenten aus dem Windows10- oder Windows Server2016-System durchzuführen.*

> Hinweis: Alle Befehle in den folgenden Schritten müssen aus einer PowerShell-Sitzung **mit erhöhten Rechten** ausgeführt werden.

### <a name="step-1-prepare-your-system-for-dockers-removal"></a>SCHRITT1: Vorbereiten des Systems für das Entfernen von Docker 
Es empfiehlt sich, sicherzustellen, dass keine Container auf Ihrem System ausgeführt werden, bevor Docker entfernt wird, sofern nicht bereits geschehen. Hier sind einige hilfreiche Befehle für den Vorgang:
```
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```
Es empfiehlt sich ebenfalls, alle Container, Containerimages, Netzwerke und Volumes aus dem System zu entfernen, bevor Docker entfernt wird:
```
docker system prune --volumes --all
```

### <a name="step-2-uninstall-docker"></a>SCHRITT 2: Deinstallieren von Docker 

#### ***<a name="steps-to-uninstall-docker-on-windows-10"></a>Schritte zum Deinstallieren von Docker unter Windows10:10:***
- Wechseln Sie zu **"Einstellungen" > "Apps"** auf Ihrem Windows10-Computer
- Suchen Sie unter **"Apps & Features"** nach **"Docker für Windows"**
- Klicken Sie auf **"Docker für Windows" > "Deinstallieren"**

#### ***<a name="steps-to-uninstall-docker-on-windows-server-2016"></a>Schritte zum Deinstallieren von Docker unter WindowsServer 2016:16:***
Verwenden Sie in einer PowerShell-Sitzung mit erhöhten Rechten die Cmdlets `Uninstall-Package` und `Uninstall-Module`, um das Docker-Modul den dazu gehörigen Anbieter für die Paketverwaltung von Ihrem System zu entfernen. 
> Tipp: So finden Sie den Anbieter des Pakets, den Sie zum Installieren von Docker verwendet haben `PS C:\> Get-PackageProvider -Name *Docker*`

*Beispiel*:
```
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

### <a name="step-3-cleanup-docker-data-and-system-components"></a>SCHRITT3: Bereinigen von Dockerdaten und Systemkomponenten
Entfernen Sie die *Standard-Netzwerke* von Docker, damit deren Konfiguration nicht auf Ihrem System verbleibt, nachdem Docker entfernt wurde:
```
Get-HNSNetwork | Remove-HNSNetwork
```
Entfernen Sie die *Programmdaten* von Docker aus dem System:
```
Remove-Item "C:\ProgramData\Docker" -Recurse
```
Sie können ebenfalls die *optionalen Features von Windows* entfernen, die Docker/Containern unter Windows zugeordnet sind. 

Dies beinhaltet mindestens das Feature "Container", das automatisch auf allen Windows10 oder Windows Server2016 aktiviert wird, wenn Docker installiert ist. Es kann ebenfalls das Feature "Hyper-V" beinhalten, das unter Windows10 automatisch aktiviert wird, wenn Docker installiert ist. Dies muss jedoch explizit auf Windows Server2016 aktiviert werden.

> **WICHTIGER HINWEIS ZUM DEAKTIVIEREN VON HYPER-V:** [Das Hyper-V-Feature](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/) ist eine allgemeine Virtualisierungsfeature, das mehr als nur Container aktiviert! Stellen Sie vor dem Deaktivieren des Hyper-V-Features sicher, dass keine anderen virtualisierten Komponenten auf dem System vorhanden sind, die dies erfordern.

#### ***<a name="steps-to-remove-windows-features-on-windows-10"></a>Schritte zum Entfernen von Windows-Features unter Windows10:10:***
- Wechseln sie auf Ihrem Windows 10-Computer zu **"Systemsteuerung" > "Programme" > "Programme und Features" > "Windows-Features aktivieren oder deaktivieren"**
- Suchen Sie den Namen der Funktion/en, die Sie deaktivieren möchten – in diesem Fall **"Container"** und (optional) **"Hyper-V"**
- **Deaktivieren Sie** das Kontrollkästchen neben dem Namen des Features, das Sie deaktivieren möchten
- Klicken Sie auf **"OK"**.

#### ***<a name="steps-to-remove-windows-features-on-windows-server-2016"></a>Schritte zum Entfernen von Windows-Features unter Windows Server2016:16:***
Verwenden Sie die folgenden Befehle aus einer PowerShell-Sitzung mit erhöhten Rechten zum Deaktivieren der **"Container"** und (optional) der **"Hyper-V"**-Features im System:
```
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V 
```

### <a name="step-4-reboot-your-system"></a>SCHRITT4: Starten Sie das System neu
Führen Sie zum Abschluss der Schritte für die Deinstallation/Bereinigung aus einer PowerShell-Sitzung mit erhöhten Rechten Folgendes aus:
```
Restart-Computer -Force
```
