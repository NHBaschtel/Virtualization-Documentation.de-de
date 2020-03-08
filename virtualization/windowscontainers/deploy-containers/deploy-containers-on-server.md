---
title: Bereitstellen von Windows-Containern unter Windows Server
description: Bereitstellen von Windows-Containern unter Windows Server
keywords: Docker, Container
author: taylorb-microsoft
ms.date: 09/09/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 9899a2d76bfa1fe312e3bd983f60d09d77c272e9
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853911"
---
# <a name="container-host-deployment-windows-server"></a>Container Host Bereitstellung: Windows Server

Für die Bereitstellung eines Windows-Containerhosts sind je nach Betriebssystem und Typ des Hostsystems (physisch oder virtuell) unterschiedliche Schritte erforderlich. In diesem Dokument wird erläutert, wie Sie einen Windows-Containerhost entweder für Windows Server 2016 oder für Windows Server Core 2016 auf einem physischen oder virtuellen System bereitstellen.

## <a name="install-docker"></a>Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus der Docker-Engine und dem docker-Client.

Zum Installieren von Docker verwenden wir das [PowerShell-Modul oneget-Anbieter](https://github.com/OneGet/MicrosoftDockerProvider). Der Anbieter aktiviert die containerfunktion auf dem Computer und installiert Docker, was einen Neustart erfordert.

Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten und führen Sie die folgenden Cmdlets aus.

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

## <a name="install-a-specific-version-of-docker"></a>Installieren Sie eine bestimmte Version von Docker.

Zurzeit sind zwei Kanäle für docker EE für Windows Server verfügbar:

* `17.06`: Verwenden Sie diese Version, wenn Sie die Docker Enterprise Edition (docker-Engine, UCP, DTR) verwenden. `17.06` ist die Standardeinstellung.
* `18.03`: Verwenden Sie diese Version, wenn Sie die Docker EE-Engine allein ausführen.

Verwenden Sie das `RequiredVersion`-Flag, um eine bestimmte Version zu installieren:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

Für die Installation bestimmter docker EE-Versionen ist möglicherweise ein Update der zuvor installierten dockermsftprovider-Module erforderlich. So aktualisieren Sie Folgendes:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Aktualisieren von Docker

Wenn Sie die Docker-EE-Engine von einem früheren Kanal auf einen späteren Kanal aktualisieren müssen, verwenden Sie die `-Update`-und `-RequiredVersion`-Flags:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>Installieren von Basis Container Images

Vor der Arbeit mit Windows-Containern muss ein Basisimage installiert werden. Basisimages sind mit Windows Server Core oder Nano Server als Containerbetriebssystem verfügbar. Ausführliche Informationen zu Docker-Containerimages finden Sie unter [Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/) (Erstellen Sie eigene Images auf docker.com).

> [!TIP]
> Mit der Auswirkung von Mai 2018 und der Bereitstellung eines konsistenten und vertrauenswürdigen Erwerbs Erlebnisses werden fast alle Container Images von Microsoft aus dem Microsoft-Container Registry _MCR.Microsoft.com_, während gleichzeitig der aktuelle Ermittlungsprozess über [_docker Hub_](https://hub.docker.com/publishers/microsoftowner)verwaltet wird.

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 und höher

Führen Sie Folgendes aus, um das Basis Image "Windows Server Core" zu installieren:

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

Führen Sie Folgendes aus, um das Basis Image "Nano Server" zu installieren:

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (Version 1607-1803)

Zum Installieren des Basisimages für Windows Server Core führen Sie folgenden Befehl aus:

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:1607
```

Zum Installieren des Basisimages für Nano Server führen Sie folgenden Befehl aus:

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1803
```

> Lesen Sie die Windows-Container-EULA für Betriebssystem Images, die Sie hier finden – [EULA](../images-eula.md).

## <a name="hyper-v-isolation-host"></a>Hyper-V-Isolations Host

Sie müssen über die Hyper-v-Rolle verfügen, um die Hyper-v-Isolation ausführen zu können. Wenn der Windows-Containerhost selbst ein virtueller Hyper-V-Computer ist, muss die geschachtelte Virtualisierung aktiviert werden, bevor die Hyper-V-Rolle installiert werden kann. Weitere Informationen dazu finden Sie unter [Geschachtelte Virtualisierung](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization).

### <a name="nested-virtualization"></a>Netzwerkvirtualisierung

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

Um das Hyper-V-Feature mithilfe von PowerShell zu aktivieren, führen Sie das folgende Cmdlet in einer PowerShell-Sitzung mit erhöhten Rechten aus.

```PowerShell
Install-WindowsFeature hyper-v
```
