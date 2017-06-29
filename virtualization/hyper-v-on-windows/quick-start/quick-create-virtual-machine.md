---
title: Erstellen eines virtuellen Computers mit Hyper-V
description: Erstellen Sie einen neuen virtuellen Computer mit Hyper-V unter Windows 10 Creators Update
keywords: Windows10, Hyper-V
author: aoatkinson
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: fefef30c3ebbda965f9b1077082a05c8e3ffc6c1
ms.sourcegitcommit: 075985b1e0ee62dc1c16573c40cd1f62427ded3c
ms.translationtype: HT
ms.contentlocale: de-DE
---
# <a name="create-a-virtual-machine-with-hyper-v"></a>Erstellen eines virtuellen Computers mit Hyper-V

Erstellen Sie einen virtuellen Computer und installieren Sie sein Betriebssystem.  

Sie benötigen eine ISO-Datei für das von Ihnen ausgewählte Betriebssystem. Wenn Sie dies noch nicht zur Hand haben, holen Sie sich eine Testversion von Windows im [TechNet Evaluation Center](http://www.microsoft.com/en-us/evalcenter/).


> DAs Windows 10 Creators Update stellte ein neues **Schnellerfassungs**-Tool vor, das das Erstellen von neuen virtuellen Computern optimiert.  
  Wenn Sie nicht das Windows 10 Creators Update oder höher verwenden, folgen Sie stattdessen dem Assistent für neue virtuelle Computer:  
  [Erstellen eines neuen virtuellen Computers](create-virtual-machine.md)  
  [Erstellen eines virtuellen Netzwerks](connect-to-network.md)

Los geht's.

![](media/quickcreatesteps_inked.jpg)

1. **Öffnen Sie den Hyper-V-Manager**  
  Drücken Sie entweder die Windows-Taste und geben Sie "Hyper-V-Manager" ein, um Anwendungen für Hyper-V-Manager zu finden oder scrollen Sie durch die Apps im Startmenü, bis Sie Hyper-V-Manager finden.

2. **Öffnen Sie die Schnellerfassung**  
  Suchen Sie in Hyper-V-Manager **Schnellerfassung** im rechten Menü unter **Aktionen**.

3. **Anpassen Ihres virtuellen Computers**
  * (Optional) Geben Sie dem virtuellen Computer einen Namen.  
    Dies ist der Name, den Hyper-V für den virtuellen Computer verwendet, nicht der Computername für das Gastbetriebssystem, das innerhalb des virtuellen Computers bereitgestellt wird.
  * Wählen Sie das Installationsmedium für den virtuellen Computer aus. Sie können ab einer ISO- oder VHDX-Datei installieren.  
    Wenn Sie Windows auf dem virtuellen Computer installieren, können Sie den sicheren Start von Windows aktivieren. Lassen Sie das Kontrollkästchen andernfalls deaktiviert.
  * Einrichten des Netzwerks.  
    Wenn Sie über einen vorhandenen virtuellen Switch verfügen, können Sie ihn aus der Dropdownliste der Netzwerke auswählen. Wenn Sie nicht über einen vorhandenen virtuellen Switch verfügen, sehen Sie eine Schaltfläche, um ein automatisches Netzwerk einzurichten, das automatisch einen externen Switch konfiguriert.

4. **Stellen Sie eine Verbindung mit dem virtuellen Computer her**  
  Wenn Sie **Verbinden** auswählen, wird „Verbindung mit virtuellen Computern” gestartet und Sie starten Ihren virtuellen Computer.     
  Machen Sie sich keine Gedanken über das Bearbeiten der Einstellungen, Sie können jederzeit darauf zurückkehren und sie ändern.  
  
    Sie werden möglicherweise aufgefordert, eine beliebige Taste zu drücken, um von CD oder DVD zu starten. Befolgen Sie diese Aufforderung.  Für den Computer bedeutet dies, sie installieren von einer CD aus.

Herzlichen Glückwunsch, Sie besitzen einen neuen virtuellen Computer.  Jetzt können Sie das Betriebssystem installieren.  

Der virtuelle Computer sollte etwa wie folgt aussehen:  
![](media/OSDeploy_upd.png) 

> **Hinweis:** Um Windows auf einem virtuellen Computer ausführen zu können, benötigen Sie eine separate Lizenz, es sei denn, Sie führen eine Volumenlizenzversion von Windows aus. Das Betriebssystem des virtuellen Computers ist unabhängig vom Betriebssystem des Hosts.
