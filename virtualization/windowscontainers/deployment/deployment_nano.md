---
title: Bereitstellen von Windows-Containern unter Nano Server
description: Bereitstellen von Windows-Containern unter Nano Server
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 07/06/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: e035a45e22eee04263861d935b338089d8009e92
ms.openlocfilehash: 876ffb4f4da32495fb77b735391203c33c78cff3

---

# Bereitstellung von Containerhosts: Nano Server

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

In diesem Dokument wird Schritt für Schritt eine sehr einfache Nano Server-Bereitstellung mit dem Windows-Containerfeature ausgeführt. Hierbei handelt es sich um ein fortgeschrittenes Thema, das ein Grundverständnis von Windows und Windows-Containern voraussetzt. Eine Einführung in Windows-Container finden Sie unter [Schnellstartanleitung: Windows-Container](../quick_start/quick_start.md).

## Vorbereiten von Nano Server

Im folgenden Abschnitt wird die Bereitstellung einer einfachen Nano Server-Konfiguration ausführlich beschrieben. Eine gründlichere Erklärung der Bereitstellungs- und Konfigurationsoptionen für Nano Server finden Sie unter [Getting Started with Nano Server] (Erste Schritte mit Nano Server) (https://technet.microsoft.com/en-us/library/mt126167.aspx).

### Erstellen einer Nano Server-VM

Laden Sie zunächst die Nano Server-Evaluierungs-VHD [hier](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula) herunter. Erstellen Sie einen virtuellen Computer aus dieser VHD, starten Sie den virtuellen Computer, und verbinden Sie ihn mittels der Hyper-V-Verbindungsoption oder einer anderen Option passend zur verwendeten Virtualisierungsplattform.

Als Nächstes muss das Administratorkennwort festgelegt werden. Drücken Sie dazu `F11` in der Nano Server-Wiederherstellungskonsole. Dadurch wird das Dialogfeld „Kennwort ändern“ bereitgestellt.

### Erstellen einer Remote-PowerShell-Sitzung

Da Nano Server nicht über interaktive Anmeldefunktionen verfügt, werden alle Verwaltungsaktivitäten von einer Remote-PowerShell-Sitzung aus abgeschlossen. Rufen Sie die IP-Adresse des Systems ab, das den Netzwerkabschnitt der Nano Server-Wiederherstellungskonsole verwendet, und führen Sie anschließend die folgenden Befehle auf dem Remotehost aus, um die Remotesitzung zu erstellen. Ersetzten Sie IPADDRESS mit der tatsächlichen IP-Adresse des Nano Server-Systems.

Fügen Sie das Nano Server-System den vertrauenswürdigen Hosts hinzu.

```none
set-item WSMan:\localhost\Client\TrustedHosts IPADDRESS -Force
```

Erstellen Sie die Remote-PowerShell-Sitzung.

```none
Enter-PSSession -ComputerName IPADDRESS -Credential ~\Administrator
```

Wenn Sie diese Schritte abgeschlossen haben, befinden Sie sich in der PowerShell-Remotesitzung mit dem Nano Server-System. Die restlichen Schritte dieses Dokuments werden, sofern nicht anders angemerkt, von der Remotesitzung aus stattfinden.


## Installieren des Containerfeatures

Der Anbieter für die Nano Server-Paketverwaltung erlaubt die Installation von Rollen und Funktionen auf Nano Server. Installieren Sie den Anbieter mithilfe dieses Befehls.

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

Stellen Sie, sobald er wieder verfügbar ist, die PowerShell-Remoteverbindung wieder her.

## Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Installieren Sie mithilfe der folgenden Schritte den Docker-Daemon und -Client.

Erstellen Sie auf dem Nano Server-Host einen Ordner für die ausführbaren Docker-Dateien.

```none
New-Item -Type Directory -Path $env:ProgramFiles'\docker\'
```

Laden Sie den Docker-Daemon und -Client herunter, und kopieren Sie die Dateien in das Verzeichnis „C:\Programme\docker\'“ auf dem Container-Host. 

**Hinweis**: Nano Server unterstützt `Invoke-WebRequest` derzeit nicht. Die Downloads müssen über ein Remotesystem ausgeführt und anschließend zum Nano Server-Host kopiert werden.

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile .\dockerd.exe
```

Laden Sie den Docker-Client herunter.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile .\docker.exe
```

Kopieren Sie den Docker-Daemon und Client nach dem Herunterladen in den Ordner 'C:\Programme\docker\' im Nano Server-Containerhost. Die Firewall von Nano Server muss konfiguriert werden, um eingehende SMB-Verbindungen zuzulassen. Dies kann mithilfe von PowerShell oder der Wiederherstellungskonsole von Nano Server abgeschlossen werden. 

```none
Set-NetFirewallRule -Name FPS-SMB-In-TCP -Enabled True
```

Die Dateien können jetzt mithilfe von Standard-SMB-Dateikopiermethoden kopiert werden.

Führen Sie diesen Befehl aus, nachdem die Datei „dockerd.exe“ in den Host kopiert wurde. Damit installieren Sie Docker als einen Windows-Dienst.

```none
& $env:ProgramFiles'\docker\dockerd.exe' --register-service
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
& $env:ProgramFiles'\docker\docker.exe' tag nanoserver:10.0.14300.1016 nanoserver:latest
```

## Verwalten von Docker unter Nano Server

Als bewährte Methode und um optimale Ergebnisse zu erzielen, verwalten Sie Docker unter Nano Server über ein Remotesystem. Zu diesem Zweck müssen die folgenden Schritte ausgeführt werden.

### Vorbereiten des Containerhosts

Erstellen Sie auf dem Containerhost eine Firewallregel für die Docker-Verbindung. Bei unsicheren Verbindungen wird Port `2375` verwendet, bei sicheren Verbindungen Port `2376`.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2376
```

Konfigurieren Sie den Docker-Daemon so, dass eingehende Verbindungen über TCP akzeptiert werden.

Erstellen Sie zunächst eine `daemon.json`-Datei unter `c:\ProgramData\docker\config\daemon.json` auf dem Nano Server-Host.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Führen Sie als Nächstes den folgenden Befehl aus, um der `daemon.json` -Datei eine Verbindungskonfiguration hinzuzufügen. Damit wird der Docker-Daemon so konfiguriert, dass eingehende Verbindungen über TCP-Port 2375 akzeptiert werden. Dies ist eine unsichere Verbindung und wird nicht empfohlen, für isolierte Tests kann sie jedoch verwendet werden. Weitere Informationen zum Sichern dieser Verbindung finden Sie unter [Protect the Docker Daemon on Docker.com](https://docs.docker.com/engine/security/https/) (Schützen des Docker-Daemon auf Docker.com).

```none
Add-Content 'c:\programdata\docker\config\daemon.json' '{ "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }'
```

Starten Sie den Docker-Dienst neu.

```none
Restart-Service docker
```

### Vorbereiten des Remoteclients

Erstellen Sie im Remotesystem, in dem Sie arbeiten werden, ein Verzeichnis das den Docker-Client enthalten soll.

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Laden Sie den Docker-Client in dieses Verzeichnis herunter.

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile "$env:ProgramFiles\docker\docker.exe"
```

Fügen Sie das Docker-Verzeichnis dem Systempfad hinzu.

```none
$env:Path += ";$env:ProgramFiles\Docker"
```

Starten Sie die PowerShell- oder Befehlssitzung neu, damit der geänderte Pfad erkannt wird.

Nach Abschluss dieses Vorgangs kann mit dem Parameter `docker -H` auf den Docker-Remotehost zugegriffen werden.

```none
docker -H tcp://<IPADDRESS>:2375 run -it nanoserver cmd
```

Sie können eine Umgebungsvariable `DOCKER_HOST` erstellen, sodass die Verwendung des Parameters `-H` nicht mehr erforderlich ist. Dazu können Sie den folgenden PowerShell-Befehl verwenden:

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

Wenn diese Variable festgelegt ist, sieht der Befehl folgendermaßen aus.

```none
docker run -it nanoserver cmd
```

## Hyper-V-Containerhost

Die Hyper-V-Rolle ist auf dem Containerhost erforderlich, um Hyper-V-Container bereitzustellen. Weitere Informationen zu Hyper-V-Containern finden Sie unter [Hyper-V-Container](../management/hyperv_container.md).

Wenn der Windows-Containerhost selbst ein virtueller Hyper-V-Computer ist, muss die geschachtelte Virtualisierung aktiviert werden. Weitere Informationen dazu finden Sie unter [Geschachtelte Virtualisierung](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).


Installieren Sie die Hyper-V-Rolle auf den Nano Server-Containerhost.

```none
Install-NanoServerPackage Microsoft-NanoServer-Compute-Package
```

Der Nano Server-Host muss nach der Installation der Hyper-V-Rolle neu gestartet werden.

```none
Restart-Computer
```


<!--HONumber=Jul16_HO1-->


