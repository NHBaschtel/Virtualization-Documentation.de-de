---
title: Bereitstellen von Windows-Containern unter Nano Server
description: Bereitstellen von Windows-Containern unter Nano Server
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
---

# Bereitstellung von Containerhosts: Nano Server

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

Bevor Sie einen Windows-Container unter Nano Server konfigurieren können, benötigen Sie ein System, auf dem Nano Server ausgeführt wird, sowie eine PowerShell-Remoteverbindung mit diesem System.

Weitere Informationen zur Bereitstellung und zum Herstellen einer Verbindung mit Nano Server finden Sie unter [Getting Started with Nano Server]( https://technet.microsoft.com/en-us/library/mt126167.aspx) (Erste Schritte mit Nano Server).

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

Der Nano Server-Host muss nach der Installation dieser Features neu gestartet werden.

## Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Installieren Sie mithilfe der folgenden Schritte den Docker-Daemon.

Laden Sie den Docker-Daemon herunter, und kopieren Sie ihn in das Verzeichnis `$env:SystemRoot\system32\` auf dem Containerhost. Nano Server unterstützt derzeit `Invoke-Webrequest` nicht, dies muss über ein Remotesystem ausgeführt werden.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Installieren Sie Docker als Windows-Dienst.

```none
dockerd.exe --register-service
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

Abschließend muss das Image als neueste Version („latest“) gekennzeichnet werden. Führen Sie dazu den folgenden Befehl aus.

```none
docker tag nanoserver:10.0.14300.1010 nanoserver:latest
```

## Hyper-V-Containerhost

Um Hyper-V-Container bereitzustellen, ist die Hyper-V-Rolle erforderlich. Wenn der Windows-Containerhost selbst ein virtueller Hyper-V-Computer ist, muss die geschachtelte Virtualisierung aktiviert werden, bevor die Hyper-V-Rolle installiert werden kann. Weitere Informationen dazu finden Sie unter „Geschachtelte Virtualisierung“.

### Geschachtelte Virtualisierung

Das folgende Skript konfiguriert die geschachtelte Virtualisierung für den Containerhost. Das Skript wird auf dem Hyper-V-Computer ausgeführt, der den virtuellen Containerhostcomputer hostet. Stellen Sie sicher, dass der virtuelle Containerhostcomputer ausgeschaltet ist, wenn dieses Skript ausgeführt wird.

```none
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### Aktivieren der Hyper-V-Rolle

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

## Verwalten von Docker unter Nano Server

**Vorbereiten des Docker-Daemons:**

Um optimale Ergebnisse zu erzielen, verwalten Sie Docker unter Nano Server über ein Remotesystem. Zu diesem Zweck müssen die folgenden Schritte ausgeführt werden.

Erstellen Sie auf dem Containerhost eine Firewallregel für die Docker-Verbindung. Verwenden Sie Port `2375` für eine unsichere Verbindung oder Port `2376` für eine sichere Verbindung.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

Konfigurieren Sie den Docker-Daemon so, dass eingehende Verbindungen über TCP akzeptiert werden.

Erstellen Sie zuerst eine `daemon.json`-Datei in `c:\ProgramData\docker\config\daemon.json`.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Kopieren Sie anschließend den folgenden JSON-Code in die Datei. Damit wird der Docker-Daemon so konfiguriert, dass eingehende Verbindungen über TCP-Port 2375 akzeptiert werden. Dies ist eine unsichere Verbindung und wird nicht empfohlen, für isolierte Tests kann sie jedoch verwendet werden.

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

Laden Sie den Docker-Client auf das Remoteverwaltungssystem herunter.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:SystemRoot\system32\docker.exe
```

Nach Abschluss dieses Vorgangs kann mit dem Parameter `Docker -H` auf den Docker-Daemon zugegriffen werden.

```none
docker -H tcp://10.0.0.5:2376 run -it nanoserver cmd
```

Sie können eine Umgebungsvariable `DOCKER_HOST` erstellen, sodass die Verwendung des Parameters `-H` nicht mehr erforderlich ist. Dazu können Sie den folgenden PowerShell-Befehl verwenden:

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server:2376"
```

Wenn diese Variable festgelegt ist, sieht der Befehl folgendermaßen aus.

```none
docker run -it nanoserver cmd
```

<!--HONumber=May16_HO5-->


