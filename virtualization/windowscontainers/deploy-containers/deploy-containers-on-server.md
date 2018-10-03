---
title: Bereitstellen von Windows-Containern unter Windows Server
description: Bereitstellen von Windows-Containern unter Windows Server
keywords: Docker, Container
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 701112cac9c3f6d647fe5fb70309350fd0d07161
ms.sourcegitcommit: d69ed13d505e96f514f456cdae0f93dab4fd3746
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/03/2018
ms.locfileid: "4340848"
---
# <a name="container-host-deployment---windows-server"></a>Containerhostbereitstellung: Windows Server

Für die Bereitstellung eines Windows-Containerhosts sind je nach Betriebssystem und Typ des Hostsystems (physisch oder virtuell) unterschiedliche Schritte erforderlich. In diesem Dokument wird erläutert, wie Sie einen Windows-Containerhost entweder für Windows Server2016 oder für Windows Server Core2016 auf einem physischen oder virtuellen System bereitstellen.

## <a name="install-docker"></a>Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. 

Verwenden Sie das [PowerShell-Modul von OneGet](https://github.com/OneGet/MicrosoftDockerProvider), um Docker zu installieren. Der Anbieter aktiviert die Containerfunktion auf Ihrem Computer und installiert Docker. Dies macht einen Neustart erforderlich. 

Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und führen Sie die folgenden Befehle aus.

Installieren Sie das PowerShell-Modul „OneGet“.

```PowerShell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Verwenden Sie OneGet, um die neueste Version von Docker zu installieren.

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Wenn die Installation abgeschlossen ist, starten Sie den Computer neu.

```PowerShell
Restart-Computer -Force
```

## <a name="install-a-specific-version-of-docker"></a>Installieren Sie eine bestimmte Version von Docker

Derzeit gibt es zwei Kanäle für Docker EE für Windows Server verfügbar:

* `17.06` -Verwenden Sie diese Version aus, wenn Sie Docker Enterprise Edition (Docker-Modul, UCP, DTR) verwenden. `17.06` ist die Standardeinstellung.
* `18.03` -Verwenden Sie diese Version aus, wenn Sie Docker EE-Modul allein ausführen.

Um eine bestimmte Version zu installieren, verwenden die `RequiredVersion` Flag:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

Installieren von Docker EE Versionen erfordern ein Update für zuvor installierten DockerMsftProvider Module. So aktualisieren Sie:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Aktualisieren von Docker

Wenn Sie Docker EE-Modul von einer früheren Kanal zu einem späteren Kanal aktualisieren müssen, verwenden Sie sowohl die `-Update` und `-RequiredVersion` Flags:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>Installieren von Basiscontainerimages

Vor der Arbeit mit Windows-Containern muss ein Basisimage installiert werden. Basisimages sind mit Windows Server Core oder Nano Server als Containerbetriebssystem verfügbar. Ausführliche Informationen zu Docker-Containerimages finden Sie unter [Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/) (Erstellen Sie eigene Images auf docker.com).

Mit der Veröffentlichung von Windows Server 2019 sind Microsoft bezogen containerimages mit einer neuen Registrierung aufgerufen, der Microsoft-Container-Registrierung verschoben. Container-Images, die von Microsoft veröffentlichten sollten weiterhin über Docker Hub erkannt werden. Für neue Container-Images mit Windows Server 2019 und darüber hinaus Sie veröffentlicht sieht sie von den MCR abrufen. Für ältere containerimages vor Windows Server 2019 veröffentlicht sollten Sie weiterhin pull von Docker Registrierung.

### <a name="windows-server-2019-and-newer"></a>WindowsServer 2019 und höher

So installieren Sie das "Windows Server Core"-Basisimage, führen Sie Folgendes aus:

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

So installieren Sie das Basisimage "Nano Server", führen Sie Folgendes aus:

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (Versionen 1607 1803)

Zum Installieren des Basisimages für Windows Server Core führen Sie folgenden Befehl aus:

```PowerShell
docker pull microsoft/windowsservercore
```

Zum Installieren des Basisimages für Nano Server führen Sie folgenden Befehl aus:

```PowerShell
docker pull microsoft/nanoserver
```

> Bitte lesen Sie die [Lizenzbedingungen](../images-eula.md) zum Betriebssystemimage für Windows-Container.

## <a name="hyper-v-container-host"></a>Hyper-V-Containerhost

Um Hyper-V-Container auszuführen, ist die Hyper-V-Rolle erforderlich. Wenn der Windows-Containerhost selbst ein virtueller Hyper-V-Computer ist, muss die geschachtelte Virtualisierung aktiviert werden, bevor die Hyper-V-Rolle installiert werden kann. Weitere Informationen dazu finden Sie unter [Geschachtelte Virtualisierung]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

### <a name="nested-virtualization"></a>Geschachtelte Virtualisierung

Das folgende Skript konfiguriert die geschachtelte Virtualisierung für den Containerhost. Dieses Skript wird auf dem übergeordneten Hyper-V-Computer ausgeführt. Stellen Sie sicher, dass der virtuelle Containerhostcomputer ausgeschaltet ist, wenn dieses Skript ausgeführt wird.

```PowerShell
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory -VMName $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### <a name="enable-the-hyper-v-role"></a>Aktivieren der Hyper-V-Rolle

Um das Hyper-V-Feature mithilfe von PowerShell zu installieren, führen Sie den folgenden Befehl in einem PowerShell-Sitzung mit erhöhten Rechten aus.

```PowerShell
Install-WindowsFeature hyper-v
```
