---
title: "Problembehandlung für Hyper-V unter Windows 10"
description: "Problembehandlung für Hyper-V unter Windows 10"
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: f0ec8eb4-ffc4-4bf1-9a19-7a8c3975b359
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: eff2016da5c76489c7e0607d92509a63ff6153b1

---

# Problembehandlung für Hyper-V unter Windows 10

## Ich habe auf Windows 10 aktualisiert und kann jetzt keine Verbindung mehr mit meinem Host mit der Vorgängerversion (Windows 8.1 oder Server 2012 R2) herstellen.
In Windows 10 wurde der Hyper-V-Manager für die Remoteverwaltung nach WinRM verschoben.  Das bedeutet, dass jetzt die Remoteverwaltung auf dem Remotehost aktiviert werden muss, damit dieser mit dem Hyper-V-Manager verwaltet werden kann.

Weitere Informationen finden Sie unter [Verwalten von Hyper-V-Remotehosts](remote_host_management.md).

## Ich habe den Prüfpunkttyp geändert, aber dennoch wird weiter der falsche Prüfpunkttyp verwendet.
Wenn Sie den Prüfpunkt von VMConnect verwenden und den Prüfpunkttyp im Hyper-V-Manager ändern, wird der Prüfpunkt verwendet, dessen Typ angegeben war, als VMConnect geöffnet wurde.

Schließen Sie VMConnect, und öffnen Sie es erneut, um den richtigen Prüfpunkttyp festzulegen.

## Beim Versuch, eine virtuelle Festplatte auf einem USB-Speicherstick zu erstellen, wird eine Fehlermeldung angezeigt.
Hyper-V unterstützt keine mit FAT/FAT32 formatierten Laufwerke, da diese Dateisysteme keinen Zugriffssteuerungslisten (ACLs) bieten und Dateien nur bis maximal 4 GB unterstützen. Mit ExFAT formatierte Datenträger bieten nur eine eingeschränkte ACL-Funktionalität und werden daher auch aus Sicherheitsgründen nicht unterstützt.
Die in PowerShell angezeigte Fehlermeldung lautet: „Fehler beim Erstellen von '\[Pfad, VHD\]': Der angeforderte Vorgang konnte aufgrund einer Dateisystemeinschränkung nicht abgeschlossen werden (0x80070299).“

Verwenden Sie stattdessen ein mit NTFS formatiertes Laufwerk. 

## Beim Versuch der Installation wird diese Meldung angezeigt: „Hyper-V kann nicht installiert werden: die Adressübersetzung der zweiten Ebene (Second Level Address Translation, SLAT) wird vom Prozessor nicht unterstützt.“
Hyper-V erfordert SLAT, um virtuelle Maschinen auszuführen. Wenn Ihr Computer SLAT nicht unterstützt, kann er nicht als Host für virtuelle Computer fungieren.

Wenn Sie nur die Verwaltungstools installieren möchten, deaktivieren Sie **Hyper-V-Plattform** in **Programme und Funktionen** > **Windows-Features aktivieren oder deaktivieren**.



<!--HONumber=Oct16_HO4-->


