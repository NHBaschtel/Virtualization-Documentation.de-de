---
title: Konfigurieren von Docker unter Windows
description: Konfigurieren von Docker unter Windows
keywords: Docker, Container
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: a04d356415e7bed84980747edc927cc1eaa1e7c1
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621088"
---
# <a name="docker-engine-on-windows"></a>Docker-Modul unter Windows

Das Docker-Modul und der Client nicht in Windows enthalten sind, und müssen werden einzeln installiert und konfiguriert. Das Docker-Modul kann zudem zahlreiche verschiedene Konfigurationen akzeptieren. Beispielsweise kann konfiguriert werden, wie der Daemon eingehende Anforderungen akzeptiert, und Sie können standardmäßige Netzwerkoptionen sowie Einstellungen für Debugging und Protokolle konfigurieren. Unter Windows können diese Konfigurationen in einer Konfigurationsdatei oder mit dem Windows-Dienststeuerungs-Manager angegeben werden. Dieses Dokument enthält Informationen zum Installieren und konfigurieren das Docker-Modul und bietet auch einige Beispiele für häufig verwendete Konfigurationen.

## <a name="install-docker"></a>Installieren von Docker

Sie benötigen Docker für die Arbeit mit Windows-Containern. Docker besteht aus dem Docker-Modul (dockerd.exe) und dem Docker-Client (docker.exe). Die einfachste Möglichkeit, alles zu installieren, ist im Schnellstart Anleitungen, mit die Hilfe Sie alles eingerichtet und Ausführen Ihres ersten Containers.

- [Windows-Container unter Windows Server 2019](../quick-start/quick-start-windows-server.md)
- [Windows-Container unter Windows 10](../quick-start/quick-start-windows-10.md)

Über die Skriptbasierte Installationen finden Sie unter [Verwendung eines Skripts Docker EE zu installieren](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Bevor Sie Docker verwenden können, müssen Sie die Container-Images installieren. Weitere Informationen finden Sie unter [der Schnellstart-Anleitung für die Verwendung von Images](../quick-start/quick-start-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Konfigurieren von Docker mit einer Konfigurationsdatei

Die bevorzugte Methode zum Konfigurieren des Docker-Moduls in Windows ist die Verwendung einer Konfigurationsdatei. Die Konfigurationsdatei finden Sie unter „C:\ProgramData\Docker\config\daemon.json”. Sie können diese Datei erstellen, wenn es nicht bereits vorhanden ist.

>[!NOTE]
>Nicht jede verfügbare Docker-Konfigurationsoption gilt für Docker unter Windows. Das folgende Beispiel zeigt die Konfigurationsoptionen, die angewendet werden. Weitere Informationen zur Konfiguration des Docker-Modul finden Sie in der [Docker-Daemon-Konfigurationsdatei](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

```json
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

Sie müssen nur die gewünschten konfigurationsänderungen der Konfigurationsdatei hinzu. Im folgende Beispiel wird z. B. das Docker-Modul, um eingehende Verbindungen über Port 2375 akzeptiert konfiguriert. Für alle weiteren Konfigurationsoptionen werden die Standardwerte verwendet.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Ebenso konfiguriert das folgende Beispiel den Docker-Daemon, um Images und Container in einem alternativen Pfad zu halten. Wenn nicht angegeben, wird standardmäßig `c:\programdata\docker`.

```json
{    
    "data-root": "d:\\docker"
}
```

Das folgende Beispiel konfiguriert den Docker-Daemon, um nur sichere Verbindungen über Port 2376 akzeptiert.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Konfigurieren von Docker über den Docker-Dienst

Das Docker-Modul kann auch konfiguriert werden, durch Ändern des Docker-Diensts mit `sc config`. Bei Verwendung dieser Methode werden die Flags des Docker-Moduls direkt im Docker-Dienst festgelegt. Führen Sie den folgenden Befehl in einem Eingabeaufforderungsfenster („cmd.exe“, nicht PowerShell) aus:

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Sie müssen diesen Befehl ausführen, wenn die Datei "Daemon.JSON" bereits enthält die `"hosts": ["tcp://0.0.0.0:2375"]` Eintrag.

## <a name="common-configuration"></a>Allgemeine Konfiguration

Die folgenden Beispiele für Konfigurationsdateien zeigen allgemeine Docker-Konfigurationen. Diese können in einer einzigen Konfigurationsdatei kombiniert werden.

### <a name="default-network-creation"></a>Standardnetzwerks

Um das Docker-Modul so konfigurieren, dass es kein NAT-Standardnetzwerk erstellt, verwenden Sie die folgende Konfiguration.

```json
{
    "bridge" : "none"
}
```

Weitere Informationen finden Sie unter [Verwalten von Docker-Netzwerken](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Festlegen der Docker-Sicherheitsgruppe

Wenn Sie den Docker-Host angemeldet haben und Docker-Befehle lokal ausführen, werden diese Befehle über eine named Pipe ausgeführt. Standardmäßig können nur Mitglieder der Administratorengruppe über die Named Pipe auf das Docker-Modul zugreifen. Verwenden Sie das Flag `group`, um eine Sicherheitsgruppe mit diesem Zugriff festzulegen.

```json
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

Weitere Informationen finden Sie in der [Windows-Konfigurationsdatei auf Docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="how-to-uninstall-docker"></a>So deinstallieren von Docker

In diesem Abschnitt erfahren Sie, wie zum Deinstallieren von Docker und eine vollständige Bereinigung der Docker-Systemkomponenten aus dem Windows 10 oder Windows Server 2016-System durchzuführen.

>[!NOTE]
>Sie müssen alle Befehle in diese Anweisungen aus einer PowerShell-Sitzung mit erhöhten Rechten ausführen.

### <a name="prepare-your-system-for-dockers-removal"></a>Vorbereiten des Systems für das Entfernen von Docker

Bevor Sie Docker deinstallieren, stellen Sie sicher, dass keine Container auf Ihrem System ausgeführt werden.

Führen Sie die folgenden Cmdlets für die Ausführung von Containern überprüfen:

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

Es ist auch sollten Sie alle Container, containerimages, Netzwerke und Volumes aus dem System zu entfernen, bevor Docker entfernt. Sie können dies tun, indem das folgende Cmdlet ausführen:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Deinstallieren von Docker

Als Nächstes müssen Sie tatsächlich Deinstallieren von Docker.

Zum Deinstallieren von Docker unter Windows 10

- Wechseln Sie zu **Einstellungen** > **Apps** auf Ihrem Windows 10-Computer
- Finden Sie unter **Apps & Features**die **Docker für Windows**
- Wechseln Sie zu **Docker für Windows** > **Deinstallieren**

Zum Deinstallieren von Docker unter Windows Server 2016:

Verwenden Sie aus einer PowerShell-Sitzung mit erhöhten Rechten die **Deinstallations-Paket** und **Deinstallation-Modul** -Cmdlets um das Docker-Modul und den entsprechenden Paketverwaltung aus dem System zu entfernen, wie im folgenden Beispiel dargestellt:

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Sie können den Anbieter des Pakets suchen, die Sie zum Installieren von Docker verwendet `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Bereinigen von dockerdaten und Systemkomponenten

Nachdem Sie Docker deinstallieren, müssen Sie Docker Netzwerken zu entfernen, damit deren Konfiguration auf Ihrem System bleiben wird nicht, nachdem Docker entfernt wurde. Sie können dies tun, indem das folgende Cmdlet ausführen:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Führen Sie das folgende Cmdlet, um Docker Programmdaten aus dem System zu entfernen:

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Sie können ebenfalls die optionalen Features von Windows entfernen, die Docker/Containern unter Windows zugeordnet sind.

Dazu gehören das Feature "Container", das automatisch auf allen Windows 10 oder Windows Server 2016 aktiviert wird, wenn Docker installiert ist. Es kann ebenfalls das Feature "Hyper-V" beinhalten, das unter Windows10 automatisch aktiviert wird, wenn Docker installiert ist. Dies muss jedoch explizit auf Windows Server2016 aktiviert werden.

>[!IMPORTANT]
>[Das Hyper-V-Feature](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/) ist eine allgemeine Virtualisierungsfunktion, die sehr viel mehr als nur Container aktiviert. Stellen Sie bevor Sie deaktivieren die Hyper-V-Feature sicher, dass keine anderen virtualisierten Komponenten auf Ihrem System, die Hyper-V erfordern.

So entfernen Sie Windows-Features unter Windows 10:

- Wechseln Sie zu **Systemsteuerung** > **Programme** > **Programme und Features** > **Windows-Funktionen ein- oder ausschalten**.
- Suchen Sie den Namen des Features oder Funktionen, die Sie deaktivieren möchten – in diesem Fall **Container** und (optional) **Hyper-V**.
- Deaktivieren Sie das Kontrollkästchen neben dem Namen des Features, die Sie deaktivieren möchten.
- Klicken Sie auf **"OK"**

So entfernen Sie Windows-Features unter Windows Server 2016:

Führen Sie eine PowerShell-Sitzung mit erhöhten Rechten die folgenden Cmdlets, um die **Container** und (optional) **Hyper-V** -Features aus dem System zu deaktivieren:

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>Neustart des Systems

Um die Deinstallation und Bereinigung abgeschlossen haben, führen Sie das folgende Cmdlet aus einer PowerShell-Sitzung mit erhöhten Ihr System neu gestartet:

```powershell
Restart-Computer -Force
```
