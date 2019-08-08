---
title: Portieren von Hyper-V WMIv1 auf WMIv2
description: Enthält Informationen zum Portieren von Hyper-V-WMIv1 auf WMIv2
keywords: Windows10, Hyper-V, WMIv1, WMIv2, WMI, Msvm_VirtualSystemGlobalSettingData, Root\virtualization
author: scooley
ms.date: 04/13/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: b13a3594-d168-448b-b0a1-7d77153759a8
ms.openlocfilehash: 963a1cc356c34c8d051c427a069c49021e3c0d27
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998907"
---
# <a name="move-from-hyper-v-wmi-v1-to-wmi-v2"></a>Wechsel Sie von Hyper-V-WMI-v1 auf WMI-v2

Die Windows-Verwaltungsinstrumentation (Windows Management Instrumentation, WMI) ist die zugrunde liegende Schnittstelle für die Verwaltung von Hyper-V-Manager und Hyper-V PowerShell-Cmdlets.  Während die meisten Benutzer unsere PowerShell-Cmdlets oder Hyper-V-Manager verwenden, benötigen Entwickler manchmal WMI direkt.  

Es gibt zwei Hyper-V-WMI-Namespaces (oder Hyper-V-WMI-API-Versionen).
* Der WMI-v1-Namespace (Root\virtualization), der unter Windows Server2008 eingeführt wurde und der neuste, der unter Windows Server2012 verfügbar ist
* Die WMI-v2-Namespace (root\virtualization\v2), der mit Windows Server2012 eingeführt wurde

Dieses Dokument enthält Verweise auf Ressourcen für Code, der mit unserem vorherigen WMI-Namespace kommunizierte und wie dieser in den neuen konvertiert werden kann.  Der Artikel dient zunächst als Repository für API-Informationen und Beispielcode bzw. Skripts, die verwendet werden können, um alle Programme oder Skripts zu portieren, die Hyper-V-WMI-APIs aus dem Namespace v1 auf den v2-Namespace verwenden.

## <a name="msdn-samples"></a>MSDN-Beispiele

[Beispiel für die Migration eines virtuellen Hyper-V-Computers](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-machine-aef356ee)  
[Beispiel eines virtuellen Hyper-V-Fiber-Channels](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-virtual-Fiber-35d27dcd)  
[Beispiel eines geplanten virtuellen Hyper-V-Computers](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-planned-virtual-8c7b7499)  
[Beispiel einer Hyper-V-Systemüberwachungsanwendung](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-application-health-dc0294f2)  
[Beispiel für die Verwaltung einer virtuellen Festplatte](http://code.msdn.microsoft.com/windowsdesktop/Virtual-hard-disk-03108ed3)  
[Beispiel einer Hyper-V-Replikation](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-replication-sample-d2558867)  
[Beispiel für Hyper-V-Metriken](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-metrics-sample-2dab2cb1)  
[Beispiel eines dynamischen Hyper-V-Arbeitsspeichers](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-dynamic-memory-9b0b1d05)  
[Filtertreiber für die erweiterbare Hyper-V-Switcherweiterung](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-Extensible-Virtual-e4b31fbb)  
[Beispiel eines Hyper-V-Netzwerks](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-networking-sample-7c47e6f5)  
[Beispiel einer Hyper-V-Ressourcenpoolverwaltung](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-resource-pool-df906d95)  
[Beispiel eines Hyper-V-Wiederherstellungssnapshot](http://code.msdn.microsoft.com/windowsdesktop/Hyper-V-recovery-snapshot-ea72320c)  

## <a name="samples-from-blogs"></a>Beispiele von Blogs

[Hinzufügen eines Netzwerkadapters auf einem virtuellen Computer mithilfe von Hyper-V-WMI-V2-Namespace](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/adding-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Verbinden eines VM-Netzwerkadapters an einen Switch mithilfe von Hyper-V-WMI-V2-Namespace](http://blogs.msdn.com/b/taylorb/archive/2013/07/15/connecting-a-vm-network-adapter-to-a-switch-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Ändern der MAC-Adresse von NIC mithilfe von Hyper-V-WMI-V2-Namespace](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/changing-the-mac-address-of-nic-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Entfernen eines Netzwerkadapters von einem virtuellen Computer mithilfe von Hyper-V-WMI-V2-Namespace](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-network-adapter-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Anfügen einer virtuellen Festplatte an einen virtuellen Computer mithilfe von Hyper-V-WMI-V2-Namespace](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/attaching-a-vhd-to-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Entfernen einer virtuellen Festplatte von einem virtuellen Computer mithilfe von Hyper-V-WMI-V2-Namespace](http://blogs.msdn.com/b/taylorb/archive/2013/08/12/removing-a-vhd-from-a-vm-using-the-hyper-v-wmi-v2-namespace.aspx)  
[Erstellen eines virtuellen Computers mithilfe von Hyper-V-WMI-V2-Namespace](http://blogs.msdn.com/b/virtual_pc_guy/archive/2013/06/20/creating-a-virtual-machine-with-wmi-v2.aspx)

