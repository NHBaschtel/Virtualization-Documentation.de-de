---
title: Geräte in Containern unter Windows
description: Welche geräteunterstützung für Container unter Windows vorhanden ist
keywords: Docker, Container, Geräte, hardware
author: cwilhit
ms.openlocfilehash: f32ba3de347bcf968088d2f3f20f22f82166d652
ms.sourcegitcommit: 34d8b2ca5eebcbdb6958560b1f4250763bee5b48
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 05/08/2019
ms.locfileid: "9621558"
---
# <a name="devices-in-containers-on-windows"></a>Geräte in Containern unter Windows

Standardmäßig sind Windows-Container minimalen Zugriff auf Hostgeräte – genau wie Linux-Container gewährt. Es gibt bestimmte Workloads, in denen es nützlich – oder sogar imperativer – zugreifen und mit Host Hardwaregeräten kommunizieren. Dieses Handbuch behandelt, welche Geräte in Containern unterstützt werden und die ersten Schritte.

> [!IMPORTANT]
> Dieses Feature erfordert eine Version von Docker, die unterstützt die `--device` Befehlszeilenoption für Windows-Container. Offizieller Docker-Support ist eingeplant, für die der bevorstehenden Version von Docker EE-Modul 19.03. Bis dahin enthält [upstream Quelle](https://master.dockerproject.org/) für Docker die erforderlichen Bits.

## <a name="requirements"></a>Anforderungen

Für dieses Feature funktioniert muss Ihre Umgebung die folgenden Anforderungen erfüllen:
- Der Container-Host muss Windows Server 2019 oder Windows 10, Version 1809 oder höher ausgeführt werden.
- Die Container-Basis-Image-Version muss 1809 oder höher sein.
- Die Container müssen Windows-Container, die im Prozess-isolierten Modus ausgeführt werden.
- Der Container-Host muss Docker-Modul 19.03 oder höher ausgeführt werden.

## <a name="run-a-container-with-a-device"></a>Führen Sie einen Container mit einem Gerät

Um einen Container mit einem Gerät zu starten, verwenden Sie den folgenden Befehl ein:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Ersetzen Sie die `{interface class guid}` mit einer entsprechenden [Gerät Schnittstellenklassen-GUID](https://docs.microsoft.com/windows-hardware/drivers/install/overview-of-device-interface-classes), die finden Sie unten im Abschnitt.

Um einen Container mit mehreren Geräten zu starten, verwenden Sie den folgenden Befehl aus, und verbinden mehrere `--device` Argumente:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

In Windows deklarieren Sie allen Geräten eine Liste der Schnittstellenklassen, die sie implementieren. Übergeben diesen Befehl für Docker, wird sie sicherstellen, dass alle Geräte, die als die angeforderte Klasse implementieren identifizieren, die in den Container konfiguriert werden werden.

Dies bedeutet, dass Sie das Gerät Weg Host **nicht** zuweisen. Stattdessen wird es der Host mit dem Container teilen. Ebenso werden _Alle_ Geräte, die diese GUID zu implementieren, da Sie eine Klasse GUID angeben, mit dem Container freigegeben.

## <a name="what-devices-are-supported"></a>Was sind Geräte unterstützt.

Die folgenden Geräte (und ihr Gerät Klasse GUIDs Schnittstelle) werden derzeit unterstützt:
  
<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:75%" cellpadding="5" cellspacing="5">
<thead>
<tr valign="top">
<th><center>Gerätetyp</center></th>
<th><center>Schnittstellenklassen-GUID</center></th>
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
<td><center>COM-Anschluss</center></td>
<td><center>86E0D1E0-8089-11D0-9CE4-08003E301F73</center></td>
</tr>
<tr valign="top">
<td><center>SPI-Bus</center></td>
<td><center>DCDE6AF9-6610-4285-828F-CAAF78C424CC</center></td>
</tr>
<tr valign="top">
<td><center>DirectX-GPU-Beschleunigung</center></td>
<td><center>Lesen Sie dedizierte Dokumentation</center></td>
</tr>
</tbody>
</table>

> [!TIP]
> Der oben aufgeführten Geräte sind die Geräte _nur_ in Windows-Container heute unterstützt. Bei dem Versuch, eine andere Klasse GUIDs übergeben führt im Container nicht gestartet.

## <a name="hyper-v-isolated-windows-container-support"></a>Hyper-V-isolierten Windows-Container-Unterstützung

Gerätezuweisung und Geräte, die Freigabe für Workloads in Hyper-V-isolierten Windows-Containern wird nicht heute unterstützt.

## <a name="hyper-v-isolated-linux-container-support"></a>Hyper-V-isolierten Linux-Container-Unterstützung

Gerätezuweisung und Freigabe für Workloads in Hyper-V-isolierten Linux-Container-Gerät wird nicht heute unterstützt.
