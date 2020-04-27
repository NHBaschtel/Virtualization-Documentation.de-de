---
title: Erstellen eines virtuellen Netzwerks
description: Erstellen eines virtuellen Switches
keywords: Windows 10, Hyper-v, Networking
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 532195c6-564b-4954-97c2-5a5795368c09
ms.openlocfilehash: 0139f51e909149dde59f4030c6571aee82fed27e
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/24/2020
ms.locfileid: "74909480"
---
# <a name="create-a-virtual-network"></a>Erstellen eines virtuellen Netzwerks

Ihre virtuellen Computer benötigen ein virtuelles Netzwerk, wenn ein Netzwerk auf Ihrem Computer freigegeben werden soll.  Das Erstellen eines virtuellen Netzwerks ist optional. Wenn Ihr virtueller Computer nicht mit dem Internet oder einem Netzwerk verbunden sein muss, fahren Sie fort mit [Erstellen eines virtuellen Windows-Computers](create-virtual-machine.md).


## <a name="connect-virtual-machines-to-the-internet"></a>Verbinden von virtuellen Computern mit dem Internet

Hyper-V unterstützt drei Arten virtueller Switches – externe, interne und private. Erstellen Sie einen externen Switch zum Teilen des Netzwerks Ihres Computers mit den darin ausgeführten virtuellen Computern.

In dieser Übung wird das Erstellen eines externen virtuellen Switches behandelt. Nach Abschluss der Übung verfügt Ihr Hyper-V-Host über einen virtuellen Switch, mit dem virtuelle Computer über die Netzwerkverbindung des Computers eine Verbindung mit dem Internet herstellen können. 

### <a name="create-a-virtual-switch-with-hyper-v-manager"></a>Erstellen eines virtuellen Switches mit dem Hyper-V-Manager

1. Öffnen Sie den Hyper-V-Manager.  Die schnellste Möglichkeit dafür ist: Wählen Sie die Windows-Taste, und geben Sie dann „Hyper-V-Manager“ ein.  
Wenn die Suche den Hyper-V-Manager nicht findet, sind Hyper-V oder die Hyper-V-Verwaltungstools nicht aktiviert.  Befolgen Sie die Anweisungen in [Aktivieren von Hyper-V](enable-hyper-v.md).

2. Wählen Sie den Server im linken Bereich, oder klicken Sie im rechten Bereich auf „Verbindung zum Server...“.

3. Wählen Sie in Hyper-V-Manager **Virtueller Switch-Manager...** aus dem Menü „Aktionen“ auf der rechten Seite. 

4. Wählen Sie unter „Virtuelle Switches“ die Option **Neuer virtueller Netzwerkswitch** aus.

5. Klicken Sie unter „Welche Art von virtuellem Switch möchten Sie erstellen?“ auf **Extern**.

6. Klicken Sie auf die Schaltfläche **Virtuellen Switch erstellen**.

7. Geben Sie unter „Eigenschaften für virtuelle Switches“ dem neuen Switch einen Namen, z. B. **Externer VM-Switch**.

8. Stellen Sie unter „Verbindungstyp“ sicher, dass **Externes Netzwerk** ausgewählt wurde.

9. Wählen Sie die physische Netzwerkkarte, die mit dem neuen virtuellen Switch verbunden werden soll. Dies ist die Netzwerkkarte, die physisch mit dem Netzwerk verbunden ist.  

    ![](media/newSwitch_upd.png)

10. Wählen Sie **Übernehmen** aus, um den virtuellen Switch zu erstellen. An dieser Stelle wird wahrscheinlich die folgende Meldung angezeigt. Klicken Sie auf **Ja**, um den Vorgang fortzusetzen.

    ![](media/pen_changes_upd.png)  

11. Klicken Sie auf **OK**, um das Fenster „Manager für virtuelle Switches“ zu schließen.


### <a name="create-a-virtual-switch-with-powershell"></a>Erstellen eines virtuellen Switches mit PowerShell

Über die folgenden Schritte mit PowerShell können Sie einen virtuellen Switch mit einer externen Verbindung erstellen. 

1. Verwenden Sie **Get-NetAdapter**, um eine Liste der Netzwerkadapter zurückzugeben, die mit dem Windows 10-System verbunden sind.

    ```powershell
    PS C:\> Get-NetAdapter

    Name                      InterfaceDescription                    ifIndex Status       MacAddress             LinkSpeed
    ----                      --------------------                    ------- ------       ----------             ---------
    Ethernet 2                Broadcom NetXtreme 57xx Gigabit Cont...       5 Up           BC-30-5B-A8-C1-7F         1 Gbps
    Ethernet                  Intel(R) PRO/100 M Desktop Adapter            3 Up           00-0E-0C-A8-DC-31        10 Mbps  
    ```

2. Wählen Sie den Netzwerkadapter aus, der mit dem Hyper-V-Switch verwendet werden soll, und legen Sie eine Instanz in einer Variablen namens **$net** ab.

    ```powershell
    $net = Get-NetAdapter -Name 'Ethernet'
    ```

3. Führen Sie den folgenden Befehl zum Erstellen des neuen virtuellen Hyper-V-Switches aus.

    ```powershell
    New-VMSwitch -Name "External VM Switch" -AllowManagementOS $True -NetAdapterName $net.Name
    ```

## <a name="virtual-networking-on-a-laptop"></a>Virtuelle Netzwerke auf dem Laptop

### <a name="nat-networking"></a>NAT-Networking
Die Netzwerkadressenübersetzung (Network Address Translation, NAT) ermöglicht dem virtuellen Computer durch die Kombination der IP-Adresse des Hostcomputers mit einem Port den Zugriff auf das Netzwerk Ihres Computers über einen internen virtuellen Hyper-V-Switch.

Dies hat einige Vorteile:
1. NAT spart IP-Adressen, indem einer großen Anzahl interner IP-Adressen nur eine externe IP-Adresse und ein Port zugeordnet werden. 
2. NAT ermöglicht mehreren virtuellen Computern das Hosten von Anwendungen, die identische (interne) Kommunikationsports benötigen, indem diese eindeutigen externen Ports zugeordnet werden.
3. NAT verwendet einen internen Switch. Sie müssen somit nicht Netzwerkverbindung verwenden, wodurch das Networking eines Computers weniger beeinflusst wird.

Im [Benutzerhandbuch für NAT-Networking](../user-guide/setup-nat-network.md) finden Sie eine Anleitung, um ein NAT-Netzwerk einzurichten und eine Verbindung mit einem virtuellen Computer herzustellen.

### <a name="the-two-switch-approach"></a>Zwei Switches verwenden

Wenn Sie Windows 10 Hyper-V auf einem Laptop ausführen und häufig zwischen einem Drahtlosnetzwerk und einem kabelgebundenen Netzwerk wechseln, empfiehlt es sich, einen virtuellen Switch für die Ethernet-Netzwerkkarte und einen für die WLAN-Netzwerkkarte zu erstellen.  Je nachdem, wie der Laptop mit dem Netzwerk verbunden ist, können Sie Ihre virtuellen Computer zwischen diesen Switches umschalten. Virtuelle Computer wechseln nicht automatisch zwischen kabelgebundenen und drahtlosen Netzwerken. 

>[!IMPORTANT]
>Der Ansatz mit zwei Switches unterstützt keinen externen vSwitch über die Drahtloskarte und sollte nur zu Testzwecken verwendet werden.

## <a name="next-step---create-a-virtual-machine"></a>Nächster Schritt: Erstellen eines virtuellen Computers
[Erstellen eines virtuellen Windows-Computers](create-virtual-machine.md)
