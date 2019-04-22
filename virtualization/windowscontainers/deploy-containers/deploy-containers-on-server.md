---
title: Bereitstellen von Windows-Containern unter Windows Server
description: Bereitstellen von Windows-Containern unter Windows Server
keywords: Docker, Container
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
ms.openlocfilehash: 35f35b490ce5aa80068578d78a6427ace7352b73
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380164"
---
# <a name="container-host-deployment-windows-server"></a>Containerhostbereitstellung: WindowsServer

Für die Bereitstellung eines Windows-Containerhosts sind je nach Betriebssystem und Typ des Hostsystems (physisch oder virtuell) unterschiedliche Schritte erforderlich. In diesem Dokument wird erläutert, wie Sie einen Windows-Containerhost entweder für Windows Server2016 oder für Windows Server Core2016 auf einem physischen oder virtuellen System bereitstellen.

## <a name="install-docker"></a>Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client.

Um Docker zu installieren, verwenden wir das [PowerShell-Modul von OneGet](https://github.com/OneGet/MicrosoftDockerProvider). Der Anbieter aktiviert die Containerfunktion auf Ihrem Computer und Installieren von Docker, die ein Neustart erforderlich ist.

Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und führen Sie die folgenden Cmdlets.

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

Es gibt derzeit zwei Kanäle für Docker EE für Windows Server verfügbar:

* `17.06` – Verwenden Sie diese Version aus, wenn Sie Docker Enterprise Edition (Docker-Modul, UCP, DTR) verwenden. `17.06` ist die Standardeinstellung.
* `18.03` – Verwenden Sie diese Version aus, wenn Sie Docker EE-Modul allein ausführen.

Um eine bestimmte Version zu installieren, verwenden die `RequiredVersion` Flag:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

Installieren von Docker EE Versionen erfordern ein Update zu zuvor installierten DockerMsftProvider-Modulen. So aktualisieren Sie:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Aktualisieren von Docker

Wenn Sie Docker EE-Modul aus einer früheren Kanal zu einem späteren Kanal aktualisieren müssen, verwenden Sie sowohl die `-Update` und `-RequiredVersion` Flags:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>Installieren von basiscontainerimages

Vor der Arbeit mit Windows-Containern muss ein Basisimage installiert werden. Basisimages sind mit Windows Server Core oder Nano Server als Containerbetriebssystem verfügbar. Ausführliche Informationen zu Docker-Containerimages finden Sie unter [Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/) (Erstellen Sie eigene Images auf docker.com).

Mit der Veröffentlichung von Windows Server 2019 sind Microsoft bezogen containerimages mit einer neuen Registrierung aufgerufen, der Microsoft-Container-Registrierung verschoben. Container-Images, die von Microsoft veröffentlichten sollten weiterhin über Docker Hub erkannt werden. Für neue Container-Images mit Windows Server 2019 und darüber hinaus, Sie veröffentlicht sieht sie aus der MCR nach innen bewegen. Für ältere containerimages vor Windows Server 2019 veröffentlicht sollten Sie weiterhin pull von Docker Registrierung.

### <a name="windows-server-2019-and-newer"></a>WindowsServer 2019 und höher

So installieren Sie das Basisimage "Windows Server Core", führen Sie Folgendes aus:

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

So installieren Sie das Basisimage 'Nano Server', führen Sie Folgendes aus:

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

> Bitte lesen Sie die Windows-Container, die OS EULA, image, die hier – [EULA](../images-eula.md)gefunden werden kann.

## <a name="hyper-v-isolation-host"></a>Hyper-V-Isolierung host

Sie müssen die Hyper-V-Rolle zum Ausführen von Hyper-V-Isolierung verfügen. Wenn der Windows-Containerhost selbst ein virtueller Hyper-V-Computer ist, muss die geschachtelte Virtualisierung aktiviert werden, bevor die Hyper-V-Rolle installiert werden kann. Weitere Informationen dazu finden Sie unter [Geschachtelte Virtualisierung](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

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

Um das Hyper-V-Feature mithilfe von PowerShell zu aktivieren, führen Sie das folgende Cmdlet in einer PowerShell-Sitzung mit erhöhten Rechten aus.

```PowerShell
Install-WindowsFeature hyper-v
```
