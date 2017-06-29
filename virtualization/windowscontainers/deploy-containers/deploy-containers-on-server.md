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
ms.openlocfilehash: b01c1112ed119908bdabfa0eeee16f4ba11a47f6
ms.sourcegitcommit: bb171f4a858fefe33dd0748b500a018fd0382ea6
ms.translationtype: HT
ms.contentlocale: de-DE
---
# <a name="container-host-deployment---windows-server"></a>Containerhostbereitstellung: Windows Server

Für die Bereitstellung eines Windows-Containerhosts sind je nach Betriebssystem und Typ des Hostsystems (physisch oder virtuell) unterschiedliche Schritte erforderlich. In diesem Dokument wird erläutert, wie Sie einen Windows-Containerhost entweder für Windows Server2016 oder für Windows Server Core2016 auf einem physischen oder virtuellen System bereitstellen.

## <a name="install-docker"></a>Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. 

Verwenden Sie das [PowerShell-Modul von OneGet](https://github.com/OneGet/MicrosoftDockerProvider), um Docker zu installieren. Der Anbieter aktiviert die Containerfunktion auf Ihrem Computer und installiert Docker. Dies macht einen Neustart erforderlich. 

Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und führen Sie die folgenden Befehle aus.

Installieren Sie das PowerShell-Modul „OneGet“.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Verwenden Sie OneGet, um die neueste Version von Docker zu installieren.

```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Wenn die Installation abgeschlossen ist, starten Sie den Computer neu.

```none
Restart-Computer -Force
```

## <a name="install-base-container-images"></a>Installieren von Basiscontainerimages

Vor der Arbeit mit Windows-Containern muss ein Basisimage installiert werden. Basisimages sind mit Windows Server Core oder Nano Server als Containerbetriebssystem verfügbar. Ausführliche Informationen zu Docker-Containerimages finden Sie unter [Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/) (Erstellen Sie eigene Images auf docker.com).

Zum Installieren des Basisimages für Windows Server Core führen Sie folgenden Befehl aus:

```none
docker pull microsoft/windowsservercore
```

Zum Installieren des Basisimages für Nano Server führen Sie folgenden Befehl aus:

```none
docker pull microsoft/nanoserver
```

> Bitte lesen Sie die [Lizenzbedingungen](../images-eula.md) zum Betriebssystemimage für Windows-Container.

## <a name="hyper-v-container-host"></a>Hyper-V-Containerhost

Um Hyper-V-Container auszuführen, ist die Hyper-V-Rolle erforderlich. Wenn der Windows-Containerhost selbst ein virtueller Hyper-V-Computer ist, muss die geschachtelte Virtualisierung aktiviert werden, bevor die Hyper-V-Rolle installiert werden kann. Weitere Informationen dazu finden Sie unter [Geschachtelte Virtualisierung]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

### <a name="nested-virtualization"></a>Geschachtelte Virtualisierung

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

### <a name="enable-the-hyper-v-role"></a>Aktivieren der Hyper-V-Rolle

Um das Hyper-V-Feature mithilfe von PowerShell zu installieren, führen Sie den folgenden Befehl in einem PowerShell-Sitzung mit erhöhten Rechten aus.

```none
Install-WindowsFeature hyper-v
```
