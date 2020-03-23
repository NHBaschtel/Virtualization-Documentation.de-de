---
title: Unterstützte Windows-Gäste
description: Unterstützte Windows-Gäste.
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: ae4a18ed-996b-4104-90c5-539c90798e4c
ms.openlocfilehash: 25c72b910af15fc0b498a5b2abce72d32e6d1efd
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439597"
---
# <a name="supported-windows-guests"></a>Unterstützte Windows-Gäste

In diesem Artikel werden die in Hyper-V unter Windows unterstützten Betriebssystemkombinationen aufgelistet.  Er dient auch als Einführung in Integrationsdienste und andere unterstützte Faktoren.

Microsoft hat diese Host-/Gast-Kombinationen getestet.  Bei Problemen mit diesen Kombinationen greift möglicherweise der Produktsupport ein.

Microsoft bietet die folgende Unterstützung:

* Der Microsoft-Support hilft beim Lösen von Problemen in Microsoft-Betriebssystemen und -Integrationsdiensten.

* Für Probleme in anderen Betriebssystemen, die vom Anbieter des Betriebssystems für die Ausführung unter Hyper-V zertifiziert wurden, wird der Support vom Anbieter geleistet.

* Andere in den Betriebssystemen ermittelte Probleme werden von Microsoft an die Supportcommunity mehrerer Anbieter weitergeleitet, [TSANet](http://www.tsanet.org/).

Um unterstützt zu werden, müssen alle Betriebssysteme (Gast und Host) auf dem neuesten Stand sein.  Überprüfen Sie Windows Update auf wichtige Updates.

## <a name="supported-guest-operating-systems"></a>Unterstützte Gastbetriebssysteme

| Gastbetriebssystem |  Maximale Anzahl virtueller Prozessoren | Hinweise |
|:-----|:-----|:-----|
| Windows 10 | 32 |Der erweiterte Sitzungsmodus funktioniert nicht unter Windows 10 Home Edition |
| Windows 8.1 | 32 | |
| Windows 8 | 32 ||
| Windows 7 mit Service Pack 1 (SP 1) | 4 | Ultimate, Enterprise und Professional Edition (32-Bit und 64-Bit). |
| Windows 7 | 4 | Ultimate, Enterprise und Professional Edition (32-Bit und 64-Bit). |
| Windows Vista mit Service Pack 2 (SP2) | 2 | Business, Enterprise und Ultimate einschließlich N- und KN-Editionen. |
| - | | |
| [Windows Server (halbjährlicher Kanal)](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview) | 64 | |
| Windows Server 2019 | 64 | |
| Windows Server 2016 | 64 | |
| Windows Server 2012 R2 | 64 | |
| Windows Server 2012 | 64 | |
| Windows Server 2008 R2 mit Service Pack 1 (SP 1) | 64 | Datacenter, Enterprise, Standard und Web Edition. |
| Windows Server 2008 mit Service Pack 2 (SP 2) | 4 | Datacenter, Enterprise, Standard und Web Edition (32-Bit und 64-Bit). |
| Windows Home Server 2011 | 4 | |
| Windows Small Business Server 2011 | Essentials Edition - 2, Standard Edition - 4 | |

> Windows 10 kann als Gastbetriebssystem auf Hyper-V-Hosts mit Windows 8.1 und Windows Server 2012 R2 ausgeführt werden.

## <a name="supported-linux-and-free-bsd"></a>Unterstützung für Linux und FreeBSD

| Gastbetriebssystem |  |
|:-----|:------|
| [CentOS und Red Hat Enterprise Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-CentOS-and-Red-Hat-Enterprise-Linux-virtual-machines-on-Hyper-V) | |
| [Virtuelle Debian-Computer in Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Debian-virtual-machines-on-Hyper-V) | |
| [SUSE](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-SUSE-virtual-machines-on-Hyper-V) | |
| [Oracle Linux](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Oracle-Linux-virtual-machines-on-Hyper-V)  | |
| [Ubuntu](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Ubuntu-virtual-machines-on-Hyper-V) | |
| [FreeBSD](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-FreeBSD-virtual-machines-on-Hyper-V) | |

Weitere Informationen einschließlich Angaben zur Unterstützung früherer Versionen von Hyper-V finden Sie unter [Virtuelle Linux- und FreeBSD-Computer in Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows).
