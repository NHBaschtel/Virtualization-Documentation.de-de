---
title: Bereitstellen von Windows-Containern unter Nano Server
description: Bereitstellen von Windows-Containern unter Nano Server
keywords: Docker, Container
author: neilpeterson
manager: timlt
ms.date: 09/28/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b82acdf9-042d-4b5c-8b67-1a8013fa1435
translationtype: Human Translation
ms.sourcegitcommit: a2c78d3945f1d5b0ebe2a4af480802f8c0c656c2
ms.openlocfilehash: a9d398de94cb0d6c54c2e82f4a024bb65de9806d

---

# Bereitstellung von Containerhosts: Nano Server

In diesem Dokument wird Schritt für Schritt eine sehr einfache Nano Server-Bereitstellung mit dem Windows-Containerfeature ausgeführt. Hierbei handelt es sich um ein fortgeschrittenes Thema, das ein Grundverständnis von Windows und Windows-Containern voraussetzt. Eine Einführung in Windows-Container finden Sie unter [Schnellstartanleitung: Windows-Container](../quick_start/quick_start.md).

## Vorbereiten von Nano Server

Im folgenden Abschnitt wird die Bereitstellung einer einfachen Nano Server-Konfiguration ausführlich beschrieben. Eine gründlichere Erklärung der Bereitstellungs- und Konfigurationsoptionen für Nano Server finden Sie unter [Getting Started with Nano Server] (Erste Schritte mit Nano Server) (https://technet.microsoft.com/en-us/library/mt126167.aspx).

### Erstellen einer Nano Server-VM

Laden Sie zunächst die Nano Server-Evaluierungs-VHD [hier](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016) herunter. Erstellen Sie einen virtuellen Computer aus dieser VHD, starten Sie den virtuellen Computer, und verbinden Sie ihn mittels der Hyper-V-Verbindungsoption oder einer anderen Option passend zur verwendeten Virtualisierungsplattform.

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

### Installieren von Windows-Updates

Wichtige Updates sind erforderlich, damit das Feature „Windows-Container“ funktioniert. Diese Updates können installiert werden, indem die folgenden Befehle ausgeführt werden.

```none
$sess = New-CimInstance -Namespace root/Microsoft/Windows/WindowsUpdate -ClassName MSFT_WUOperationsSession
Invoke-CimMethod -InputObject $sess -MethodName ApplyApplicableUpdates
```

Starten Sie das System neu, nachdem die Updates eingespielt wurden.

```none
Restart-Computer
```

Stellen Sie, sobald er wieder verfügbar ist, die PowerShell-Remoteverbindung wieder her.

## Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Verwenden Sie das [PowerShell-Modul von OneGet](https://github.com/oneget/oneget), um Docker zu installieren. Der Anbieter aktiviert die Containerfunktion auf Ihrem Computer und installiert Docker. Dies macht einen Neustart erforderlich. 

Führen Sie die folgenden Befehle in Ihrer PowerShell-Remotesitzung aus.

Zunächst installieren Sie das PowerShell-Modul von OneGet.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Als Nächstes verwenden Sie OneGet, um die neueste Version von Docker zu installieren.

```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Wenn die Installation abgeschlossen ist, starten Sie den Computer neu.

```none
Restart-Computer -Force
```

## Installieren von Basiscontainerimages

Basisimages des Betriebssystems dienen als Basis aller Windows Server- oder Hyper-V-Container. Basisimages stehen sowohl für Windows Server Core als auch für Nano Server als zugrunde liegendes Betriebssystem bereit und können mithilfe von `docker pull` installiert werden. Ausführliche Informationen zu Docker-Containerimages finden Sie unter [Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/) (Erstellen Ihrer eigenen Images auf docker.com).

Führen Sie die folgenden Befehle aus, um das Basisimage für Windows Server und Nano Server herunterzuladen und zu installieren.

```none
docker pull microsoft/nanoserver
```

```none
docker pull microsoft/windowsservercore
```

> Bitte lesen Sie sich die [Lizenzbedingungen](../Images_EULA.md) zum Betriebssystemimage für Windows-Container durch.

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
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Extrahieren Sie das komprimierte Paket.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
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



<!--HONumber=Oct16_HO2-->


