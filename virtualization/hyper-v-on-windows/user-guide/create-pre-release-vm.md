---
title: Testen von Features der Vorabversion für Hyper-V
description: Testen von Features der Vorabversion für Hyper-V
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 426c87cc-fa50-4b8d-934e-0b653d7dea7d
ms.openlocfilehash: 725466f657ae8fc4f14813822e90657e12d26fa6
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439557"
---
# <a name="try-pre-release-features-for-hyper-v"></a>Testen von Features der Vorabversion für Hyper-V

> Dieser Inhalt ist vorläufig und kann geändert werden.  
  Virtuelle Computer mit Vorabversion sind nur für Entwicklungs- oder Testumgebungen gedacht, da sie von Microsoft nicht unterstützt werden.

Sichern Sie sich einen frühen Zugriff auf Vorabfeatures für Hyper-V unter Windows Server 2016 Technical Preview, um diese in Ihrer Entwicklungs- oder Testumgebung auszuprobieren. Sie könnten der Erste sein, der die neuesten Hyper-V-Features zu sehen bekommt, und können das Produkt mitgestalten, indem Sie frühzeitig Feedback bereitstellen.

Für die virtuellen Computer, die Sie als Vorabversion erstellen, sind weder buildübergreifende Kompatibilität noch künftige Unterstützung gegeben.  Verwenden Sie diese nicht in einer Produktionsumgebung.

Hier einige weitere Gründe dafür, dass diese Computer ausschließlich in nicht produktiven Umgebungen eingesetzt werden dürfen:

* Für virtuelle Computer mit Vorabversion ist keine Aufwärtskompatibilität gegeben. Sie können kein Upgrade dieser virtuellen Computer auf eine neue Konfigurationsversion durchführen.
* Für virtuelle Computer mit Vorabversion ist keine konsistente buildübergreifende Definition vorhanden. Wenn Sie das Hostbetriebssystem aktualisieren, sind vorhandene virtuelle Computer mit Vorabversion möglicherweise nicht mehr mit dem Host kompatibel. Diese virtuellen Computer können möglicherweise nicht mehr gestartet werden, oder sie scheinen zunächst zu funktionieren, später treten aber erhebliche Kompatibilitätsprobleme auf.
* Wenn Sie einen virtuellen Computer mit Vorabversion auf einen Host mit einem anderen Build importieren, sind die Ergebnisse unvorhersehbar. Sie können einen virtuellen Computer mit Vorabversion auf einen anderen Host verschieben. Dieses Szenario funktioniert aber voraussichtlich nur, wenn auf beiden Hosts der gleiche Build ausgeführt wird.

## <a name="create-a-pre-release-virtual-machine"></a>Erstellen eines virtuellen Computers mit Vorabversion

Sie können einen virtuellen Computer mit Vorabversion auf Hyper-V-Hosts erstellen, auf denen Windows Server 2016 Technical Preview ausgeführt wird.

1. Klicken Sie auf dem Windows-Desktop auf die Schaltfläche „Start“, und geben Sie einen beliebigen Teil des Namens **Windows PowerShell** ein.
2. Klicken Sie mit der rechten Maustaste auf **Windows PowerShell**, und wählen Sie **Als Administrator ausführen** aus.
3. Verwenden Sie das Cmdlet [New-VM](https://docs.microsoft.com/powershell/module/hyper-v/new-vm?view=win10-ps) mit dem Flag „-Prerelease“, um den virtuellen Computer mit Vorabversion zu erstellen. Führen Sie z. B. folgenden Befehl aus – „VM Name“ steht für den Namen des virtuellen Computers, den Sie erstellen möchten.

``` PowerShell
New-VM -Name <VM Name> -Prerelease
```
Hier einige weitere Beispiele, in denen Sie das Flag „-Prerelease“ verwenden können:
 - Informationen zum Erstellen eines virtuellen Computers, der eine vorhandene virtuelle Festplatte oder eine neue Festplatte verwendet, finden Sie in den PowerShell-Beispielen unter [Create a virtual machine in Hyper-V on Windows Server 2016 Technical Preview](https://docs.microsoft.com/windows-server/virtualization/hyper-v/get-started/Create-a-virtual-machine-in-Hyper-V#BKMK_PowerShell) (Erstellen eines virtuellen Computers in Hyper-V unter Windows Server 2016 Technical Preview).
 - Informationen zum Erstellen einer neuen virtuellen Festplatte, die mit einem Betriebssystemimage gestartet wird, finden Sie im PowerShell-Beispiel unter [Bereitstellen eines virtuellen Windows-Computers in Hyper-V unter Windows 10](https://docs.microsoft.com/virtualization/hyper-v-on-windows/quick-start/create-virtual-machine).

 Die Beispiele in diesen Artikeln funktionieren für Hyper-V-Hosts, auf denen Windows 10 oder Windows Server 2016 Technical Preview ausgeführt wird. Zurzeit können Sie das Flag „-Prerelease“ nur verwenden, um einen virtuellen Computer mit Vorabversion auf Hyper-V-Hosts zu erstellen, auf denen Windows Server 2016 Technical Preview ausgeführt wird.

## <a name="see-also"></a>Weitere Informationen
-  [Blog zum Thema Virtualisierung](https://techcommunity.microsoft.com/t5/Virtualization/bg-p/Virtualization): Erfahren Sie mehr zu verfügbaren Vorabfeatures und dazu, wie Sie diese ausprobieren können.
- [Supported virtual machine configuration versions](https://docs.microsoft.com/windows-server/virtualization/hyper-v/deploy/Upgrade-virtual-machine-version-in-Hyper-V-on-Windows-or-Windows-Server#BKMK_SupportedConfigVersions) (Unterstützte Konfigurationsversionen für virtuelle Computer): Erfahren Sie, wie Sie die Konfigurationsversion des virtuellen Computers überprüfen und welche Versionen von Microsoft unterstützt werden.
