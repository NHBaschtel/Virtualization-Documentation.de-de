---
title: Geräte in Containern unter Windows
description: Welche Geräteunterstützung ist für Container unter Windows vorhanden?
keywords: docker, Container, Geräte, Hardware
author: cwilhit
ms.openlocfilehash: ee9c5da5ef87dceb3374977670da2ea50ea87382
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883163"
---
# <a name="devices-in-containers-on-windows"></a>Geräte in Containern unter Windows

Standardmäßig werden Windows-Containern minimaler Zugriff auf Hostgeräte gewährt – genau wie Linux-Container. Es gibt bestimmte Arbeitsauslastungen, bei denen es vorteilhaft ist – oder sogar zwingend –, auf Host-Hardwaregeräten zuzugreifen und mit Ihnen zu kommunizieren. In diesem Leitfaden wird erläutert, welche Geräte in Containern unterstützt werden und wie Sie beginnen können.

## <a name="requirements"></a>Anforderungen

Damit dieses Feature funktioniert, muss Ihre Umgebung die folgenden Voraussetzungen erfüllen:
- Auf dem Container Host muss Windows Server 2019 oder Windows 10, Version 1809 oder höher, ausgeführt werden.
- Die Version Ihres Container-Basis Bilds muss 1809 oder höher sein.
- Ihre Container müssen Windows-Container sein, die im Prozess isolierten Modus ausgeführt werden.
- Auf dem Container Host muss das docker Modul 19,03 oder höher ausgeführt werden.

## <a name="run-a-container-with-a-device"></a>Ausführen eines Containers mit einem Gerät

Verwenden Sie den folgenden Befehl, um einen Container mit einem Gerät zu starten:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Sie müssen die `{interface class guid}` durch eine entsprechende [Device Interface-Klassen-GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)ersetzen, die im folgenden Abschnitt zu finden ist.

Wenn Sie einen Container mit mehreren Geräten starten möchten, verwenden Sie den folgenden Befehl und `--device` die Zeichenfolge zusammen mehrere Argumente:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Unter Windows deklarieren alle Geräte eine Liste der von Ihnen implementierten Schnittstellen Klassen. Durch Übergabe dieses Befehls an Andockfenster wird sichergestellt, dass alle Geräte, die die angeforderte Klasse implementieren, in den Container übertragen werden.

Das bedeutet, dass Sie das Gerät **nicht** vom Host entfernen. Stattdessen teilt der Host ihn mit dem Container. Außerdem werden _alle_ Geräte, die diese GUID implementieren, für den Container freigegeben, da Sie eine Klassen-GUID angeben.

## <a name="what-devices-are-supported"></a>Welche Geräte werden unterstützt?

Die folgenden Geräte (und deren Device Interface-Klassen-GUIDs) werden heute unterstützt:
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Gerätetyp</center></th>
<th><center>Schnittstellen Klassen-GUID</center></th>
</tr>
</thead>
<tbody>
<tr valign="top">
<td><center>GPIO</center></td>
<td><center>916EF1CB-8426-468D-A6F7-9AE8076881B3</center></td>
</tr>
<tr valign="top">
<td><center>I2C-Bus</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>COM-Port</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>SPI-Bus</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>DirectX GPU-Beschleunigung</center></td>
<td><center>Informationen zu <a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">GPU</a> -Beschleunigungs Dokumenten</center></td>
</tr>
</tbody>
</table>

> [!TIP]
> Die oben aufgeführten Geräte sind die _einzigen_ Geräte, die heute in Windows-Containern unterstützt werden. Wenn Sie versuchen, andere Klassen-GUIDs zu übergeben, kann der Container nicht gestartet werden.

## <a name="hyper-v-isolated-windows-container-support"></a>Unterstützung für Hyper-V-isolierte Windows-Container

Geräte Zuweisungen und Geräte Freigaben für Arbeitslasten in Hyper-V-isolierten Windows-Containern werden heute nicht unterstützt.

## <a name="hyper-v-isolated-linux-container-support"></a>Unterstützung für Hyper-V-isolierte Linux-Container

Geräte Zuweisungen und Geräte Freigaben für Arbeitslasten in Hyper-V-isolierten Linux-Containern werden heute nicht unterstützt.