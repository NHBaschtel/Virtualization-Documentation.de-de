---
title: GPU-Beschleunigung in Windows-Containern
description: Welche Ebene der GPU-Beschleunigung in Windows-Container vorhanden ist
keywords: Docker, Container, Geräte, hardware
author: cwilhit
ms.openlocfilehash: 066f97b859b133a03e24df5db95cafe405ea3110
ms.sourcegitcommit: 2b456022ee666863ef53082580ac1d432de86939
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 05/16/2019
ms.locfileid: "9657368"
---
# <a name="gpu-acceleration-in-windows-containers"></a>GPU-Beschleunigung in Windows-Containern

Für viele in Containern Workloads bieten CPU-Compute-Ressourcen über ausreichende Leistung. Allerdings kann für eine bestimmte Klasse Workload, die parallele Compute-Leistungsfähigkeit von GPUs (Graphics processing Unit) angebotenen Vorgänge um Größenordnungen, Deaktivierung des Kosten und Verbesserung der Durchsatz äußerst beschleunigen.

GPUs sind bereits ein allgemeines Tool für gängige Workloads, von herkömmlichen Rendering- und Simulation Machine Learning-Schulung und Rückschluss. Windows-Container unterstützen die GPU-Beschleunigung für DirectX und alle Frameworks, die es imagebasierter.

> [!IMPORTANT]
> Dieses Feature erfordert eine Version von Docker, die unterstützt die `--device` Befehlszeilenoption für Windows-Container. Diese Unterstützung steht derzeit nur in der `Docker Desktop for Windows Edge` freizugeben. Sie können die Edge-Version von Docker herunterladen [hier](https://docs.docker.com/docker-for-windows/edge-release-notes/).

## <a name="requirements"></a>Anforderungen

Für dieses Feature funktioniert muss Ihre Umgebung die folgenden Anforderungen erfüllen:

- Der Container-Host muss Windows Server 2019 oder Windows 10, Version 1809 oder höher ausgeführt werden.
- Der Container-Basis-Image darf [mcr.microsoft.com/windows:1809](https://hub.docker.com/_/microsoft-windowsfamily-windows) oder höher. Windows Server Core und Nano Server-Container-Images werden derzeit nicht unterstützt.
- Der Container-Host muss Docker-Modul 19.03 oder höher ausgeführt werden.
- Der Container-Host muss es sich um eine GPU ausgeführten Anzeige Treiber-Version WDDM 2.5 oder höher sein.

Um die Version WDDM Anzeige Treiber zu suchen, führen Sie das DirectX-Diagnosetool (dxdiag.exe) auf dem Hostcontainer. Suchen Sie in der Registerkarte "Anzeige" das Tool im Abschnitt "Drivers", wie unten angegeben.

![DxDiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>Führen Sie einen Container mit GPU-Beschleunigung

Um einen Container mit GPU-Beschleunigung zu starten, führen Sie den folgenden Befehl aus:

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (und alle Frameworks, die es imagebasierter) sind die einzigen APIs, die beschleunigt werden können mit einer GPU heute. 3. Party-Frameworks werden nicht unterstützt.

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-V-isolierten Windows-Container-Unterstützung

GPU-Beschleunigung für Workloads in Hyper-V-isolierten Windows-Containern ist derzeit nicht unterstützt.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V-isolierten Linux-Container-Unterstützung

GPU-Beschleunigung für Workloads in Hyper-V-isolierten Linux-Container ist derzeit nicht unterstützt.
