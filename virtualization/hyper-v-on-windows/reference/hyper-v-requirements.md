---
title: Systemanforderungen von Hyper-V unter Windows 10
description: Systemanforderungen von Hyper-V unter Windows 10
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6e5e6b01-7a9d-4123-8cc7-f986e10cd372
ms.openlocfilehash: ebc9be132f05c20eb8daf9b5e6713b9258012305
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853988"
---
# <a name="windows-10-hyper-v-system-requirements"></a>Systemanforderungen von Hyper-V unter Windows 10

Hyper-V ist in der 64-Bit-Version von Windows 10 pro, Enterprise und Education verfügbar. Hyper-V erfordert eine Adressübersetzung der zweiten Ebene (SLAT) – diese ist in der aktuellen Generation der 64-Bit-Prozessoren von Intel und AMD vorhanden.

Auf einem Host mit 4 GB RAM können Sie drei bis vier virtuelle Computer ausführen. Für weitere virtuelle Computer benötigen Sie allerdings weitere Ressourcen. Am anderen Ende des Spektrums können Sie auch, abhängig von der physischen Hardware, große virtuelle Computer mit 32 Prozessoren und 512 GB RAM erstellen.

## <a name="operating-system-requirements"></a>Betriebssystemanforderungen

Die Rolle „Hyper-V“ kann bei diesen Versionen von Windows 10 aktiviert werden:

- Windows 10 Enterprise
- Windows 10 Pro
- Windows 10 Education

Die Rolle „Hyper-V“ kann bei diesen Versionen **nicht** installiert werden:

- Windows 10 Home
- Windows 10 Mobile
- Windows 10 Mobile Enterprise

>Für Windows 10 Home Edition kann ein Upgrade auf Windows 10 pro durchgeführt werden. Öffnen Sie dazu **Einstellungen** > **Update und Sicherheit** > **Aktivierung**. Hier können Sie den Store besuchen und ein Upgrade erwerben.

## <a name="hardware-requirements"></a>Hardwareanforderungen

Dieses Dokument bietet zwar keine vollständige Liste von mit Hyper-V kompatibler Hardware, doch die folgenden Elemente sind erforderlich:

- 64-Bit-Prozessor mit Second Level-Adressübersetzung (SLAT).
- CPU-Unterstützung für VM Monitor Mode Extension (VT-x für Intel CPU).
- Mindestens 4 GB Speicher. Doch da virtuelle Computer Arbeitsspeicher mit dem Hyper-V-Host gemeinsam nutzen, müssen Sie ausreichend Arbeitsspeicher bereitstellen, damit der erwartete virtuelle Workload bewältigt werden kann.

Die folgenden Elemente müssen im BIOS des Systems aktiviert sein:
- Virtualisierungstechnologie (wird je nach Hersteller der Hauptplatine anders bezeichnet)
- Von der Hardware erzwungene Datenausführungsverhinderung (Data Execution Prevention, DEP)

## <a name="verify-hardware-compatibility"></a>Prüfen der Hardwarekompatibilität

Nachdem Sie das Betriebssystem und die Hardwareanforderungen überprüft haben, überprüfen Sie die Hardware Kompatibilität in Windows, indem Sie eine PowerShell-Sitzung oder ein Eingabe Aufforderungs Fenster (cmd. exe) öffnen, **System Info**eingeben und dann den Abschnitt "Hyper-V-Anforderungen" überprüfen. Wenn alle aufgelisteten Hyper-V-Anforderungen den Wert **Ja** haben, kann die Rolle „Hyper-V“ auf Ihrem System ausgeführt werden. Wenn für ein Element **Nein** zurückgegeben wird, überprüfen Sie die in diesem Dokument aufgeführten Anforderungen und nehmen, sofern möglich, Anpassungen vor.

![](media/SystemInfo-upd.png)

## <a name="final-check"></a>Abschließende Prüfung

Wenn alle Betriebssystem-, Hardware-und Kompatibilitätsanforderungen erfüllt sind, wird in der **Systemsteuerung** **Hyper-V** angezeigt: Aktivieren oder deaktivieren Sie Windows-Funktionen, und es werden zwei Optionen angeboten.

1. Hyper-V-Plattform
1. Hyper-V-Verwaltungstools

![](media/hyper_v_feature_screenshot.png)

> [!NOTE] Wenn in der Systemsteuerung **Windows Hypervisor Platform** anstelle von **Hyper-v** angezeigt wird **: Windows-Features aktivieren oder > deaktivieren** Sie das System möglicherweise nicht mit Hyper-v kompatibel, und überprüfen Sie dann die Anforderungen.
>
>Wenn Sie **systeminfo** auf einem vorhandenen Hyper-V-Host ausführen, enthält der Abschnitt „Hyper-V-Anforderungen“ Folgendes:
>
>```
>Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V will not be displayed.
>```
