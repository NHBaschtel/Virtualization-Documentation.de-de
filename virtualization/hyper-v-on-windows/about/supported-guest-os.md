---
title: "Unterstützte Windows-Gäste"
description: "Unterstützte Windows-Gäste."
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: 361b76e18125bb14f5e88b1892eed882a3ae5508
ms.sourcegitcommit: 59541f11d481d8df341597bd73ce7fac14f442ee
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/05/2018
---
# <a name="supported-windows-guests"></a>Unterstützte Windows-Gäste 

In diesem Artikel werden die in Hyper-V unter Windows unterstützten Betriebssystemkombinationen aufgelistet.  Er dient auch als Einführung in Integrationsdienste und andere unterstützte Faktoren.

## <a name="what-does-support-mean"></a>Was bedeutet Unterstützung? 
Unterstützung bedeutet, dass Microsoft diese Host-/Gast-Kombinationen getestet hat.  Bei Problemen mit diesen Kombinationen greift möglicherweise der Produktsupport ein.
 
Microsoft bietet auf folgende Weise Unterstützung für Gastbetriebssysteme:
* Der Microsoft-Support hilft beim Lösen von Problemen in Microsoft-Betriebssystemen und -Integrationsdiensten.
* Für Probleme in anderen Betriebssystemen, die vom Anbieter des Betriebssystems für die Ausführung unter Hyper-V zertifiziert wurden, wird der Support vom Anbieter geleistet.
* Andere in den Betriebssystemen ermittelte Probleme werden von Microsoft an die Supportcommunity mehrerer Anbieter weitergeleitet, [TSANet](http://www.tsanet.org/).

Um unterstützt zu werden, müssen Hyper-V-Host und -Gast über Windows Update mit allen wichtigen Updates aktualisiert werden.

## <a name="supported-guest-operating-systems"></a>Unterstützte Gastbetriebssysteme

Um Unterstützung zu erhalten, müssen Windows-Gastbetriebssysteme und das Hostbetriebssystem über Windows Update mit allen wichtigen Updates auf dem neuesten Stand sein.

| Gastbetriebssystem |  Maximale Anzahl virtueller Prozessoren | Anmerkungen | 
|:-----|:-----|:-----|
| Windows10 | 32 |Der erweiterte Sitzungsmodus funktioniert nicht unter Windows 10 Home Edition |
| Windows 8.1 | 32 | |
| Windows8 | 32 |  |
| Windows 7 mit Service Pack 1 (SP 1) | 4 | Ultimate, Enterprise und Professional Edition (32-Bit und 64-Bit). |
| Windows7 | 4 | Ultimate, Enterprise und Professional Edition (32-Bit und 64-Bit). |
| WindowsVista mit Service Pack 2 (SP2) | 2 | Business, Enterprise und Ultimate einschließlich N- und KN-Editionen. | 
| - | | |
| [Windows Server (Semi-Annual Channel)](https://docs.microsoft.com/en-us/windows-server/get-started/semi-annual-channel-overview) | 64 | |
| Windows Server 2016 | 64 | |
| Windows Server2012 R2 | 64 | |
| Windows Server2012 | 64 | |
| Windows Server 2008 R2 mit Service Pack 1 (SP 1) | 64 | Datacenter, Enterprise, Standard und Web Edition. |
| Windows Server 2008 mit Service Pack 2 (SP 2) | 4 | Datacenter, Enterprise, Standard und Web Edition (32-Bit und 64-Bit). |
| Windows Home Server2011 | 4 | |
| Windows Small Business Server2011 | Essentials Edition - 2, Standard Edition - 4 | |
  
 > Windows 10 kann als Gastbetriebssystem auf Hyper-V-Hosts mit Windows 8.1 und Windows Server 2012 R2 ausgeführt werden.

## <a name="supported-linux-and-free-bsd"></a>Unterstützung für Linux und FreeBSD

| Gastbetriebssystem |  |
|:-----|:------|
| [CentOS und Red Hat Enterprise Linux ](https://technet.microsoft.com/library/dn531026.aspx) | |
| [Virtuelle Debian-Computer in Hyper-V](https://technet.microsoft.com/library/dn614985.aspx) | |
| [SUSE](https://technet.microsoft.com/en-us/library/dn531027.aspx) | |
| [Oracle Linux](https://technet.microsoft.com/en-us/library/dn609828.aspx)  | |
| [Ubuntu](https://technet.microsoft.com/en-us/library/dn531029.aspx) | |
| [FreeBSD](https://technet.microsoft.com/library/dn848318.aspx) | |

Weitere Informationen einschließlich Angaben zur Unterstützung früherer Versionen von Hyper-V finden Sie unter [Virtuelle Linux- und FreeBSD-Computer in Hyper-V](https://technet.microsoft.com/library/dn531030.aspx).
