---
title: &2124471135 Problembehandlung für Hyper-V unter Windows 10
description: Problembehandlung für Hyper-V unter Windows 10
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &297854728 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: f0ec8eb4-ffc4-4bf1-9a19-7a8c3975b359
---

# Problembehandlung für Hyper-V unter Windows 10

## Ich habe auf Windows 10 aktualisiert und kann jetzt keine Verbindung mehr mit meinem Host mit der Vorgängerversion (Windows 8.1 oder Server 2012 R2) herstellen.

In Windows 10 wurde der Hyper-V-Manager für die Remoteverwaltung nach WinRM verschoben. Das bedeutet, dass jetzt die Remoteverwaltung auf dem Remotehost aktiviert werden muss, damit dieser mit dem Hyper-V-Manager verwaltet werden kann.

Weitere Informationen finden Sie unter <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Verwalten von Hyper-V-Remotehosts</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

## Ich habe den Prüfpunkttyp geändert, aber dennoch wird weiter der falsche Prüfpunkttyp verwendet.

Wenn Sie den Prüfpunkt von VMConnect verwenden und den Prüfpunkttyp im Hyper-V-Manager ändern, wird der Prüfpunkt verwendet, dessen Typ angegeben war, als VMConnect geöffnet wurde.

Schließen Sie VMConnect, und öffnen Sie es erneut, um den richtigen Prüfpunkttyp festzulegen.

## Beim Versuch, eine virtuelle Festplatte auf einem USB-Speicherstick zu erstellen, wird eine Fehlermeldung angezeigt.

Hyper-V unterstützt keine mit FAT/FAT32 formatierten Laufwerke, da diese Dateisysteme keinen Zugriffssteuerungslisten (ACLs) bieten und Dateien nur bis maximal 4 GB unterstützen. Mit ExFAT formatierte Datenträger bieten nur eine eingeschränkte ACL-Funktionalität und werden daher auch aus Sicherheitsgründen nicht unterstützt.
Die in PowerShell angezeigte Fehlermeldung lautet: Fehler beim Erstellen von '\[Pfad, VHD\]': Der angeforderte Vorgang konnte aufgrund einer Dateisystemeinschränkung nicht abgeschlossen werden. (0x80070299).

Verwenden Sie stattdessen ein mit NTFS formatiertes Laufwerk.

## Beim Versuch der Installation wird diese Meldung angezeigt: „Hyper-V kann nicht installiert werden: die Adressübersetzung der zweiten Ebene (Second Level Address Translation, SLAT) wird vom Prozessor nicht unterstützt.“

Hyper-V erfordert SLAT, um virtuelle Maschinen auszuführen. Wenn Ihr Computer SLAT nicht unterstützt, kann er nicht als Host für virtuelle Computer fungieren.

Wenn Sie nur die Verwaltungstools installieren möchten, deaktivieren Sie <g id="2" ctype="x-strong">Hyper-V-Plattform</g> in <g id="4" ctype="x-strong">Programme und Funktionen</g> > <g id="6" ctype="x-strong">Windows-Features aktivieren oder deaktivieren</g>.






<!--HONumber=May16_HO1-->


