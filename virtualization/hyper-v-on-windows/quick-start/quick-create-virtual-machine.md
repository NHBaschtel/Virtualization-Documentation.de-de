---
title: Erstellen eines virtuellen Computers mit Hyper-V
description: Erstellen Sie einen neuen virtuellen Computer mit Hyper-V unter Windows 10 Creators Update
keywords: Windows 10, Hyper-V, virtueller Computer
author: scooley
ms.date: 04/07/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: 6035143bc1449bc4a8e9bb7a4484b4c5329e6d3c
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439627"
---
# <a name="create-a-virtual-machine-with-hyper-v"></a>Erstellen eines virtuellen Computers mit Hyper-V

Erstellen Sie einen virtuellen Computer und installieren Sie sein Betriebssystem.

Wir haben neue Tools zum Erstellen virtueller Computer entwickelt, daher haben sich die Anweisungen während der letzten drei Releases erheblich geändert.

Wählen Sie Ihr Betriebssystem aus, um die richtigen Anweisungen zu erhalten:

* [Windows 10 Fall Creators Update (v1709) oder höher](quick-create-virtual-machine.md#windows-10-fall-creators-update-windows-10-version-1709)
* [Windows 10 Creators Update (v1703)](quick-create-virtual-machine.md#windows-10-creators-update-windows-10-version-1703)
* [Windows 10 Anniversary Update (v1607) und neuere Versionen](quick-create-virtual-machine.md#before-windows-10-creators-update-windows-10-version-1607-and-earlier)

Los geht's.

## <a name="windows-10-fall-creators-update-windows-10-version-1709"></a>Windows 10 Fall Creators Update (Windows 10, Version 1709)

Im Fall Creators Update wurde die Schnellerfassung um einen VM-Katalog erweitert, der unabhängig vom Hyper-V-Manager gestartet werden kann.

So erstellen Sie einen neuen virtuelle Computer im Fall Creators Update:

1. Öffnen Sie die Hyper-V-Schnellerfassung über das Startmenü.

    ![Schnellerfassungskatalog im Windows-Startmenü](media/quick-create-start-menu.png)

1. Wählen Sie ein Betriebssystem aus, oder wählen Sie Ihr eigenes Image mithilfe einer lokalen Installationsquelle aus.

    ![Katalogansicht](media/vmgallery.png)

    1. Wenn Sie Ihr eigenes Image verwenden, um den virtuellen Computer zu erstellen, wählen Sie **Lokale Installationsquelle** aus.
    1. Wählen Sie **Installationsquelle ändern** aus.
      ![Schaltfläche zum Verwenden einer lokalen Installationsquelle](media/change-source.png)
    1. Wählen Sie die ISO- oder VHDX-Datei aus, die Sie für einen neuen virtuellen Computer verwenden möchten.
    1. Wenn das Image ein Linux-Image ist, deaktivieren Sie die Option für sicheren Start.
      ![Schaltfläche zum Verwenden einer lokalen Installationsquelle](media/toggle-secure-boot.png)

1. Wählen Sie „Virtuellen Computer erstellen” aus.

Das war's.  Die Schnellerfassung kümmert sich um den Rest.

## <a name="windows-10-creators-update-windows-10-version-1703"></a>Windows 10 Creators Update (Windows 10, Version 1703)

![Screenshot der Schnellerfassungs-UI](media/quickcreatesteps_inked.jpg)

1. Öffnen Sie den Hyper-V Manager über das Startmenü.

1. Suchen Sie in Hyper-V-Manager **Schnellerfassung** im rechten Menü unter **Aktionen**.

1. Passen Sie Ihren virtuellen Computer an.

    * (Optional) Geben Sie dem virtuellen Computer einen Namen.
    * Wählen Sie das Installationsmedium für den virtuellen Computer aus. Sie können ab einer ISO- oder VHDX-Datei installieren.
    Wenn Sie Windows auf dem virtuellen Computer installieren, können Sie den sicheren Start von Windows aktivieren. Lassen Sie das Kontrollkästchen andernfalls deaktiviert.
    * Einrichten des Netzwerks.
    Wenn Sie über einen vorhandenen virtuellen Switch verfügen, können Sie ihn aus der Dropdownliste der Netzwerke auswählen. Wenn Sie nicht über einen vorhandenen virtuellen Switch verfügen, wird eine Schaltfläche angezeigt, um ein automatisches Netzwerk einzurichten, das automatisch ein virtuelles Netzwerk konfiguriert.

1. Klicken Sie auf **Verbinden**, um den virtuellen Computer zu starten. Machen Sie sich keine Gedanken über das Bearbeiten der Einstellungen, Sie können jederzeit darauf zurückkehren und sie ändern.

    Sie werden möglicherweise aufgefordert, eine beliebige Taste zu drücken, um von CD oder DVD zu starten. Befolgen Sie diese Aufforderung.  Für den Computer bedeutet dies, sie installieren von einer CD aus.

Herzlichen Glückwunsch, Sie besitzen einen neuen virtuellen Computer.  Jetzt können Sie das Betriebssystem installieren.

Der virtuelle Computer sollte etwa wie folgt aussehen:

![Startbildschirm des virtuellen Computers](media/OSDeploy_upd.png)

> **Hinweis:** Um Windows auf einem virtuellen Computer ausführen zu können, benötigen Sie eine separate Lizenz, es sei denn, Sie führen eine Volumenlizenzversion von Windows aus. Das Betriebssystem des virtuellen Computers ist unabhängig vom Betriebssystem des Hosts.

## <a name="before-windows-10-creators-update-windows-10-version-1607-and-earlier"></a>Vor Windows 10 Creators Update (Windows 10, Version 1607 und spätere Versionen)

Wenn Sie nicht das Windows 10 Creators Update oder höher verwenden, folgen Sie stattdessen dem Assistent für neue virtuelle Computer:

1. [Erstellen eines virtuellen Netzwerks](connect-to-network.md)
1. [Erstellen eines neuen virtuellen Computers](create-virtual-machine.md)
