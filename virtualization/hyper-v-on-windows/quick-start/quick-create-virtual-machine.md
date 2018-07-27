---
title: Erstellen eines virtuellen Computers mit Hyper-V
description: Erstellen Sie einen neuen virtuellen Computer mit Hyper-V unter Windows 10 Creators Update
keywords: Windows 10, Hyper-V, VM
author: scooley
ms.date: 04/07/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: 3bb873a35d905e9eefada151184d6115bb18d2e7
ms.sourcegitcommit: 9653a3f7451011426f8af934431bb14dbcb30a62
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 06/21/2018
ms.locfileid: "2082871"
---
# <a name="create-a-virtual-machine-with-hyper-v"></a>Erstellen eines virtuellen Computers mit Hyper-V

Erstellen Sie einen virtuellen Computer und installieren Sie sein Betriebssystem.

Wir haben neue Tools zum Erstellen virtueller Maschinen entwickelt, daher haben sich die Anweisungen über die letzten drei Versionen erheblich geändert.

Wählen Sie Ihr Betriebssystem für die richtigen Anweisungen:

* [Windows 10 Fall Creators Update oder höher](quick-create-virtual-machine.md#windows-10-fall-creators-update)
* [Windows 10 Creators Update](quick-create-virtual-machine.md#windows-10-creators-update)
* [Windows 10 Anniversary Update und neuere Versionen](quick-create-virtual-machine.md#before-windows-10-creators-update)

Los geht's.

## <a name="windows-10-fall-creators-update"></a>Windows 10 Fall Creators Update

Im Fall Creators Update wurde die Schnellerfassung mit der Erstellung einer virtuellen Computergalerie erweitert, die unabhängig vom Hyper-V-Manager gestartet werden kann.

So erstellen Sie einen neuen virtuelle Computer im Fall Creators Update:

1. Öffnen Sie über das Startmenü die Hyper-V Schnellerfassung.

    ![Schnellerfassungskatalog im Windows-Startmenü](media/quick-create-start-menu.png)

1. Wählen Sie ein Betriebssystem aus, oder wählen Sie Ihr eigenes mithilfe einer lokalen Installationsquelle aus.

    ![Galerieansicht](media/vmgallery.png)

    1. Wenn Sie Ihr eigenes Bild verwenden, um einen virtuellen Computer zu erstellen, wählen Sie **Lokalinstallationsquelle** aus.
    1. Wählen Sie **Installationsquelle ändern**.
      ![Schaltfläche zum Verwenden einer lokalen Installationsquelle](media/change-source.png)
    1. Wählen Sie die ISO- oder VHDX-Datei, die Sie in einem neuen virtuellen Computer aktivieren möchten.
    1. Wenn das Bild ein Linux-Bild ist, deaktivieren Sie die Option für den sicheren Start.
      ![Schaltfläche zum Verwenden einer lokalen Installationsquelle](media/toggle-secure-boot.png)

1. Wählen Sie „Erstellen Sie Ihren virtuellen Computer”

Das war's.  Die Schnellerfassung kümmert sich um den Rest.

## <a name="windows-10-creators-update"></a>Windows10 Creators Update

![Screenshot der Schnellerfassungs-UI](media/quickcreatesteps_inked.jpg)

1. Öffnen Sie über das Startmenü den Hyper-V Manager.

1. Suchen Sie in Hyper-V-Manager **Schnellerfassung** im rechten Menü unter **Aktionen**.

1. Anpassen Ihres virtuellen Computers.

    * (Optional) Geben Sie dem virtuellen Computer einen Namen.
    * Wählen Sie das Installationsmedium für den virtuellen Computer aus. Sie können ab einer ISO- oder VHDX-Datei installieren.
    Wenn Sie Windows auf dem virtuellen Computer installieren, können Sie den sicheren Start von Windows aktivieren. Lassen Sie das Kontrollkästchen andernfalls deaktiviert.
    * Einrichten des Netzwerks.
    Wenn Sie über einen vorhandenen virtuellen Switch verfügen, können Sie ihn aus der Dropdownliste der Netzwerke auswählen. Wenn Sie nicht über einen vorhandenen virtuellen Switch verfügen, sehen Sie eine Schaltfläche, um ein automatisches Netzwerk einzurichten, das automatisch ein virtuelles Netzwerk konfiguriert.

1. Klicken Sie auf **Verbinden**, um die virtuelle Maschine zu starten. Machen Sie sich keine Gedanken über das Bearbeiten der Einstellungen, Sie können jederzeit darauf zurückkehren und sie ändern.

    Sie werden möglicherweise aufgefordert, eine beliebige Taste zu drücken, um von CD oder DVD zu starten. Befolgen Sie diese Aufforderung.  Für den Computer bedeutet dies, sie installieren von einer CD aus.

Herzlichen Glückwunsch, Sie besitzen einen neuen virtuellen Computer.  Jetzt können Sie das Betriebssystem installieren.

Der virtuelle Computer sollte etwa wie folgt aussehen:

![Startbildschirm des virtuellen Computers](media/OSDeploy_upd.png)

> **Hinweis:** Um Windows auf einem virtuellen Computer ausführen zu können, benötigen Sie eine separate Lizenz, es sei denn, Sie führen eine Volumenlizenzversion von Windows aus. Das Betriebssystem des virtuellen Computers ist unabhängig vom Betriebssystem des Hosts.

## <a name="before-windows-10-creators-update"></a>Vor Windows10 Creators Update

Wenn Sie nicht das Windows 10 Creators Update oder höher verwenden, folgen Sie stattdessen dem Assistent für neue virtuelle Computer:

1. [Erstellen eines virtuellen Netzwerks](connect-to-network.md)
1. [Erstellen eines neuen virtuellen Computers](create-virtual-machine.md)
