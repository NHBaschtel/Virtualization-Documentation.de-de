---
title: Gerätefreigabe für virtuelle Windows-Computer
description: Diese Schnellstartanleitung führt Sie durch die Freigabe von Geräten für virtuelle Hyper-V-Computer (USB, Audio, Mikrofon und bereitgestellte Laufwerke).
keywords: Windows10, Hyper-V
ms.author: scooley
ms.date: 10/20/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: d1aeb9cb-b18f-43cb-a568-46b33346a188
ms.openlocfilehash: c891a723d43a9e6e0a0a8bc7bfc2b47a960732d1
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998947"
---
# <a name="share-devices-with-your-virtual-machine"></a>Gerätefreigabe für Ihren virtuellen Computer

> Nur verfügbar für virtuelle Windows-Computer.

Mit dem erweiterten Sitzungsmodus kann sich Hyper-V mit virtuellen Computer via RDP (Remotedesktopprotokoll) verbinden.  Dies verbessert nicht nur die Anzeige virtueller Computer, die Verbindung mit RDP ermöglicht auch, dass der virtuelle Computer Geräte für Ihren Computer freigeben kann.  Da dies unter Windows10 standardmäßig aktiviert ist, verwenden Sie für die Verbindung mit Ihren virtuellen Windows-Computern wahrscheinlich bereits RDP.  In diesem Artikel werden einige der Vorteile und verborgenen Optionen im Dialogfeld für die Verbindungseinstellungen beschrieben.

RDP/Erweiterter Sitzungsmodus:

* Ermöglicht die Größenänderung virtueller Computer und erhöht die DPI-Kompatibilität.
* Verbessert die Integration virtueller Computer.
  * Freigegebene Zwischenablage
  * Dateifreigabe per Drag & Drop und Kopieren/Einfügen
* Ermöglicht die Gerätefreigabe.
  * Mikrofon/Lautsprecher
  * USB-Geräte
  * Datenträger (z.B. "C:")
  * Drucker

In diesem Artikel erfahren Sie, wie Sie Ihren Sitzungstyp anzeigen, den erweiterten Sitzungsmodus aufrufen und Ihre Sitzungseinstellungen konfigurieren.

## <a name="share-drives-and-devices"></a>Freigabe von Laufwerken und Geräten

Die Gerätefreigabefunktionen des erweiterten Sitzungsmodus ist in diesem unscheinbaren Verbindungsfenster verborgen, das geöffnet wird, wenn Sie eine Verbindung mit einem virtuellen Computer herstellen:

![](media/esm-default-view.png)

Standardmäßig geben virtuelle Computer, die den erweiterten Sitzungsmodus verwenden, Zwischenablage und Drucker frei.  Sie sind außerdem standardmäßig so konfiguriert, dass Audio vom virtuellen Computer zurück an den Computerlautsprecher übertragen wird.

So geben Sie Geräte für den virtuellen Computer frei oder ändern diese Standardeinstellungen:

1. Weitere Optionen anzeigen

  ![](media/esm-show-options.png)

1. Lokale Ressourcen anzeigen

  ![](media/esm-local-resources.png)

### <a name="share-storage-and-usb-devices"></a>Freigabe von Speicher und USB-Geräten

Standardmäßig geben virtuelle Computer, die den erweiterten Sitzungsmodus verwenden, Drucker und Zwischenablage frei und übergeben Smartcard- und andere Sicherheitsgeräte an den virtuellen Computer, damit Sie sichere Anmeldetools vom virtuellen Computer verwenden können.

Wählen Sie zum Freigeben anderer Geräten, z.B. USB-Geräte oder Laufwerk "C:", im Menü die Option "Mehr...":  
![](media/esm-more-devices.png)

Hier können Sie die Geräte auswählen, die Sie für den virtuellen Computer freigeben möchten.  Das Systemlaufwerk (Windows C:) ist besonders hilfreich für die Dateifreigabe.  
![](media/esm-drives-usb.png)

### <a name="share-audio-devices-speakers-and-microphones"></a>Freigabe von Audiogeräten (Lautsprecher und Mikrofone)

Standardmäßig geben virtuelle Computer, die den erweiterten Sitzungsmodus verwenden, Audio weiter, sodass Sie Ton vom virtuellen Computer hören.  Der virtuelle Computer verwendet das Audiogerät, das aktuell auf dem Hostcomputer ausgewählt ist.

So ändern Sie diese Einstellungen oder fügen die Mikrofonübergabe hinzu (damit Sie Audio auf einem virtuellen Computer aufzeichnen können):

Menü "Einstellungen..." für die Konfiguration von Remote-Audioeinstellungen auswählen  
![](media/esm-audio.png)

Jetzt Audio- und Mikrofoneinstellungen konfigurieren  
![](media/esm-audio-settings.png)

Da der virtuelle Computer wahrscheinlich lokal ausgeführt wird, erzeugen die Optionen "Auf diesem Computer wiedergeben" und "Auf dem Remotecomputer wiedergeben" das gleiche Ergebnis.

## <a name="re-launching-the-connection-settings"></a>Neustarten der Verbindungseinstellungen

Wenn das Dialogfeld für die Auflösung und Gerätefreigabe nicht angezeigt wird, versuchen Sie, VMConnect unabhängig vom Windows-Menü oder über die Befehlszeile als Administrator zu starten.  

``` Powershell
vmconnect.exe
```

## <a name="check-session-type"></a>Prüfen des Sitzungstyps

Sie können die Art der Verbindung überprüfen, indem Sie das Symbol für die erweiterte Sitzung am oberen Rand des VMConnect-Tools verwenden.  Über diese Schaltfläche können Sie auch zwischen dem einfachen Sitzungsmodus und dem erweiterten Sitzungsmodus umschalten.

![](media/esm-button-location.png)

| Symbol | Verbindungsstatus |
|:-----|:---------|
|![](media/esm-basic.png)| Sie befinden sich derzeit im erweiterten Sitzungsmodus.  Wenn Sie auf dieses Symbol klicken, werden Sie erneut mit dem virtuellen Computer im einfachen Modus verbunden. |
|![](media/esm-connect.png)| Sie befinden sich derzeit im einfachen Sitzungsmodus, der erweiterte Sitzungsmodus ist jedoch verfügbar.  Wenn Sie auf dieses Symbol klicken, werden Sie erneut mit dem virtuellen Computer im erweiterten Sitzungsmodus verbunden.  |
|![](media/esm-stop.png)| Sie befinden sich derzeit im einfachen Modus.  Der erweiterte Sitzungsmodus ist für diesen virtuellen Computer nicht verfügbar. |