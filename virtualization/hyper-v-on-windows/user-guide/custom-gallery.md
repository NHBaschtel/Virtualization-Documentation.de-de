---
title: Erstellen einer benutzerdefinierten virtuellen Computergalerie
description: Erstellen Sie Ihre eigenen Einträge in der virtuellen Computergalerie im Windows 10 Creators Update und höher.
keywords: Windows 10, Hyper-V, schnelles Erstellen, virtuelle Computergalerie
ms.author: scooley
ms.date: 05/04/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d9238389-7028-4015-8140-27253b156f37
ms.openlocfilehash: c7a6462b331f469148eb4cf5a0a2740c9929fa29
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74911060"
---
# <a name="create-a-custom-virtual-machine-gallery"></a>Erstellen einer benutzerdefinierten virtuellen Computergalerie

> Windows 10 Fall Creators Update oder höher.

Im Fall Creators Update wurde die Schnellerfassung erweitert, um virtuelle Computergalerie zu enthalten.

![Schnellerfassung VM-Galerie mit benutzerdefinierten Bildern](media/vmgallery.png)

Neben den von Microsoft- und Microsoft-Partnern bereitgestellten Bildern kann die Galerie auch Ihre eigene Bilder anzeigen.

In diesem Artikel finden Sie weitere Details zu:

* Erstellen von virtuellen Computern, die mit der Galerie kompatibel sind.
* Erstellen einer neuen Galeriequelle.
* Hinzufügen Ihrer benutzerdefinierten Galeriequelle zur Galerie.

## <a name="gallery-architecture"></a>Galeriearchitektur

Die VM-Galerie ist eine grafische Ansicht für eine Gruppe von virtuellen Computerquellen, die in der Windows-Registrierung definiert sind.  Jede virtuelle Computerquelle ist ein Pfad (lokaler Pfad oder URI) auf eine JSON-Datei mit virtuellen Computern als Listenelemente.

Die Liste der virtuellen Computer, die Sie in der Galerie finden, ist der gesamte Inhalt der ersten Quelle, gefolgt von den Inhalten der zweiten Quelle usw., bis alle verfügbaren virtuellen Computer aufgelistet sind.  Die Liste wird dynamisch erstellt, jedes Mal, wenn Sie die Galerie starten.

![Galeriearchitektur](media/vmgallery-architecture.png)

Registrierungsschlüssel: `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization`

Wertname: `GalleryLocations`

Typ: `REG_MULTI_SZ`

## <a name="create-gallery-compatible-virtual-machines"></a>Erstellen einer Galerie, die mit virtuellen Computern kompatibel ist

Virtuelle Computer in der Galerie können entweder ein Datenträger-Image (.iso) oder eine virtuelle Festplatte (.vhdx) sein.

Virtuelle Computer, die von einer virtuellen Festplatte erstellt werden, haben Konfigurationsanforderungen:

1. Zur Unterstützung von UEFI-Firmware erstellt. Wenn sie mit Hyper-V erstellt wurden, ist dies eine VM der Generation 2.
1. Die virtuelle Festplatte sollte mindestens 20 GB sein – Denken Sie daran, das dies die maximale Größe ist.  Hyper-V verwendet keinen Speicherplatz, den der virtuellen Computer nicht aktiv verwendet.

### <a name="testing-a-new-vm-image"></a>Testen eines neuem VM-Images

Die VM-Galerie erstellt virtuelle Computer mit demselben Mechanismus wie die Installation von einer lokalen Installationsquelle.

So überprüfen Sie, dass das Bild eines virtuellen Computers gestartet und ausgeführt wird:

1. Öffnen Sie die VM-Galerie (Hyper-V Schnellerfassung), und wählen Sie **Lokalinstallationsquelle** aus.
  ![Schaltfläche, um eine lokale Installationsquelle zu verwenden](media/use-local-source.png)
1. Wählen Sie **Installationsquelle ändern**.
  ![Schaltfläche, um eine lokale Installationsquelle zu verwenden](media/change-source.png)
1. Wählen Sie die .ISO- oder .VHDX-Datei, die in der Galerie verwendet werden soll.
1. Wenn das Bild ein Linux-Bild ist, deaktivieren Sie die Option für den sicheren Start.
  ![Schaltfläche, um eine lokale Installationsquelle zu verwenden](media/toggle-secure-boot.png)
1. Erstellen eines virtuellen Computers  Wenn der virtuelle Computer ordnungsgemäß gestartet wird, ist er für die Galerie bereit.

## <a name="build-a-new-gallery-source"></a>Erstellen einer neuen Galeriequelle

Im nächsten Schritt wird eine neue Galeriequelle erstellt.  Mit dieser JSON-Datei werden die virtuellen Computer aufgelistet und alle zusätzlichen Informationen hinzufügt, die Sie in der Galerie sehen.

Textinformationen:

![Position des Texts der Galeriebezeichnung](media/gallery-text.png)

* **Name** – erforderlich – dies ist der Name, der in der linken Spalte sowie am oberen Rand der VM-Ansicht angezeigt wird.
* **Herausgeber** - erforderlich
* **Beschreibung** – erforderlich – Liste der Zeichenfolgen, die den virtuellen Computer beschreiben.
* **Version** - erforderlich
* LastUpdated – Standardeinstellung ist Montag, 1. Januar 0001.

  Das Format ist: yyyy-mm-ddThh:mm:ssZ

  Der folgende PowerShell-Befehl gibt das heutige Datum im richtigen Forma tan und fügt es in die Zwischenablage ein:

  ``` PowerShell
  Get-Date -UFormat "%Y-%m-%dT%TZ" | clip.exe
  ```

* Gebietsschema – die Standardeinstellung ist leer.

Bilder:

![Position des Bilds der Galeriebezeichnung](media/gallery-pictures.png)

* **Logo** - erforderlich
* symbol
* thumbnail

Und natürlich Ihr virtueller Computer (.iso oder .vhdx).

Um die Hashes zu generieren, können Sie den folgenden PowerShell-Befehl verwenden:

  ``` PowerShell
  Get-FileHash -Path .\TMLogo.jpg -Algorithm SHA256
  ```

Die folgende JSON-Vorlage enthält Startelemente und das Galerieschema.  Wenn Sie sie in VSCode bearbeiten, wird automatisch IntelliSense bereitgestellt.

[!code-json[main](../../../hyperv-tools/vmgallery/vm-gallery-template.json)]

## <a name="connect-your-gallery-to-the-vm-gallery-ui"></a>Verbinden Sie Ihre Galerie mit der VM-Galerie-Benutzeroberfläche

Die einfachste Möglichkeit, Ihre VM-Galeriequelle der benutzerdefinierten Galerie hinzuzufügen ist, sie dem Registrierungs-Editor hinzuzufügen.

1. Öffnen Sie **regedit.exe**
1. Wechseln Sie zu `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\`.
1. Suchen Sie nach `GalleryLocations`.

    Wenn sie bereits vorhanden ist, fahren Sie im Menü **Bearbeiten** mit der Option **ändern** fort.

    Wenn sie nicht bereits vorhanden ist, gehen Sie zum Menü **Bearbeiten** und navigieren Sie auf **Neu** zu **mehrteiliger Zeichenfolgenwert**

1. Fügen Sie Ihre Galerie dem Registrierungsschlüssel `GalleryLocations` hinzu.

    ![Galerie-Registrierungsschlüssel mit dem neuen Element](media/new-gallery-uri.png)

## <a name="troubleshooting"></a>Problembehandlung

### <a name="check-for-errors-loading-gallery"></a>Fehler beim Laden der Galerie überprüfen

Die VM-Galerie bietet die Fehlerberichterstattung in der Windows-Ereignisanzeige.  Überprüfung auf Fehler:

1. Öffnen Sie die Ereignisanzeige.
1. Wechseln Sie zu **Windows-Protokolle** -> **Anwendung**.
1. Suchen Sie nach Ereignissen von der Quelle VMCreate.

## <a name="resources"></a>Ressourcen

In GitHub stehen einige Galerieskripts und Hilfsprogramme zur Verfügung [Link](https://github.com/MicrosoftDocs/Virtualization-Documentation/tree/live/hyperv-tools/vmgallery).

Informationen finden Sie im Beispieleintrag der Galerie [hier](https://go.microsoft.com/fwlink/?linkid=851584).  Dies ist die JSON-Datei, die die Inbox-Galerie definiert.
