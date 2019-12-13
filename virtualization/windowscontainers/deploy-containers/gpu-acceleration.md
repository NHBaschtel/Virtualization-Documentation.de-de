---
title: GPU-Beschleunigung in Windows-Containern
description: Die Ebene der GPU-Beschleunigung in Windows-Containern
keywords: docker, Container, Geräte, Hardware
author: cwilhit
ms.openlocfilehash: 8f63c74d7839385e21206188263b9e5d08e7eb60
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909910"
---
# <a name="gpu-acceleration-in-windows-containers"></a>GPU-Beschleunigung in Windows-Containern

Für viele containerisierte Workloads bieten CPU-Compute-Ressourcen eine ausreichende Leistung. Bei einer bestimmten Arbeitsauslastung kann die von GPUs (Graphics Processing Units) angebotene hochgradig parallele computeleistung jedoch den Betrieb nach Größenordnungen beschleunigen, wodurch Kosten gesenkt und der Durchsatz enorm verbessert wird.

GPUs sind bereits ein gängiges Tool für viele beliebte Workloads, von herkömmlichem Rendering und Simulation bis hin zu Schulungs-und Inferenz von Machine Learning. Windows-Container unterstützen die GPU-Beschleunigung für DirectX und alle darauf erstellten Frameworks.

> [!NOTE]
> Diese Funktion ist in docker Desktop, Version 2,1, und der Docker-Engine-Enterprise, Version 19,03 oder höher, verfügbar.

## <a name="requirements"></a>Anforderungen

Damit dieses Feature funktioniert, muss Ihre Umgebung die folgenden Anforderungen erfüllen:

- Auf dem Container Host muss Windows Server 2019 oder Windows 10, Version 1809 oder höher, ausgeführt werden.
- Das Containerbasis Image muss [MCR.Microsoft.com/Windows:1809](https://hub.docker.com/_/microsoft-windows) oder neuer sein. Windows Server Core-und Nano Server-Container Images werden zurzeit nicht unterstützt.
- Auf dem Container Host muss die Docker-Engine 19,03 oder höher ausgeführt werden.
- Der Container Host muss über eine GPU verfügen, auf der die Anzeigetreiber Version WDDM 2,5 oder höher ausgeführt wird.

Um die WDDM-Version der Anzeigetreiber zu überprüfen, führen Sie das DirectX-Diagnose Tool (dxdiag. exe) auf dem Container Host aus. Sehen Sie sich auf der Registerkarte "Display" des Tools den Abschnitt "Drivers" (Treiber) an, wie unten angegeben.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>Ausführen eines Containers mit GPU-Beschleunigung

Führen Sie den folgenden Befehl aus, um einen Container mit GPU-Beschleunigung zu starten:

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (und alle darauf erstellten Frameworks) sind die einzigen APIs, die heute mit einer GPU beschleunigt werden können. Frameworks von Drittanbietern werden nicht unterstützt.

## <a name="hyper-v-isolated-windows-container-support"></a>Unterstützung von Hyper-V-isolierten Windows-Containern

Die GPU-Beschleunigung für Workloads in Hyper-V-isolierten Windows-Containern wird derzeit nicht unterstützt.

## <a name="hyper-v-isolated-linux-container-support"></a>Unterstützung von Hyper-V-isolierten Linux-Containern

Die GPU-Beschleunigung für Workloads in Hyper-V-isolierten Linux-Containern wird derzeit nicht unterstützt.

## <a name="more-information"></a>Weitere Informationen

Ein umfassendes Beispiel für eine in Containern enthaltene DirectX-APP, die GPU-Beschleunigung nutzt, finden Sie unter [Beispiel für DirectX-Container](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/master/windows-container-samples/directx).
