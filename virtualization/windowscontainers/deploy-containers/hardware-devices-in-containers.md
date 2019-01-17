---
title: Geräte-Container unter Windows
description: Welche Unterstützung von Geräten für Windows-Container vorhanden ist.
keywords: Docker, Container, Geräte, hardware
author: cwilhit
ms.openlocfilehash: 6397a5050ee0c7cb4b62dc935af4975d9ab6b3db
ms.sourcegitcommit: 1b6a244c3604e48c42c851e580e3b59e2384c91a
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 01/17/2019
ms.locfileid: "9014517"
---
# <a name="devices-in-containers-on-windows"></a>Geräte-Container unter Windows

Standardmäßig sind Windows-Container minimalen Zugriff auf Hostgeräte – genau wie Linux-Container gewährt. Es gibt bestimmte Workloads, in denen es ist von Vorteil – oder sogar imperativer – zugreifen und mit Host-Hardware-Geräten kommunizieren. Dieses Handbuch behandelt, welche Geräte in Containern unterstützt werden und die ersten Schritte.

## <a name="requirements"></a>Anforderungen

- Sie müssen ausführen, Windows Server 2019 oder höher oder Windows 10 Pro/Enterprise mit den Oktober 2018 aktualisieren
- Die Container-Image-Version muss 1809 oder höher sein.
- Die Container müssen Windows-Container, die im Prozess isoliert ausgeführt werden.
- Während die Windows-Geräte-Funktionalität in der Docker-Daemon vorhanden ist, es ist noch nicht vorhanden in der Docker-Client (siehe [Pull-Anforderung](https://github.com/docker/cli/pull/1606) zum Nachverfolgen). In der Interrim müssen Sie [Ihre eigenen Docker ausführbare Dateien erstellen](https://github.com/moby/moby/blob/master/docs/contributing/software-req-win.md) , aus der Quelle Moby als umgehen. Wenn Sie nicht vertraut sind, empfehlen wir, dass Sie warten, bis die oben verknüpfte PR zusammengeführt wird diese Funktion zu testen.

## <a name="run-a-container-with-a-device"></a>Führen Sie einen Container mit einem Gerät

Um einen Container mit einem Gerät zu starten, verwenden Sie den folgenden Befehl aus:

```shell
docker run --isolation=process --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

Ersetzen Sie die `{interface class guid}` mit einer entsprechenden [Gerät Schnittstellenklassen-GUID](https://docs.microsoft.com/en-us/windows-hardware/drivers/install/overview-of-device-interface-classes), die im folgenden Abschnitt gefunden werden können.

Um einen Container mit mehreren Geräten zu starten, verwenden Sie den folgenden Befehl ein, und verbinden mehrere `--device` Argumente:

```shell
docker run --isolation=process --device="class/{interface class GUID}" --device="class/{interface class GUID}" mcr.microsoft.com/windows/servercore:1809
```

In Windows deklarieren Sie allen Geräten eine Liste der Schnittstellenklassen, die sie implementieren. Übergeben diesen Befehl für Docker, wird sie sicherstellen, dass alle Geräte das implementiert die angeforderte Klasse identifiziert, die in den Container konfiguriert werden werden.

Dies bedeutet, dass Sie das Gerät von Host **nicht** zuweisen. Stattdessen wird sie der Host mit dem Container teilen. Ebenso werden _Alle_ Geräte, die diese GUID zu implementieren, da Sie eine Klasse GUID angeben, mit dem Container freigegeben.

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
</tbody>
</table>

> [!TIP]
> Der oben aufgeführten Geräte sind die Geräte _nur_ heute in Windows-Container unterstützt. Bei dem Versuch, eine andere Klasse GUIDs übergeben führt in der Container gestartet.

## <a name="hyper-v-container-device-support"></a>Unterstützung für Hyper-V-Container-Geräte

Gerätezuweisung Geräte- und werden nicht im Hyper-V isolierte Container heute unterstützt.

## <a name="linux-containers-on-windows-lcow-device-support"></a>Linux-Container unter Windows (LCOW) Unterstützung von Geräten

Gerätezuweisung Geräte- und werden nicht in LCOW heute unterstützt.