---
title: GPU-Beschleunigung in Windows-Containern
description: Welche Ebene der GPU-Beschleunigung in Windows-Containern vorhanden ist
keywords: docker, Container, Geräte, Hardware
author: cwilhit
ms.openlocfilehash: 6e5010efee10f9b488cbeb57b14bc86f30c1e766
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883273"
---
# <a name="gpu-acceleration-in-windows-containers"></a>GPU-Beschleunigung in Windows-Containern

Für viele Container-Workloads bieten CPU-Rechenressourcen eine ausreichende Leistung. Bei einer bestimmten Arbeitsauslastung können jedoch die von GPUs (Grafik Verarbeitungseinheiten) bereitgestellten massiv parallelen Rechenleistung den Betrieb nach Größenordnungen beschleunigen, die Kosten senken und den Durchsatz enorm verbessern.

GPUs sind bereits ein gängiges Tool für viele gängige Arbeitsauslastungen, von herkömmlicher Darstellung und Simulation bis hin zu maschineller Schulung und Rückschluss. Windows-Container unterstützen die GPU-Beschleunigung für DirectX und alle oben darauf integrierten Frameworks.

## <a name="requirements"></a>Anforderungen

Damit dieses Feature funktioniert, muss Ihre Umgebung die folgenden Voraussetzungen erfüllen:

- Auf dem Container Host muss Windows Server 2019 oder Windows 10, Version 1809 oder höher, ausgeführt werden.
- Das Containerbasis Bild muss [MCR.Microsoft.com/Windows:1809](https://hub.docker.com/_/microsoft-windowsfamily-windows) oder neuer sein. Windows Server Core-und Nano Server-Container Bilder werden zurzeit nicht unterstützt.
- Auf dem Container Host muss das docker Modul 19,03 oder höher ausgeführt werden.
- Der Container-Host muss über eine GPU mit den Anzeigetreibern Version WDDM 2,5 oder höher verfügen.

Führen Sie zum Überprüfen der WDDM-Version Ihrer Bildschirmtreiber das DirectX-Diagnose Tool (dxdiag. exe) auf dem Container Host aus. Schauen Sie im Reiter "Display" des Tools im Abschnitt "Treiber" nach, wie unten angegeben.

![Dxdiag](media/dxdiag.png)

## <a name="run-a-container-with-gpu-acceleration"></a>Ausführen eines Containers mit GPU-Beschleunigung

Führen Sie den folgenden Befehl aus, um einen Container mit GPU-Beschleunigung zu starten:

```shell
docker run --isolation process --device class/5B45201D-F2F2-4F3B-85BB-30FF1F953599 mcr.microsoft.com/windows:1809
```

> [!IMPORTANT]
> DirectX (und alle oben darauf integrierten Frameworks) sind die einzigen APIs, die heute mit einer GPU beschleunigt werden können. Frameworks von Drittanbietern werden nicht unterstützt.

## <a name="hyper-v-isolated-windows-container-support"></a>Unterstützung für Hyper-V-isolierte Windows-Container

Die GPU-Beschleunigung für Arbeitslasten in Hyper-V-isolierten Windows-Containern wird heute nicht unterstützt.

## <a name="hyper-v-isolated-linux-container-support"></a>Unterstützung für Hyper-V-isolierte Linux-Container

Die GPU-Beschleunigung für Arbeitslasten in Hyper-V-isolierten Linux-Containern wird heute nicht unterstützt.