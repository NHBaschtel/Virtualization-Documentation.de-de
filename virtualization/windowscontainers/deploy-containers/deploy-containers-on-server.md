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
ms.openlocfilehash: e045539b189eb8cd1594da0784ab0c88e848c948
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998777"
---
# <a name="container-host-deployment-windows-server"></a>Container Host Bereitstellung: Windows Server

Für die Bereitstellung eines Windows-Containerhosts sind je nach Betriebssystem und Typ des Hostsystems (physisch oder virtuell) unterschiedliche Schritte erforderlich. In diesem Dokument wird erläutert, wie Sie einen Windows-Containerhost entweder für Windows Server2016 oder für Windows Server Core2016 auf einem physischen oder virtuellen System bereitstellen.

## <a name="install-docker"></a>Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Andockfenster besteht aus dem Andock Modul und dem docker-Client.

Zum Installieren von Docker verwenden wir das [PowerShell-Modul für OneGet-Anbieter](https://github.com/OneGet/MicrosoftDockerProvider). Der Anbieter aktiviert die Container Funktion auf Ihrem Computer und installiert den Docker, für den ein Neustart erforderlich ist.

Öffnen Sie eine erhöhte PowerShell-Sitzung, und führen Sie die folgenden Cmdlets aus.

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

## <a name="install-a-specific-version-of-docker"></a>Installieren einer bestimmten Version von Docker

Für docker EE für Windows Server stehen derzeit zwei Kanäle zur Verfügung:

* `17.06` – Verwenden Sie diese Version, wenn Sie docker Enterprise Edition (Andock Modul, UCP, DTR) verwenden. `17.06` ist die Standardeinstellung.
* `18.03` – Verwenden Sie diese Version, wenn Sie nur das docker EE-Modul ausführen.

Verwenden Sie das `RequiredVersion` Flag, um eine bestimmte Version zu installieren:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Force -RequiredVersion 18.03
```

Für die Installation bestimmter docker EE-Versionen muss möglicherweise ein Update für zuvor installierte DockerMsftProvider-Module erforderlich sein. So aktualisieren Sie Folgendes:

```PowerShell
Update-Module DockerMsftProvider
```

## <a name="update-docker"></a>Docker aktualisieren

Wenn Sie das docker EE-Modul von einem früheren Kanal zu einem späteren Kanal aktualisieren müssen, verwenden Sie `-Update` die `-RequiredVersion` Flags und:

```PowerShell
Install-Package -Name docker -ProviderName DockerMsftProvider -Update -Force -RequiredVersion 18.03
```

## <a name="install-base-container-images"></a>Installieren von Basis Container Bildern

Vor der Arbeit mit Windows-Containern muss ein Basisimage installiert werden. Basisimages sind mit Windows Server Core oder Nano Server als Containerbetriebssystem verfügbar. Ausführliche Informationen zu Docker-Containerimages finden Sie unter [Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/) (Erstellen Sie eigene Images auf docker.com).

Mit der Veröffentlichung von Windows Server 2019 werden Microsoft-Container Bilder in eine neue Registrierung mit der Bezeichnung Microsoft Container Registry verschoben. Von Microsoft veröffentlichte Container Bilder sollten weiterhin über den docker-Hub erkannt werden. Für neue Container Bilder, die mit Windows Server 2019 und darüber hinaus veröffentlicht wurden, sollten Sie Sie aus dem "Sie" herausziehen. Für ältere Container Bilder, die vor Windows Server 2019 veröffentlicht wurden, sollten Sie Sie weiterhin aus der Registrierung des Dockers ziehen.

### <a name="windows-server-2019-and-newer"></a>Windows Server 2019 und höher

Führen Sie die folgenden Schritte aus, um das Basisabbild "Windows Server Core" zu installieren:

```PowerShell
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

Führen Sie die folgenden Schritte aus, um das Basisabbild "Nano Server" zu installieren:

```PowerShell
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

### <a name="windows-server-2016-versions-1607-1803"></a>Windows Server 2016 (Versionen 1607-1803)

Zum Installieren des Basisimages für Windows Server Core führen Sie folgenden Befehl aus:

```PowerShell
docker pull microsoft/windowsservercore
```

Zum Installieren des Basisimages für Nano Server führen Sie folgenden Befehl aus:

```PowerShell
docker pull microsoft/nanoserver
```

> Bitte lesen Sie den EULA für Windows Containers OS-Bild, das hier zu finden ist – [EULA](../images-eula.md).

## <a name="hyper-v-isolation-host"></a>Hyper-V-Isolierungs Host

Sie müssen über die Hyper-v-Rolle verfügen, um die Hyper-v-Isolierung ausführen zu können. Wenn der Windows-Containerhost selbst ein virtueller Hyper-V-Computer ist, muss die geschachtelte Virtualisierung aktiviert werden, bevor die Hyper-V-Rolle installiert werden kann. Weitere Informationen dazu finden Sie unter [Geschachtelte Virtualisierung](https://docs.microsoft.com/virtualization/hyper-v-on-windows/user-guide/nested-virtualization).

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

Um das Hyper-V-Feature mithilfe von PowerShell zu aktivieren, führen Sie das folgende Cmdlet in einer erhöhten PowerShell-Sitzung aus.

```PowerShell
Install-WindowsFeature hyper-v
```
