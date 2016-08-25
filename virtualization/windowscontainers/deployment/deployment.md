---
title: Bereitstellen von Windows-Containern unter Windows Server
description: Bereitstellen von Windows-Containern unter Windows Server
keywords: Docker, Container
author: neilpeterson
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
translationtype: Human Translation
ms.sourcegitcommit: 2319649d1dd39677e59a9431fbefaf82982492c6
ms.openlocfilehash: b60329a09ea0f119446fa2aa20de68e3edc2b245

---

# Containerhostbereitstellung: Windows Server

**Dieser Inhalt ist vorläufig und kann geändert werden.**

Für die Bereitstellung eines Windows-Containerhosts sind je nach Betriebssystem und Typ des Hostsystems (physisch oder virtuell) unterschiedliche Schritte erforderlich. In diesem Dokument wird erläutert, wie Sie einen Windows-Containerhost entweder für Windows Server 2016 oder für Windows Server Core 2016 auf einem physischen oder virtuellen System bereitstellen.

## Azure-Image 

Ein vollständig konfiguriertes Windows Server-Image ist in Azure verfügbar. Um dieses Image zu verwenden, stellen Sie durch Klicken auf die Schaltfläche unten eine virtuelle Maschine bereit. Wenn Sie ein Windows-Containersystem mithilfe dieser Vorlage in Azure bereitstellen, kann der Rest dieses Dokuments übersprungen werden.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## Installieren des Containerfeatures

Das Containerfeature muss aktiviert werden, bevor Sie mit Windows-Containern arbeiten können. Führen Sie dazu den folgenden Befehl in einer PowerShell-Sitzung mit erhöhten Rechten aus.

```none
Install-WindowsFeature containers
```

Starten Sie den Computer neu, wenn die Installation des Features abgeschlossen ist.

```none
Restart-Computer -Force
```

## Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Für diese Übung werden beide installiert.

Laden Sie das Docker-Modul und den Docker-Client als ZIP-Archiv herunter.

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

Erweitern Sie das ZIP-Archiv in „Programme“, die Archivinhalte befinden sich bereits im Docker-Verzeichnis.

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Führen Sie die folgenden beiden Befehle aus, um das Docker-Verzeichnis zum Systempfad hinzuzufügen.

```none
# for quick use, does not require shell to be restarted
$env:path += ";c:\program files\docker"

# for persistent use, will apply even after a reboot 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Starten Sie die PowerShell-Sitzung neu, damit der geänderte Pfad erkannt wird.

Führen Sie den folgenden Befehl aus, um Docker als Windows-Dienst zu installieren.

```none
dockerd --register-service
```

Nach Abschluss der Installation kann der Dienst gestartet werden.

```none
Start-Service Docker
```

## Installieren von Basiscontainerimages

Vor der Arbeit mit Windows-Containern muss ein Basisimage installiert werden. Basisimages sind mit Windows Server Core oder Nano Server als zugrunde liegendem Betriebssystem verfügbar. Ausführliche Informationen zu Windows-Containerimages finden Sie unter [Verwalten von Containerimages](../management/manage_images.md).

Zum Installieren des Basisimages für Windows Server Core führen Sie folgenden Befehl aus:

```none
docker pull microsoft/windowsservercore
```

Zum Installieren des Basisimages für Nano Server führen Sie folgenden Befehl aus:

```none
docker pull microsoft/nanoserver
```

## Hyper-V-Containerhost

Um Hyper-V-Container auszuführen, ist die Hyper-V-Rolle erforderlich. Wenn der Windows-Containerhost selbst ein virtueller Hyper-V-Computer ist, muss die geschachtelte Virtualisierung aktiviert werden, bevor die Hyper-V-Rolle installiert werden kann. Weitere Informationen dazu finden Sie unter [Geschachtelte Virtualisierung]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

### Geschachtelte Virtualisierung

Das folgende Skript konfiguriert die geschachtelte Virtualisierung für den Containerhost. Dieses Skript wird auf dem übergeordneten Hyper-V-Computer ausgeführt. Stellen Sie sicher, dass der virtuelle Containerhostcomputer ausgeschaltet ist, wenn dieses Skript ausgeführt wird.

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

Um das Hyper-V-Feature mithilfe von PowerShell zu installieren, führen Sie den folgenden Befehl in einem PowerShell-Sitzung mit erhöhten Rechten aus.

```none
Install-WindowsFeature hyper-v
```



<!--HONumber=Aug16_HO4-->


