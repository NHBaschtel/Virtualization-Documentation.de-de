---
title: Problembehandlung für Hyper-V unter Windows 10
description: Problembehandlung für Hyper-V unter Windows 10
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: f0ec8eb4-ffc4-4bf1-9a19-7a8c3975b359
ms.openlocfilehash: 03bbb4494bbbd790f16c4b6afef387905f7c6c83
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910890"
---
# <a name="troubleshoot-hyper-v-on-windows-10"></a>Problembehandlung für Hyper-V unter Windows 10

## <a name="i-updated-to-windows-10-and-now-i-cant-connect-to-my-downlevel-windows-81-or-server-2012-r2-host"></a>Ich habe auf Windows 10 aktualisiert und kann jetzt keine Verbindung mehr mit meinem Host mit der Vorgängerversion (Windows 8.1 oder Server 2012 R2) herstellen.
In Windows 10 wurde der Hyper-V-Manager für die Remoteverwaltung nach WinRM verschoben.  Das bedeutet, dass jetzt die Remoteverwaltung auf dem Remotehost aktiviert werden muss, damit dieser mit dem Hyper-V-Manager verwaltet werden kann.

Weitere Informationen finden Sie unter [Verwalten von Hyper-V-Remotehosts](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/Remotely-manage-Hyper-V-hosts).

## <a name="i-changed-the-checkpoint-type-but-it-is-still-taking-the-wrong-type-of-checkpoint"></a>Ich habe den Prüfpunkttyp geändert, aber dennoch wird weiter der falsche Prüfpunkttyp verwendet.
Wenn Sie den Prüfpunkt von VMConnect verwenden und den Prüfpunkttyp im Hyper-V-Manager ändern, wird der Prüfpunkt verwendet, dessen Typ angegeben war, als VMConnect geöffnet wurde.

Schließen Sie VMConnect, und öffnen Sie es erneut, um den richtigen Prüfpunkttyp festzulegen.

## <a name="when-i-try-to-create-a-virtual-hard-disk-on-a-flash-drive-an-error-message-is-displayed"></a>Beim Versuch, eine virtuelle Festplatte auf einem USB-Speicherstick zu erstellen, wird eine Fehlermeldung angezeigt.
Hyper-V unterstützt keine mit FAT/FAT32 formatierten Laufwerke, da diese Dateisysteme keinen Zugriffssteuerungslisten (ACLs) bieten und Dateien nur bis maximal 4 GB unterstützen. Mit ExFAT formatierte Datenträger bieten nur eine eingeschränkte ACL-Funktionalität und werden daher auch aus Sicherheitsgründen nicht unterstützt.
Die in PowerShell angezeigte Fehlermeldung lautet: „Fehler beim Erstellen von '\[Pfad, VHD\]': Der angeforderte Vorgang konnte aufgrund einer Dateisystemeinschränkung nicht abgeschlossen werden (0x80070299).“

Verwenden Sie stattdessen ein mit NTFS formatiertes Laufwerk. 

## <a name="i-get-this-message-when-i-try-to-install-hyper-v-cannot-be-installed-the-processor-does-not-support-second-level-address-translation-slat"></a>Beim Versuch der Installation wird diese Meldung angezeigt: „Hyper-V kann nicht installiert werden: die Adressübersetzung der zweiten Ebene (Second Level Address Translation, SLAT) wird vom Prozessor nicht unterstützt.“
Hyper-V erfordert SLAT, um virtuelle Maschinen auszuführen. Wenn Ihr Computer SLAT nicht unterstützt, kann er nicht als Host für virtuelle Computer fungieren.

Wenn Sie nur die Verwaltungstools installieren möchten, deaktivieren Sie **Hyper-V-Plattform** in **Programme und Funktionen** > **Windows-Features aktivieren oder deaktivieren**.
