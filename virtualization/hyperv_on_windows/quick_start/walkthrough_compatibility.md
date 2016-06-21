---
title: &396143525 Systemanforderungen von Hyper-V unter Windows 10
description: Systemanforderungen von Hyper-V unter Windows 10
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1749853639 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 6e5e6b01-7a9d-4123-8cc7-f986e10cd372
---

# Systemanforderungen von Hyper-V unter Windows 10

Hyper-V unter Windows 10 funktioniert nur mit einer bestimmten Gruppe von Hardware- und Betriebssystemkonfigurationen. Dieses Dokument zeigt die Hyper-V-Anforderungen, und wie Sie Ihr System auf Kompatibilität überprüfen können.

## Betriebssystemanforderungen

Die Rolle „Hyper-V“ kann bei diesen Versionen von Windows 10 aktiviert werden:

- Windows 10 Enterprise
- Windows 10 Professional
- Windows 10 Education

Die Rolle „Hyper-V“ kann bei diesen Versionen <g id="2" ctype="x-strong">nicht</g> installiert werden:

- Windows 10 Home
- Windows 10 Mobile
- Windows 10 Mobile Enterprise

>Für Windows 10 Home Edition kann ein Upgrade auf Windows 10 Professional erfolgen. Öffnen Sie dazu <g id="2" ctype="x-strong">Einstellungen</g> > <g id="4" ctype="x-strong">Update und Sicherheit</g> > <g id="6" ctype="x-strong">Aktivierung</g>. Hier können Sie den Store besuchen und ein Upgrade erwerben.

## Hardwareanforderungen

Dieses Dokument bietet zwar keine vollständige Liste von mit Hyper-V kompatibler Hardware, doch die folgenden Elemente sind erforderlich:

- 64-Bit-Prozessor mit Second Level Address Translation (SLAT).
- CPU-Unterstützung für VM Monitor Mode Extension (VT-c bei Intel-CPUs).
- Mindestens 4 GB Arbeitsspeicher. Doch da virtuelle Computer Arbeitsspeicher mit dem Hyper-V-Host gemeinsam nutzen, müssen Sie ausreichend Arbeitsspeicher bereitstellen, damit der erwartete virtuelle Workload bewältigt werden kann.

Die folgenden Elemente müssen im BIOS des Systems aktiviert sein:
- Virtualisierungstechnologie (wird je nach Hersteller der Hauptplatine anders bezeichnet)
- Von der Hardware erzwungene Datenausführungsverhinderung (Data Execution Prevention, DEP)

## Überprüfen der Hardwarekompatibilität

Öffnen Sie zum Überprüfen der Kompatibilität PowerShell oder eine Eingabeaufforderung (cmd.exe), und geben Sie <g id="2" ctype="x-strong">systeminfo.exe</g> ein. Wenn alle aufgelisteten Hyper-V-Anforderungen den Wert <g id="2" ctype="x-strong">Ja</g> haben, kann die Rolle „Hyper-V“ auf Ihrem System ausgeführt werden. Wenn für ein Element <g id="2" ctype="x-strong">Nein</g> zurückgegeben wird, überprüfen Sie die in diesem Dokument aufgeführten Anforderungen und nehmen, sofern möglich, Anpassungen vor.

<g id="1" ctype="x-linkText"></g>

Wenn Sie <g id="2" ctype="x-strong">systeminfo.exe</g> auf einem vorhandenen Hyper-V-Host ausführen, enthält der Abschnitt „Hyper-V-Anforderungen“ Folgendes:

```
Hyper-V Requirements: A hypervisor has been detected. Features required for Hyper-V are not be displayed.
```

## Nächster Schritt – Installieren von Hyper-V

<g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Installieren von Hyper-V</g><g id="1CapsExtId3" ctype="x-title"></g></g>






<!--HONumber=May16_HO1-->


