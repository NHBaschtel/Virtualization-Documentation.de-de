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
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129260"
---
# <a name="docker-engine-on-windows"></a>Docker-Modul unter Windows

Das Andock Modul und der Client sind nicht im Lieferumfang von Windows enthalten und müssen einzeln installiert und konfiguriert werden. Das Docker-Modul kann zudem zahlreiche verschiedene Konfigurationen akzeptieren. Beispielsweise kann konfiguriert werden, wie der Daemon eingehende Anforderungen akzeptiert, und Sie können standardmäßige Netzwerkoptionen sowie Einstellungen für Debugging und Protokolle konfigurieren. Unter Windows können diese Konfigurationen in einer Konfigurationsdatei oder mit dem Windows-Dienststeuerungs-Manager angegeben werden. In diesem Dokument wird beschrieben, wie Sie das Andock Modul installieren und konfigurieren, und es werden auch einige Beispiele für häufig verwendete Konfigurationen bereitgestellt.

## <a name="install-docker"></a>Installieren von Docker

Sie benötigen Andockfenster, um mit Windows-Containern arbeiten zu können. Docker besteht aus dem Docker-Modul (dockerd.exe) und dem Docker-Client (docker.exe). Die einfachste Möglichkeit, alles zu installieren, finden Sie im Schnellstarthandbuch, das Ihnen dabei hilft, alles einzurichten und ihren ersten Container auszuführen.

- [Installieren von Docker](../quick-start/set-up-environment.md)

Informationen zu Skriptinstallationen finden Sie unter [Verwenden eines Skripts zum Installieren von Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee).

Bevor Sie docker verwenden können, müssen Sie die Container Bilder installieren. Weitere Informationen finden Sie unter [Dokumente für unsere Containerbasis Bilder](../manage-containers/container-base-images.md).

## <a name="configure-docker-with-a-configuration-file"></a>Konfigurieren des andockfensters mit einer Konfigurationsdatei

Die bevorzugte Methode zum Konfigurieren des Docker-Moduls in Windows ist die Verwendung einer Konfigurationsdatei. Die Konfigurationsdatei finden Sie unter „C:\ProgramData\Docker\config\daemon.json”. Sie können diese Datei erstellen, wenn Sie noch nicht vorhanden ist.

>[!NOTE]
>Nicht jede verfügbare docker-Konfigurationsoption gilt für Andockfenster unter Windows. Das folgende Beispiel zeigt die Konfigurationsoptionen, die angewendet werden. Weitere Informationen zur Konfiguration des Andock Moduls finden Sie unter [Konfigurationsdatei des docker-Daemons](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

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

Sie müssen der Konfigurationsdatei nur die gewünschten Konfigurationsänderungen hinzufügen. Im folgenden Beispiel wird das Andock Modul so konfiguriert, dass eingehende Verbindungen auf Port 2375 akzeptiert werden. Für alle weiteren Konfigurationsoptionen werden die Standardwerte verwendet.

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

Im folgenden Beispiel wird der andocker-Daemon so konfiguriert, dass Bilder und Container in einem alternativen Pfad gespeichert werden. Wenn keine Angabe erfolgt, lautet `c:\programdata\docker`der Standardwert.

```json
{    
    "data-root": "d:\\docker"
}
```

Im folgenden Beispiel wird der andocker-Daemon so konfiguriert, dass nur gesicherte Verbindungen über Port 2376 akzeptiert werden.

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Konfigurieren des andockfensters im Andock Dienst

Das Andock Modul kann auch durch Ändern des Andock Diensts mit `sc config`konfiguriert werden. Bei Verwendung dieser Methode werden die Flags des Docker-Moduls direkt im Docker-Dienst festgelegt. Führen Sie den folgenden Befehl in einem Eingabeaufforderungsfenster („cmd.exe“, nicht PowerShell) aus:

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Sie müssen diesen Befehl nicht ausführen, wenn Ihre Daemon. JSON-Datei bereits den `"hosts": ["tcp://0.0.0.0:2375"]` Eintrag enthält.

## <a name="common-configuration"></a>Allgemeine Konfiguration

Die folgenden Beispiele für Konfigurationsdateien zeigen allgemeine Docker-Konfigurationen. Diese können in einer einzigen Konfigurationsdatei kombiniert werden.

### <a name="default-network-creation"></a>Standardmäßige Netzwerk Erstellung

Verwenden Sie die folgende Konfiguration, um das Andock Modul so zu konfigurieren, dass kein Standard-NAT-Netzwerk erstellt wird.

```json
{
    "bridge" : "none"
}
```

Weitere Informationen finden Sie unter [Verwalten von Docker-Netzwerken](../container-networking/network-drivers-topologies.md).

### <a name="set-docker-security-group"></a>Docker-Sicherheitsgruppe einrichten

Wenn Sie sich beim andocker-Host angemeldet haben und die Docker-Befehle lokal ausführen, werden diese Befehle über eine Named Pipe ausgeführt. Standardmäßig können nur Mitglieder der Administratorengruppe über die Named Pipe auf das Docker-Modul zugreifen. Verwenden Sie das Flag `group`, um eine Sicherheitsgruppe mit diesem Zugriff festzulegen.

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

Weitere Informationen finden Sie unter [Windows-Konfigurationsdatei auf Docker.com](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file).

## <a name="how-to-uninstall-docker"></a>So deinstallieren Sie docker

In diesem Abschnitt erfahren Sie, wie Sie docker deinstallieren und eine vollständige Bereinigung der Docker Systemkomponenten von Ihrem Windows 10-oder Windows Server 2016-System durchführen.

>[!NOTE]
>Sie müssen alle Befehle in diesen Anweisungen in einer erhöhten PowerShell-Sitzung ausführen.

### <a name="prepare-your-system-for-dockers-removal"></a>Vorbereiten des Systems für das Entfernen des Dockers

Bevor Sie docker deinstallieren, stellen Sie sicher, dass auf Ihrem System keine Container ausgeführt werden.

Führen Sie die folgenden Cmdlets aus, um nach ausgeführten Containern zu suchen:

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

Darüber hinaus empfiehlt es sich, alle Container, Container Bilder, Netzwerke und Volumes aus Ihrem System zu entfernen, bevor Sie docker entfernen. Führen Sie dazu das folgende Cmdlet aus:

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Deinstallieren von Docker

Als nächstes müssen Sie docker tatsächlich deinstallieren.

So deinstallieren Sie docker unter Windows 10

- Wechseln Sie zu **Einstellungen** > -**apps** auf Ihrem Windows 10-Computer.
- Suchen Sie unter **apps #a0 Features** **Andockfenster für Windows** .
- Zu **docker für Windows** > -**Deinstallation** wechseln

So deinstallieren Sie docker unter Windows Server 2016:

Verwenden Sie in einer erhöhten PowerShell-Sitzung die Cmdlets **Deinstallationspaket** und **Deinstallations Modul** , um das docker Modul und den entsprechenden Paket Verwaltungsanbieter aus Ihrem System zu entfernen, wie im folgenden Beispiel gezeigt:

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Sie können den Paketanbieter finden, mit dem Sie docker installiert haben `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Bereinigen von andocker-Daten und Systemkomponenten

Nachdem Sie docker deinstalliert haben, müssen Sie die Standardnetzwerke von Docker entfernen, damit Ihre Konfiguration nicht auf Ihrem System verbleibt, nachdem docker nicht mehr vorhanden ist. Führen Sie dazu das folgende Cmdlet aus:

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

Führen Sie das folgende Cmdlet aus, um die Programm Daten von andockern aus Ihrem System zu entfernen:

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Sie können ebenfalls die optionalen Features von Windows entfernen, die Docker/Containern unter Windows zugeordnet sind.

Dies umfasst die Funktion "Container", die bei der Installation von Docker automatisch auf Windows 10-oder Windows Server 2016 aktiviert wird. Es kann ebenfalls das Feature "Hyper-V" beinhalten, das unter Windows10 automatisch aktiviert wird, wenn Docker installiert ist. Dies muss jedoch explizit auf Windows Server2016 aktiviert werden.

>[!IMPORTANT]
>[Das Hyper-V-Feature](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/) ist eine allgemeine Virtualisierungs-Funktion, die viel mehr als nur Container ermöglicht. Bevor Sie das Hyper-v-Feature deaktivieren, stellen Sie sicher, dass auf Ihrem System keine anderen virtualisierten Komponenten vorhanden sind, für die Hyper-v erforderlich ist.

So entfernen Sie Windows-Features unter Windows 10:

- Wechseln Sie **** > **zu Programme** > **und Funktionen** > der Systemsteuerung, um**Windows-Features zu aktivieren oder zu deaktivieren**.
- Suchen Sie den Namen der Features, die Sie deaktivieren möchten – in diesem Fall **Container** und (optional) **Hyper-V**.
- Deaktivieren Sie das Kontrollkästchen neben dem Namen des Features, das Sie deaktivieren möchten.
- Wählen Sie **"OK"** aus.

So entfernen Sie Windows-Features unter Windows Server 2016:

Führen Sie in einer erhöhten PowerShell-Sitzung die folgenden Cmdlets aus, um die **Container** und (optional) **Hyper-V-** Features Ihres Systems zu deaktivieren:

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>Starten Sie Ihr System neu.

Um die Deinstallation und Bereinigung abzuschließen, führen Sie das folgende Cmdlet aus einer erhöhten PowerShell-Sitzung aus, um Ihr System neu zu starten:

```powershell
Restart-Computer -Force
```
