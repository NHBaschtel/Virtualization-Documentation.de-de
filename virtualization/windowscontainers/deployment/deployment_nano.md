---
title: Bereitstellen von Windows-Containern unter Nano Server
description: Bereitstellen von Windows-Containern unter Nano Server
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/17/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
ms.sourcegitcommit: 1ba6af300d0a3eba3fc6d27598044f983a4c9168
ms.openlocfilehash: f2790186aa641378b1981a1f946665ca46fdbd73

---

# Bereitstellung von Containerhosts: Nano Server

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

Bevor Sie einen Windows-Container unter Nano Server konfigurieren können, benötigen Sie ein System, auf dem Nano Server ausgeführt wird, sowie eine PowerShell-Remoteverbindung mit diesem System. Weitere Informationen zur Bereitstellung und zum Herstellen einer Verbindung mit Nano Server finden Sie unter [Getting Started with Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx) (Erste Schritte mit Nano Server).

Eine Evaluierungsversion von Nano Server finden Sie [hier](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula).

## Installieren des Containerfeatures

Installieren Sie den Anbieter für die Nano Server-Paketverwaltung.

```none
Install-PackageProvider NanoServerPackage
```

Nach der Installation des Paketanbieters installieren Sie das Containerfeature.

```none
Install-NanoServerPackage -Name Microsoft-NanoServer-Containers-Package
```

Der Nano Server-Host muss nach der Installation der Containerfeatures neu gestartet werden.

```none
Restart-Computer
```

## Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Installieren Sie mithilfe der folgenden Schritte den Docker-Daemon und -Client.

Erstellen Sie auf dem Nano Server-Host einen Ordner für die ausführbaren Docker-Dateien.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Laden Sie den Docker-Daemon und -Client herunter, und kopieren Sie die Dateien in das Verzeichnis „C:\Programme\docker\'“ auf dem Container-Host. 

**Hinweis**: Nano Server unterstützt derzeit `Invoke-WebRequest` nicht, Die Downloads müssen über ein Remotesystem ausgeführt werden.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Laden Sie den Docker-Client herunter.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

Nachdem der Docker-Daemon und -Client heruntergeladen und auf dem Nano Server-Containerhost kopiert wurden, führen Sie diesen Befehl auf dem Host aus, um Docker als Windows-Dienst zu installieren.

```none
& 'C:\Program Files\docker\dockerd.exe' --register-service
```

Starten Sie den Docker-Dienst.

```none
Start-Service Docker
```

## Installieren von Basisimages für Container

Basisimages des Betriebssystems dienen als Basis aller Windows Server- oder Hyper-V-Container. Basisimages stehen sowohl für Windows Server Core als auch für Nano Server als zugrunde liegendes Betriebssystem bereit und können mithilfe des Containerimageanbieters installiert werden. Ausführliche Informationen zu Windows-Containerimages finden Sie unter [Verwalten von Containerimages](../management/manage_images.md).

Der folgende Befehl kann verwendet werden, um den Containerimageanbieter zu installieren.

```none
Install-PackageProvider ContainerImage -Force
```

Zum Herunterladen und Installieren des Basisimages für Nano Server führen Sie die folgenden Schritte aus:

```none
Install-ContainerImage -Name NanoServer
```

**Hinweis**: Zurzeit ist nur das Nano Server-Basisimage mit Nano Server-Containerhosts kompatibel.

Starten Sie den Docker-Dienst neu.

```none
Restart-Service Docker
```

Markieren Sie das Nano Server-Basisimage als aktuellstes Image.

```none
& 'C:\Program Files\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## Verwalten von Docker unter Nano Server

Als bewährte Methode und um optimale Ergebnisse zu erzielen, verwalten Sie Docker unter Nano Server über ein Remotesystem. Zu diesem Zweck müssen die folgenden Schritte ausgeführt werden.

**Vorbereiten des Docker-Daemons:**

Erstellen Sie auf dem Containerhost eine Firewallregel für die Docker-Verbindung. Verwenden Sie Port `2375` für eine unsichere Verbindung oder Port `2376` für eine sichere Verbindung.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

Konfigurieren Sie den Docker-Daemon so, dass eingehende Verbindungen über TCP akzeptiert werden.

Erstellen Sie zuerst eine `daemon.json`-Datei in `c:\ProgramData\docker\config\daemon.json`.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Kopieren Sie anschließend den folgenden JSON-Code in die Konfigurationsdatei. Damit wird der Docker-Daemon so konfiguriert, dass eingehende Verbindungen über TCP-Port 2375 akzeptiert werden. Dies ist eine unsichere Verbindung und wird nicht empfohlen, für isolierte Tests kann sie jedoch verwendet werden.

```none
{
    "hosts": ["tcp://0.0.0.0:2375", "npipe://"]
}
```

Das folgende Beispiel konfiguriert eine sichere Remoteverbindung. Die TLS-Zertifikate müssen erstellt und in die richtigen Speicherorte kopiert werden. Weitere Informationen finden Sie unter [Docker-Daemon unter Windows](./docker_windows.md).

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

Starten Sie den Docker-Dienst neu.

```none
Restart-Service docker
```

**Vorbereiten des Docker-Clients:**

Erstellen Sie ein Verzeichnis zum Speichern des Docker-Clients.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Laden Sie den Docker-Client auf das Remoteverwaltungssystem herunter.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "C:\Program Files\docker\docker.exe"
```

Fügen Sie das Docker-Verzeichnis dem Systempfad hinzu.

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Starten Sie die PowerShell- oder Befehlssitzung neu, damit der geänderte Pfad erkannt wird.

Nach Abschluss dieses Vorgangs kann mit dem Parameter `docker -H` auf den Docker-Remotehost zugegriffen werden.

```none
docker -H tcp://10.0.0.5:2375 run -it nanoserver cmd
```

Sie können eine Umgebungsvariable `DOCKER_HOST` erstellen, sodass die Verwendung des Parameters `-H` nicht mehr erforderlich ist. Dazu können Sie den folgenden PowerShell-Befehl verwenden:

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2375"
```

Wenn diese Variable festgelegt ist, sieht der Befehl folgendermaßen aus.

```none
docker run -it nanoserver cmd
```

## Hyper-V-Containerhost

Um Hyper-V-Container bereitzustellen, ist die Hyper-V-Rolle erforderlich. Weitere Informationen zu Hyper-V-Containern finden Sie unter [Hyper-V-Container](../management/hyperv_container.md).

Wenn der Windows-Containerhost selbst ein virtueller Hyper-V-Computer ist, muss die geschachtelte Virtualisierung aktiviert werden. Weitere Informationen dazu finden Sie unter [Geschachtelte Virtualisierung](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).


Installieren der Hyper-V-Rolle:

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

Der Nano Server-Host muss nach der Installation der Hyper-V-Rolle neu gestartet werden.

```none
Restart-Computer
```






<!--HONumber=Jun16_HO3-->


