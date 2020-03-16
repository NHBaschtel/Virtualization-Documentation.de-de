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
ms.openlocfilehash: c84a6652b5918238ee8ef6e1fa7a9b2aa596aefd
ms.sourcegitcommit: 8eedfdc1fda9d0abb36e28dc2b5fb39891777364
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/15/2020
ms.locfileid: "79402881"
---
# <a name="docker-engine-on-windows"></a>Docker-Modul unter Windows

Die Docker-Engine und der-Client sind nicht in Windows enthalten und müssen einzeln installiert und konfiguriert werden. Das Docker-Modul kann zudem zahlreiche verschiedene Konfigurationen akzeptieren. Beispielsweise kann konfiguriert werden, wie der Daemon eingehende Anforderungen akzeptiert, und Sie können standardmäßige Netzwerkoptionen sowie Einstellungen für Debugging und Protokolle konfigurieren. Unter Windows können diese Konfigurationen in einer Konfigurationsdatei oder mit dem Windows-Dienststeuerungs-Manager angegeben werden. In diesem Dokument wird ausführlich beschrieben, wie Sie die Docker-Engine installieren und konfigurieren. Außerdem finden Sie hier einige Beispiele für häufig verwendete Konfigurationen.

## <a name="install-docker"></a>Installieren von Docker

Sie benötigen Docker, um mit Windows-Containern zu arbeiten. Docker besteht aus dem Docker-Modul (dockerd.exe) und dem Docker-Client (docker.exe). Die einfachste Möglichkeit, alles zu installieren, finden Sie im Schnellstart Handbuch, das Ihnen hilft, alles einzurichten und ihren ersten Container auszuführen.

- [Installieren von Docker](../quick-start/set-up-environment.md)

Informationen zu Skript Installationen finden [Sie unter Verwenden eines Skripts zum Installieren von Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Bevor Sie docker verwenden können, müssen Sie die Container Images installieren. Weitere Informationen finden Sie in der Dokumentation [zu den Containerbasis Images](../manage-containers/container-base-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Konfigurieren von Docker mit einer Konfigurationsdatei

Die bevorzugte Methode zum Konfigurieren des Docker-Moduls in Windows ist die Verwendung einer Konfigurationsdatei. Die Konfigurationsdatei finden Sie unter „C:\ProgramData\Docker\config\daemon.json”. Sie können diese Datei erstellen, wenn Sie nicht bereits vorhanden ist.

>[!NOTE]
>Nicht jede verfügbare docker-Konfigurationsoption gilt für docker unter Windows. Das folgende Beispiel zeigt die Konfigurationsoptionen, die angewendet werden. Weitere Informationen zur Konfiguration der Docker-Engine finden Sie unter [docker-daemonkonfigurationsdatei](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

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

Sie müssen nur die gewünschten Konfigurationsänderungen zur Konfigurationsdatei hinzufügen. Im folgenden Beispiel wird die Docker-Engine so konfiguriert, dass eingehende Verbindungen an Port 2375 akzeptiert werden. Für alle weiteren Konfigurationsoptionen werden die Standardwerte verwendet.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Im folgenden Beispiel wird der Docker-Daemon so konfiguriert, dass Bilder und Container in einem alternativen Pfad aufbewahrt werden. Wenn nicht angegeben, wird der Standardwert `c:\programdata\docker`.

```json
{    
    "data-root": "d:\\docker"
}
```

Im folgenden Beispiel wird der Docker-Daemon so konfiguriert, dass nur gesicherte Verbindungen über Port 2376 angenommen werden.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Konfigurieren von Docker auf dem docker-Dienst

Die Docker-Engine kann auch konfiguriert werden, indem der Docker-Dienst mit `sc config`geändert wird. Bei Verwendung dieser Methode werden die Flags des Docker-Moduls direkt im Docker-Dienst festgelegt. Führen Sie den folgenden Befehl in einem Eingabeaufforderungsfenster („cmd.exe“, nicht PowerShell) aus:

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Sie müssen diesen Befehl nicht ausführen, wenn die Datei "Daemon. JSON" bereits den `"hosts": ["tcp://0.0.0.0:2375"]` Eintrag enthält.

## <a name="common-configuration"></a>Allgemeine Konfiguration

Die folgenden Beispiele für Konfigurationsdateien zeigen allgemeine Docker-Konfigurationen. Diese können in einer einzigen Konfigurationsdatei kombiniert werden.

### <a name="default-network-creation"></a>Standard Netzwerk Erstellung

Verwenden Sie die folgende Konfiguration, um die Docker-Engine so zu konfigurieren, dass kein NAT-Standard Netzwerk erstellt wird.

```json
{
    "bridge" : "none"
}
```

Weitere Informationen finden Sie unter [Verwalten von Docker-Netzwerken](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Festlegen der Docker-Sicherheitsgruppe

Wenn Sie sich beim docker-Host angemeldet haben und die Docker-Befehle lokal ausführen, werden diese Befehle über eine Named Pipe ausgeführt. Standardmäßig können nur Mitglieder der Administratorengruppe über die Named Pipe auf das Docker-Modul zugreifen. Verwenden Sie das Flag `group`, um eine Sicherheitsgruppe mit diesem Zugriff festzulegen.

```json
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>Proxykonfiguration

Erstellen Sie eine Windows-Umgebungsvariable namens `docker search` oder `docker pull` und den Wert der Proxyinformationen, um Proxyinformationen für `HTTP_PROXY` und `HTTPS_PROXY` festzulegen. Sie können dies in PowerShell tun, indem Sie einen Befehl ähnlich dem folgenden verwenden:

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

Starten Sie den Docker-Dienst neu, sobald die Variable festgelegt wurde.

```powershell
Restart-Service docker
```

Weitere Informationen finden Sie unter [Windows-Konfigurationsdatei auf Docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="how-to-uninstall-docker"></a>Deinstallieren von Docker

In diesem Abschnitt erfahren Sie, wie Sie docker deinstallieren und eine vollständige Bereinigung der Docker-Systemkomponenten von Ihrem Windows 10-oder Windows Server 2016-System ausführen.

>[!NOTE]
>Sie müssen alle Befehle in diesen Anweisungen in einer PowerShell-Sitzung mit erhöhten Rechten ausführen.

### <a name="prepare-your-system-for-dockers-removal"></a>Vorbereiten Ihres Systems für das Entfernen von Docker

Stellen Sie vor der Deinstallation von Docker sicher, dass auf Ihrem System keine Container ausgeführt werden.

Führen Sie die folgenden Cmdlets aus, um die Ausführung von Containern zu überprüfen:

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

Außerdem empfiehlt es sich, vor dem Entfernen von Docker alle Container, Container Images, Netzwerke und Volumes aus Ihrem System zu entfernen. Hierzu können Sie das folgende Cmdlet ausführen:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Deinstallieren von Docker

Als nächstes müssen Sie docker wirklich deinstallieren.

So deinstallieren Sie docker unter Windows 10

- Wechseln Sie zu **Einstellungen** > **apps** auf Ihrem Windows 10-Computer.
- Suchen Sie unter **Apps & Features**nach **docker für Windows**
- Wechseln Sie zu **docker für Windows** > **deinstallieren** .

So deinstallieren Sie docker unter Windows Server 2016:

Verwenden Sie in einer PowerShell-Sitzung mit erhöhten Rechten die Cmdlets " **Uninstall-Package** " und " **Uninstall-Module** ", um das docker-Modul und den entsprechenden Paketverwaltung Anbieter aus Ihrem System zu entfernen, wie im folgenden Beispiel gezeigt:

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Den Paketanbieter, den Sie zum Installieren von Docker verwendet haben, finden Sie unter `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Bereinigen von Docker-Daten und-Systemkomponenten

Nachdem Sie docker deinstalliert haben, müssen Sie die Standard Netzwerke von Docker entfernen, damit Ihre Konfiguration nach dem Ausfall von Docker nicht auf Ihrem System verbleiben kann. Hierzu können Sie das folgende Cmdlet ausführen:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Führen Sie das folgende Cmdlet aus, um die Programm Daten von Docker aus Ihrem System zu entfernen:

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Möglicherweise möchten Sie auch die optionalen Windows-Funktionen entfernen, die Docker/Container unter Windows zugeordnet sind.

Dazu gehört auch das Feature "Container", das bei der Installation von Docker automatisch auf allen Windows 10-oder Windows Server-2016 Servern aktiviert wird. Es kann ebenfalls das Feature "Hyper-V" beinhalten, das unter Windows 10 automatisch aktiviert wird, wenn Docker installiert ist. Dies muss jedoch explizit auf Windows Server 2016 aktiviert werden.

>[!IMPORTANT]
>[Das Hyper-V-Feature](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/) ist ein allgemeines Virtualisierungsfeature, das viel mehr als nur Container ermöglicht. Bevor Sie das Hyper-v-Feature deaktivieren, stellen Sie sicher, dass auf Ihrem System keine anderen virtualisierten Komponenten vorhanden sind, die Hyper-v erfordern.

So entfernen Sie Windows-Features unter Windows 10:

- Wechseln Sie zur **Systemsteuerung** > **Programme** > **Programme und Funktionen** , > aktivieren **oder deaktivieren Sie Windows-Funktionen**.
- Suchen Sie den Namen der Features, die Sie deaktivieren möchten – in diesem Fall **Container** und (optional) **Hyper-V**.
- Deaktivieren Sie das Kontrollkästchen neben dem Namen des Features, das Sie deaktivieren möchten.
- Wählen Sie **"OK"** aus.

So entfernen Sie Windows-Features unter Windows Server 2016:

Führen Sie in einer PowerShell-Sitzung mit erhöhten Rechten die folgenden Cmdlets aus, um die **Container** und (optional) **Hyper-V-** Features von Ihrem System zu deaktivieren:

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>System neu starten

Führen Sie das folgende Cmdlet aus einer PowerShell-Sitzung mit erhöhten Rechten aus, um die Installation und die Bereinigung abzuschließen, um das System neu zu starten:

```powershell
Restart-Computer -Force
```
