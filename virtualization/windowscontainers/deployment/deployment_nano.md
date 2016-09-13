---
title: Bereitstellen von Windows-Containern unter Nano Server
description: Bereitstellen von Windows-Containern unter Nano Server
keywords: Docker, Container
author: neilpeterson
manager: timlt
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: 939a1b69f159504b998792adb95ccabc326db333
ms.openlocfilehash: 538fb27d6170f0a8dab5c189b90040e40c546e14

---

# Bereitstellung von Containerhosts: Nano Server

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

In diesem Dokument wird Schritt für Schritt eine sehr einfache Nano Server-Bereitstellung mit dem Windows-Containerfeature ausgeführt. Hierbei handelt es sich um ein fortgeschrittenes Thema, das ein Grundverständnis von Windows und Windows-Containern voraussetzt. Eine Einführung in Windows-Container finden Sie unter [Schnellstartanleitung: Windows-Container](../quick_start/quick_start.md).

## Vorbereiten von Nano Server

Im folgenden Abschnitt wird die Bereitstellung einer einfachen Nano Server-Konfiguration ausführlich beschrieben. Eine gründlichere Erklärung der Bereitstellungs- und Konfigurationsoptionen für Nano Server finden Sie unter [Getting Started with Nano Server] (Erste Schritte mit Nano Server) (https://technet.microsoft.com/en-us/library/mt126167.aspx).

### Erstellen einer Nano Server-VM

Laden Sie zunächst die Nano Server-Evaluierungs-VHD [hier](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/nano_eula) herunter. Erstellen Sie einen virtuellen Computer aus dieser VHD, starten Sie den virtuellen Computer, und verbinden Sie ihn mittels der Hyper-V-Verbindungsoption oder einer anderen Option passend zur verwendeten Virtualisierungsplattform.

### Erstellen einer Remote-PowerShell-Sitzung

Da Nano Server nicht über interaktive Anmeldefunktionen verfügt, werden alle Verwaltungsaktivitäten von einem Remotesystem aus unter Verwendung von PowerShell ausgeführt.

Fügen Sie das Nano Server-System zu den vertrauenswürdigen Hosts des Remotesystems hinzu. Ersetzen Sie die IP-Adresse durch die IP-Adresse der Nano Server-Instanz.

```none
Set-Item WSMan:\localhost\Client\TrustedHosts 192.168.1.50 -Force
```

Erstellen Sie die Remote-PowerShell-Sitzung.

```none
Enter-PSSession -ComputerName 192.168.1.50 -Credential ~\Administrator
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

Für die Arbeit mit Windows-Containern ist das Docker-Modul erforderlich. Installieren Sie das Docker-Modul mithilfe der folgenden Schritte.

Stellen Sie zunächst sicher, dass die Nano Server-Firewall für SMB konfiguriert wurde. Dies kann durch Ausführen dieses Befehls auf dem Nano Server-Host erfolgen.

```none
Set-NetFirewallRule -Name FPS-SMB-In-TCP -Enabled True
```

Erstellen Sie auf dem Nano Server-Host einen Ordner für die ausführbaren Docker-Dateien.

```none
New-Item -Type Directory -Path $env:ProgramFiles'\docker\'
```

Laden Sie das Docker-Modul und den Docker-Client herunter, und kopieren Sie die Dateien in das Verzeichnis „C:\Programme\docker\'“ auf dem Containerhost. 

> `Invoke-WebRequest` wird derzeit von Nano Server nicht unterstützt. Der Download muss auf einem Remotesystem ausgeführt werden, und die Dateien müssen auf den Nano Server-Host kopiert werden.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile .\docker-1.12.0.zip -UseBasicParsing
```

Extrahieren Sie das heruntergeladene Paket. Nachdem Sie diesen Schritt abgeschlossen haben, verfügen Sie über ein Verzeichnis, das die Dateien **dockerd.exe** und **docker.exe** enthält. Kopieren Sie diese beide Dateien in den Ordner **C:\Programme\docker\** auf dem Nano Server-Containerhost. 

```none
Expand-Archive .\docker-1.12.0.zip
```

Fügen Sie das Docker-Verzeichnis zum Systempfad auf der Nano Server-Instanz hinzu.

> Vergessen Sie nicht, wieder zur Nano Server-Remotesitzung zurückzukehren.

```none
# For quick use, does not require shell to be restarted.
$env:path += “;C:\program files\docker”

# For persistent use, will apply even after a reboot.
setx PATH $env:path /M
```

Installieren Sie Docker als Windows-Dienst.

```none
dockerd --register-service
```

Starten Sie den Docker-Dienst.

```none
Start-Service Docker
```

## Installieren von Basiscontainerimages

Basisimages des Betriebssystems dienen als Basis aller Windows Server- oder Hyper-V-Container. Basisimages stehen sowohl für Windows Server Core als auch für Nano Server als zugrunde liegendes Betriebssystem bereit und können mithilfe von `docker pull` installiert werden. Ausführliche Informationen zu Windows-Containerimages finden Sie unter [Verwalten von Containerimages](../management/manage_images.md).

Zum Herunterladen und Installieren des Basisimages für Nano Server führen Sie die folgenden Schritte aus:

```none
docker pull microsoft/nanoserver
```

> Zurzeit ist nur das Nano Server-Basisimage mit Nano Server-Containerhosts kompatibel.

## Verwalten von Docker unter Nano Server

Als bewährte Methode und um optimale Ergebnisse zu erzielen, verwalten Sie Docker unter Nano Server über ein Remotesystem. Zu diesem Zweck müssen die folgenden Schritte ausgeführt werden.

### Vorbereiten des Containerhosts

Erstellen Sie auf dem Containerhost eine Firewallregel für die Docker-Verbindung. Bei unsicheren Verbindungen wird Port `2375` verwendet, bei sicheren Verbindungen Port `2376`.

```none
netsh advfirewall firewall add rule name="Docker daemon " dir=in action=allow protocol=TCP localport=2375
```

Konfigurieren Sie das Docker-Modul so, dass eingehende Verbindungen über TCP akzeptiert werden.

Erstellen Sie zunächst eine `daemon.json`-Datei unter `c:\ProgramData\docker\config\daemon.json` auf dem Nano Server-Host.

```none
new-item -Type File c:\ProgramData\docker\config\daemon.json
```

Führen Sie als Nächstes den folgenden Befehl aus, um der `daemon.json` -Datei eine Verbindungskonfiguration hinzuzufügen. Damit wird das Docker-Modul so konfiguriert, dass eingehende Verbindungen über TCP-Port 2375 akzeptiert werden. Dies ist eine unsichere Verbindung und wird nicht empfohlen, für isolierte Tests kann sie jedoch verwendet werden. Weitere Informationen zum Sichern dieser Verbindung finden Sie unter [Protect the Docker Daemon on Docker.com](https://docs.docker.com/engine/security/https/) (Schützen des Docker-Daemon auf Docker.com).

```none
Add-Content 'c:\programdata\docker\config\daemon.json' '{ "hosts": ["tcp://0.0.0.0:2375", "npipe://"] }'
```

Starten Sie den Docker-Dienst neu.

```none
Restart-Service docker
```

### Vorbereiten des Remoteclients

Laden Sie den Docker-Client in das Remotesystem herunter, in dem Sie arbeiten werden.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Extrahieren Sie das komprimierte Paket.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Führen Sie die folgenden beiden Befehle aus, um das Docker-Verzeichnis zum Systempfad hinzuzufügen.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Nach Abschluss dieses Vorgangs kann mit dem Parameter `docker -H` auf den Docker-Remotehost zugegriffen werden.

```none
docker -H tcp://<IPADDRESS>:2375 run -it microsoft/nanoserver cmd
```

Sie können eine Umgebungsvariable `DOCKER_HOST` erstellen, sodass die Verwendung des Parameters `-H` nicht mehr erforderlich ist. Dazu können Sie den folgenden PowerShell-Befehl verwenden:

```none
$env:DOCKER_HOST = "tcp://<ipaddress of server>:2375"
```

Wenn diese Variable festgelegt ist, sieht der Befehl folgendermaßen aus.

```none
docker run -it microsoft/nanoserver cmd
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



<!--HONumber=Sep16_HO2-->


