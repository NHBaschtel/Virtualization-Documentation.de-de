---
title: Erstellen eines benutzerdefinierten VM-Katalogs
description: Erstellen Sie Ihre eigenen Einträge im VM-Katalog im Windows 10 Creators Update und höher.
keywords: Windows 10, Hyper-V, Schnellerfassung, virtueller Computer, Katalog
ms.author: scooley
ms.date: 05/04/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d9238389-7028-4015-8140-27253b156f37
ms.openlocfilehash: 1348b9923d9de1314818f13414abdacee2cb9735
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439712"
---
# <a name="create-a-custom-virtual-machine-gallery"></a>Erstellen eines benutzerdefinierten VM-Katalogs

> Windows 10 Fall Creators Update oder höher.

Im Fall Creators Update wurde die Schnellerfassung um einen VM-Katalog erweitert.

![Schnellerfassung: VM-Katalog mit benutzerdefinierten Images](media/vmgallery.png)

Neben den von Microsoft- und Microsoft-Partnern bereitgestellten Images kann der Katalog auch Ihre eigene Images auflisten.

In diesem Artikel finden Sie weitere Details zu:

* Erstellen von virtuellen Computern, die mit dem Katalog kompatibel sind.
* Erstellen einer neuen Katalogquelle.
* Hinzufügen Ihrer benutzerdefinierten Katalogquelle zum Katalog.

## <a name="gallery-architecture"></a>Katalogarchitektur

Der VM-Katalog ist eine grafische Ansicht für eine Gruppe von VM-Quellen, die in der Windows-Registrierung definiert sind.  Jede VM-Quelle ist ein Pfad (lokaler Pfad oder URI) zu einer JSON-Datei mit virtuellen Computern als Listenelemente.

Die Liste der virtuellen Computer, die Sie im Katalog sehen, ist der gesamte Inhalt der ersten Quelle, gefolgt von den Inhalten der zweiten Quelle usw., bis alle verfügbaren virtuellen Computer aufgelistet wurden.  Die Liste wird jedes Mal, wenn Sie den Katalog starten, dynamisch erstellt.

![Katalogarchitektur](media/vmgallery-architecture.png)

Registrierungsschlüssel: `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization`

Wertname: `GalleryLocations`

Typ: `REG_MULTI_SZ`

## <a name="create-gallery-compatible-virtual-machines"></a>Erstellen von virtuellen Computern, die mit dem Katalog kompatibel sind

Virtuelle Computer im Katalog können entweder ein Datenträgerimage (ISO-Datei) oder eine virtuelle Festplatte (VHDX-Datei) sein.

Virtuelle Computer, die von einer virtuellen Festplatte erstellt werden, weisen einige Konfigurationsanforderungen auf:

1. Erstellt für Unterstützung von UEFI-Firmware. Wenn sie mit Hyper-V erstellt wurden, handelt es sich um eine VM der Generation 2.
1. Die virtuelle Festplatte sollte mindestens 20 GB groß sein – denken Sie daran, das dies die maximale Größe ist.  Hyper-V belegt keinen Speicherplatz, den der virtuellen Computer nicht aktiv verwendet.

### <a name="testing-a-new-vm-image"></a>Testen eines neuem VM-Images

Der VM-Katalog erstellt virtuelle Computer mit demselben Mechanismus wie bei der Installation von einer lokalen Installationsquelle.

So bestätigen Sie, dass ein VM-Image gestartet und ausgeführt wird:

1. Öffnen Sie den VM-Katalog (Hyper-V-Schnellerfassung), und wählen Sie **Lokale Installationsquelle** aus.
  ![Schaltfläche zum Verwenden einer lokalen Installationsquelle](media/use-local-source.png)
1. Wählen Sie **Installationsquelle ändern** aus.
  ![Schaltfläche zum Verwenden einer lokalen Installationsquelle](media/change-source.png)
1. Wählen Sie die ISO- oder VHDX-Datei aus, die im Katalog verwendet werden soll.
1. Wenn das Image ein Linux-Image ist, deaktivieren Sie die Option für sicheren Start.
  ![Schaltfläche zum Verwenden einer lokalen Installationsquelle](media/toggle-secure-boot.png)
1. Erstellen die den virtuellen Computer.  Wenn der virtuelle Computer ordnungsgemäß gestartet wird, ist er für den Katalog bereit.

## <a name="build-a-new-gallery-source"></a>Erstellen einer neuen Katalogquelle

Im nächsten Schritt wird eine neue Katalogquelle erstellt.  Mit dieser JSON-Datei werden die virtuellen Computer aufgelistet und alle zusätzlichen Informationen hinzufügt, die Sie im Katalog sehen.

Textinformationen:

![Beschriftete Textpositionen des Katalogs](media/gallery-text.png)

* **Name**: Erforderlich. Dies ist der Name, der in der linken Spalte sowie am oberen Rand der VM-Ansicht angezeigt wird.
* **Herausgeber**: Erforderlich.
* **Beschreibung**: Erforderlich. Die Liste der Zeichenfolgen, die den virtuellen Computer beschreiben.
* **Version**: Erforderlich.
* lastUpdated: Standardeinstellung ist Montag, 1. Januar 0001.

  Das Format ist: jjjj-mm-ttThh:mm:ssZ

  Der folgende PowerShell-Befehl gibt das heutige Datum im richtigen Format an und fügt es in die Zwischenablage ein:

  ``` PowerShell
  Get-Date -UFormat "%Y-%m-%dT%TZ" | clip.exe
  ```

* Gebietsschema: Die Standardeinstellung ist leer.

Bilder:

![Beschriftete Bildpositionen des Katalogs](media/gallery-pictures.png)

* **Logo**: Erforderlich.
* symbol
* Miniaturansicht

Und natürlich Ihr virtueller Computer (ISO- oder VHDX-Datei).

Zum Generieren der Hashwerte können Sie den folgenden PowerShell-Befehl verwenden:

  ``` PowerShell
  Get-FileHash -Path .\TMLogo.jpg -Algorithm SHA256
  ```

Die folgende JSON-Vorlage enthält Startelemente und das Schema des Katalogs.  Wenn Sie sie in VSCode bearbeiten, wird automatisch IntelliSense bereitgestellt.

[!code-json[main](../../../hyperv-tools/vmgallery/vm-gallery-template.json)]

## <a name="connect-your-gallery-to-the-vm-gallery-ui"></a>Verbinden Ihres Katalogs mit der Benutzeroberfläche des VM-Katalogs

Die einfachste Möglichkeit, Ihre benutzerdefinierte VM-Katalogquelle dem VM-Katalog hinzuzufügen, besteht in der Verwendung des Registrierungs-Editors.

1. Öffnen Sie **regedit.exe**.
1. Navigieren Sie zu `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\`.
1. Suchen Sie nach dem `GalleryLocations`-Element.

    Wenn es bereits vorhanden ist, fahren Sie im Menü **Bearbeiten** mit der Option **Ändern** fort.

    Wenn es noch nicht vorhanden ist, navigieren Sie zum Menü **Bearbeiten**, und navigieren Sie dann über **Neu** zu **Wert der mehrteiligen Zeichenfolge**.

1. Fügen Sie Ihren Katalog dem Registrierungsschlüssel `GalleryLocations` hinzu.

    ![Katalogregistrierungsschlüssel mit dem neuen Element](media/new-gallery-uri.png)

## <a name="troubleshooting"></a>Problembehandlung

### <a name="check-for-errors-loading-gallery"></a>Überprüfen auf Fehler beim Laden des Katalogs

Der VM-Katalog bietet Fehlerberichterstattung in der Windows-Ereignisanzeige.  Überprüfung auf Fehler:

1. Öffnen Sie die Ereignisanzeige.
1. Navigieren Sie zu **Windows-Protokolle** -> **Anwendung**.
1. Suchen Sie nach Ereignissen aus der Quelle VMCreate.

## <a name="resources"></a>Ressourcen

In GitHub stehen einige Katalogskripts und Hilfsprogramme zur Verfügung [Link](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/live/hyperv-tools/vmgallery).

Einen Katalogbeispieleintrag finden Sie [hier](https://go.microsoft.com/fwlink/?linkid=851584).  Dies ist die JSON-Datei, die den Posteingangskatalog definiert.
