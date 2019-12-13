---
title: Geräte in Containern unter Windows
description: Welche Geräte Unterstützung ist für Container unter Windows vorhanden?
keywords: docker, Container, Geräte, Hardware
author: cwilhit
ms.openlocfilehash: 1ad63c158a42f116882c949b242274dde8d893fc
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910600"
---
# <a name="devices-in-containers-on-windows"></a>Geräte in Containern unter Windows

Standardmäßig erhalten Windows-Containern minimalen Zugriff auf Host Geräte, genau wie Linux-Container. Es gibt bestimmte Arbeits Auslastungen, bei denen es von Vorteil oder sogar Imperativ ist, auf Host Hardware Geräte zuzugreifen und diese zu kommunizieren. In dieser Anleitung wird erläutert, welche Geräte in Containern unterstützt werden und wie Sie loslegen.

## <a name="requirements"></a>Anforderungen

Damit dieses Feature funktioniert, muss Ihre Umgebung die folgenden Anforderungen erfüllen:
- Auf dem Container Host muss Windows Server 2019 oder Windows 10, Version 1809 oder höher, ausgeführt werden.
- Ihre Containerbasis Image Version muss 1809 oder höher sein.
- Die Container müssen Windows-Container sein, die im Prozess isolierten Modus ausgeführt werden.
- Auf dem Container Host muss die Docker-Engine 19,03 oder höher ausgeführt werden.

## <a name="run-a-container-with-a-device"></a>Ausführen eines Containers mit einem Gerät

Verwenden Sie den folgenden Befehl, um einen Container mit einem Gerät zu starten:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Sie müssen den `{interface class guid}` durch eine entsprechende [GUID für die Geräteschnittstellen Klasse](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes)ersetzen, die im folgenden Abschnitt zu finden ist.

Um einen Container mit mehreren Geräten zu starten, verwenden Sie den folgenden Befehl, und verbinden Sie mehrere `--device` Argumente:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

In Windows deklarieren alle Geräte eine Liste von Schnittstellen Klassen, die Sie implementieren. Durch die Übergabe dieses Befehls an docker wird sichergestellt, dass alle Geräte, die als Implementierung der angeforderten Klasse identifiziert werden, in den Container übertragen werden.

Dies bedeutet, dass Sie das Gerät **nicht** vom Host zuweisen. Stattdessen wird Sie vom Host für den Container freigegeben. Ebenso, weil Sie eine Klassen-GUID angeben, werden _alle_ Geräte, die diese GUID implementieren, für den Container freigegeben.

## <a name="what-devices-are-supported"></a>Welche Geräte werden unterstützt?

Die folgenden Geräte (und ihre Geräteschnittstellen Klassen-GUIDs) werden heute unterstützt:
  
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
<td><center>916ef1cb-8426-468d-a6f 7-9ae8076881b3</center></td>
</tr>
<tr valign="top">
<td><center>I2C-Bus</center></td>
<td><center>A11EE3C6-8421-4202-A3E7-B91FF90188E4</center></td>
</tr>
<tr valign="top">
<td><center>COM-Port</center></td>
<td><center>86e0d1e0-8089-11D0-9ce4-08003e301f 73</center></td>
</tr>
<tr valign="top">
<td><center>SPI-Bus</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>DirectX-GPU-Beschleunigung</center></td>
<td><center>Siehe <a href="https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/gpu-acceleration">GPU Acceleration</a> docs</center></td>
</tr>
</tbody>
</table>

> [!IMPORTANT]
> Geräte Unterstützung ist Treiber abhängig. Der Versuch, in der obigen Tabelle nicht definierte Klassen-GUIDs zu übergeben, kann zu undefiniertem Verhalten führen.

## <a name="hyper-v-isolated-windows-container-support"></a>Unterstützung von Hyper-V-isolierten Windows-Containern

Geräte Zuweisung und Geräte Freigabe für Workloads in Hyper-V-isolierten Windows-Containern wird derzeit nicht unterstützt.

## <a name="hyper-v-isolated-linux-container-support"></a>Unterstützung von Hyper-V-isolierten Linux-Containern

Geräte Zuweisung und Geräte Freigabe für Workloads in Hyper-V-isolierten Linux-Containern wird derzeit nicht unterstützt.
