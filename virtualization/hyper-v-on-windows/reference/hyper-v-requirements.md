---
title: Systemanforderungen von Hyper-V unter Windows10
description: Systemanforderungen von Hyper-V unter Windows10
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6e5e6b01-7a9d-4123-8cc7-f986e10cd372
ms.openlocfilehash: 51720a92cd04cfae25c7d0c010ee7b441073dd7e
ms.sourcegitcommit: 5e5644bff6dba70e384db6c80787b3bbe7adb93c
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/03/2018
ms.locfileid: "4303916"
---
# <a name="windows-10-hyper-v-system-requirements"></a>Systemanforderungen von Hyper-V unter Windows10

Hyper-V ist in 64-Bit-Version von Windows 10 Pro, Enterprise und Education verfügbar. Hyper-V erfordert eine Adressübersetzung der zweiten Ebene (SLAT) – diese ist in der aktuellen Generation der 64-Bit-Prozessoren von Intel und AMD vorhanden.

Auf einem Host mit 4GB RAM können Sie drei bis vier virtuelle Computer ausführen. Für weitere virtuelle Computer benötigen Sie allerdings weitere Ressourcen. Am anderen Ende des Spektrums können Sie auch, abhängig von der physischen Hardware, große virtuelle Computer mit 32Prozessoren und 512GB RAM erstellen.

## <a name="operating-system-requirements"></a>Betriebssystemanforderungen

Die Rolle „Hyper-V“ kann bei diesen Versionen von Windows 10 aktiviert werden:

- Windows 10 Enterprise
- Windows10 Pro
- Windows 10 Education

Die Rolle „Hyper-V“ kann bei diesen Versionen **nicht** installiert werden:

- Windows 10 Home
- Windows 10 Mobile
- Windows 10 Mobile Enterprise

>Windows 10 Home Edition kann auf Windows 10 Pro aktualisiert werden. Öffnen Sie dazu **Einstellungen** > **Update und Sicherheit** > **Aktivierung**. Hier können Sie den Store besuchen und ein Upgrade erwerben.

## <a name="hardware-requirements"></a>Hardwareanforderungen

Dieses Dokument bietet zwar keine vollständige Liste von mit Hyper-V kompatibler Hardware, doch die folgenden Elemente sind erforderlich:
    
- 64-Bit-Prozessor mit Second Level Address Translation (SLAT).
- CPU-Unterstützung für VM Monitor Mode Extension (VT-c bei Intel-CPUs).
- Mindestens 4 GB Speicher. Doch da virtuelle Computer Arbeitsspeicher mit dem Hyper-V-Host gemeinsam nutzen, müssen Sie ausreichend Arbeitsspeicher bereitstellen, damit der erwartete virtuelle Workload bewältigt werden kann.

Die folgenden Elemente müssen im BIOS des Systems aktiviert sein:
- Virtualisierungstechnologie (wird je nach Hersteller der Hauptplatine anders bezeichnet)
- Von der Hardware erzwungene Datenausführungsverhinderung (Data Execution Prevention, DEP)

## <a name="verify-hardware-compatibility"></a>Überprüfen der Hardwarekompatibilität

Öffnen Sie zum Überprüfen der Kompatibilität PowerShell oder eine Eingabeaufforderung (cmd.exe), und geben Sie **systeminfo** ein. Wenn alle aufgelisteten Hyper-V-Anforderungen den Wert **Ja** aufweisen, kann die Hyper-V-Rolle auf Ihrem System ausgeführt werden. Wenn für ein Element **Nein** zurückgegeben wird, überprüfen Sie die in diesem Dokument aufgeführten Anforderungen und nehmen, sofern möglich, Anpassungen vor.

![](media/SystemInfo-upd.png)

Wenn Sie **systeminfo** auf einem vorhandenen Hyper-V-Host ausführen, enthält der Abschnitt „Hyper-V-Anforderungen“ Folgendes:

```
Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V are not be displayed.
```