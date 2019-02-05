---
title: Geräte im Container unter Windows
description: Welche Device-Unterstützung für Container unter Windows vorhanden ist
keywords: Docker, Container, Geräte, hardware
author: cwilhit
ms.openlocfilehash: da9785b051826efa4bb2c64542a7c75a12ddd2b4
ms.sourcegitcommit: 4490d384ade48699e7f56dc265185dac75bf9d77
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 02/05/2019
ms.locfileid: "9058990"
---
**Dies ist derzeit Vorschau Material. Sehen Sie 4. Artikel im Abschnitt "Anforderungen" unten für Weitere Informationen.**

# <a name="devices-in-containers-on-windows"></a>Geräte im Container unter Windows

Standardmäßig sind Windows-Container minimalen Zugriff auf Hostgeräte – genau wie Linux-Container gewährt. Es gibt bestimmte Workloads, in denen es nützlich – oder sogar imperativer – zugreifen und mit Host Hardwaregeräten kommunizieren. Dieses Handbuch behandelt, welche Geräte in Containern unterstützt werden und die ersten Schritte.

## <a name="requirements"></a>Anforderungen

- Sie müssen ausführen, Windows Server 2019 oder höher oder Windows 10 Pro/Enterprise mit der October 2018 Update
- Die Container-Image-Version muss 1809 oder höher sein.
- Die Container müssen Windows-Container, die im Prozess-isolierten Modus ausgeführt werden.
- Während die Windows-Geräte-Funktionalität in der Docker-Daemon vorhanden ist, es ist noch nicht vorhanden in der Docker-Client (siehe [Pull-Anforderung](https://github.com/docker/cli/pull/1606) zum Nachverfolgen). Sie müssen warten, für eine zukünftige Version von Docker für Windows / Docker EE mit diesem Code, um dieses Feature zu nutzen. Dieses Dokument wird aktualisiert werden, wenn der Status wechselt.

## <a name="run-a-container-with-a-device"></a>Führen Sie einen Container mit einem Gerät

Um einen Container mit einem Gerät zu starten, verwenden Sie den folgenden Befehl ein:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Ersetzen Sie die `{interface class guid}` mit einer entsprechenden [Gerät Schnittstellenklassen-GUID](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/overview-of-device-interface-classes), die finden Sie unten im Abschnitt.

Um einen Container mit mehreren Geräten zu starten, verwenden Sie den folgenden Befehl ein, und verbinden mehrere `--device` Argumente:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

In Windows deklarieren Sie allen Geräten eine Liste der Schnittstellenklassen, die sie implementieren. Indem Sie diesen Befehl an Docker übergeben, wird es sichergestellt, dass alle Geräte, die als die angeforderte Klasse implementieren identifizieren, die in den Container konfiguriert werden werden.

Dies bedeutet, dass Sie das Gerät von Host **nicht** zuweisen. Stattdessen wird es der Host mit dem Container teilen. Ebenso werden _Alle_ Geräte, die diese GUID zu implementieren, da Sie eine Klasse GUID angeben, mit dem Container freigegeben.

## <a name="what-devices-are-supported"></a>Welche Geräte werden unterstützt.

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
</tbody>
</table>

> [!TIP]
> Der oben aufgeführten Geräte sind die Geräte _nur_ in Windows-Container heute unterstützt. Wenn Sie versuchen, eine andere Klasse GUIDs übergeben führt zu der Container gestartet.

## <a name="hyper-v-container-device-support"></a>Unterstützung für Hyper-V-Container-Geräte

Gerätezuweisung Geräte- und werden nicht in Hyper-V isolierte Container heute unterstützt.

## <a name="linux-containers-on-windows-lcow-device-support"></a>Linux-Container unter Windows (LCOW)-Support für Geräte

Gerätezuweisung Geräte- und werden nicht in LCOW heute unterstützt.
