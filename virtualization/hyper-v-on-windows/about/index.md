---
title: Einführung in Hyper-V unter Windows 10
description: Einführung in Hyper-V, Virtualisierung und verwandte Technologien.
keywords: Windows 10, Hyper-V
author: scooley
ms.date: 06/25/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
ms.openlocfilehash: b43c3b112700591f67fcd0247b5ebd88a9c53729
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909360"
---
# <a name="introduction-to-hyper-v-on-windows-10"></a>Einführung in Hyper-V unter Windows 10

Als Entwickler von Software, IT-Experte oder Technologiefan müssen Sie häufig mehrere Betriebssysteme ausführen. Hyper-V ermöglicht Ihnen, mehrere Betriebssysteme als virtuelle Computer unter Windows auszuführen.

![Virtueller Computer unter Windows](media/HyperVNesting.png)

Hyper-V bietet insbesondere Hardwarevirtualisierung an.  Das bedeutet, dass jeder virtuelle Computer auf virtueller Hardware ausgeführt wird.  Mit Hyper-V können Sie virtuelle Festplatten, virtuelle Switches und zahlreiche andere virtuelle Geräte erstellen, die alle virtuellen Computern hinzugefügt werden können.

## <a name="reasons-to-use-virtualization"></a>Gründe für die Virtualisierung

Die Virtualisierung ermöglicht folgendes:

* Führen Sie Software aus, die eine ältere Version von Windows oder andere Betriebssysteme als Windows erfordert.

* Experimentieren Sie mit anderen Betriebssystemen. Hyper-V erleichtert das Inbetriebnehmen und Entfernen verschiedener Betriebssysteme.

* Testen Sie Software unter mehreren Betriebssystemen mithilfe mehrerer virtueller Computer. Mit Hyper-V können Sie alle auf einem einzelnen Desktop- oder Laptopcomputer ausführen. Diese virtuellen Computer können exportiert und anschließend in ein anderes Hyper-V-System, einschließlich Azure, importiert werden.

## <a name="system-requirements"></a>Systemanforderungen

Hyper-V ist in 64-Bit-Versionen von Windows 10 Professional, Enterprise und Education verfügbar. Die Funktion steht für Home Edition nicht zur Verfügung.

> Ein Upgrade von Windows 10 Home auf Windows 10 Professional ist unter **Einstellungen** > **Update und Sicherheit** > **Aktivierung** möglich. Hier können Sie den Store besuchen und ein Upgrade erwerben.

Die meisten Computer führen Hyper-V aus, jedoch führt jeder virtuelle Computer ein völlig separates Betriebssystem aus.  Sie können auf einem Computer mit 4 GB RAM in der Regel einen oder mehrere virtuelle Computer ausführen, obwohl Sie für zusätzliche virtuelle Computer oder zum Installieren und Ausführen von ressourcenintensiver Software wie Spielen, Videos oder -Engineering-Designsoftware weitere Ressourcen benötigen.

Weitere Informationen zu den Systemanforderungen von Hyper-V und der Überprüfung, ob Hyper-V auf Ihrem Computer ausgeführt wird, finden Sie unter [Verweis auf Hyper-V‑Anforderungen](../reference/hyper-v-requirements.md).

## <a name="operating-systems-you-can-run-in-a-virtual-machine"></a>Auf einem virtuellen Computer ausführbare Betriebssysteme

Hyper-V unter Windows unterstützt viele verschiedene Betriebssysteme auf einem virtuellen Computer einschließlich verschiedener Releases von Linux, FreeBSD und Windows.

Zur Erinnerung: Sie müssen für alle Betriebssysteme, die Sie in den virtuellen Computern verwenden, über eine gültige Lizenz verfügen.

Informationen darüber, welche Betriebssysteme als Gäste in Hyper-V unter Windows unterstützt werden, finden Sie unter [Unterstützte Windows-Gastbetriebssysteme](supported-guest-os.md) und [unterstützte Linux Gastbetriebssysteme](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows).

## <a name="differences-between-hyper-v-on-windows-and-hyper-v-on-windows-server"></a>Unterschiede zwischen Hyper-V unter Windows und Hyper-V unter Windows Server

Es gibt einige Features, die in Hyper-V unter Windows und Hyper-V unter Windows Server unterschiedlich funktionieren.

Nur unter Windows Server verfügbare Hyper-V-Features:

* Die Livemigration von einem Host auf einen anderen
* Hyper-V-Replikat
* Virtueller Fibre Channel
* SR-IOV-Netzwerk
* Freigegebene VHDX-Datei

Nur unter Windows 10 verfügbare Hyper-V-Features:

* Schnelles Erstellen und VM-Katalog
* Standardnetzwerk (NAT-Switch)

Das Speicherverwaltungsmodell unterscheidet sich bei Hyper-V unter Windows. Auf einem Server wird Hyper-V-Arbeitsspeicher unter der Annahme verwaltet, dass nur die virtuellen Computer auf dem Server ausgeführt werden. In Hyper-V unter Windows wird Arbeitsspeicher in der Erwartung verwaltet, dass die meisten Clientcomputer sowohl Software auf dem Host als auch virtuelle Computer ausführen.

## <a name="limitations"></a>Einschränkungen

Programme, die von bestimmter Hardware abhängig sind, funktionieren ebenfalls nicht ordnungsgemäß auf einem virtuellen Computer. Beispielsweise sind Spiele oder Anwendungen, die eine Verarbeitung mit GPUs erfordern, ggf. nicht gut geeignet. Außerdem können bei Anwendungen, die Hochpräzisions-Ereigniszeitgeber im Bereich von weniger als 10 ms benötigen (latenzempfindliche Apps, beispielsweise zum Mischen von Livemusik), Probleme auftreten, wenn sie auf einem virtuellen Computer ausgeführt werden.

Darüber hinaus können bei Hyper-V-fähigen Anwendungen mit hohen Anforderungen an Latenz und Präzision auch Probleme bei der Ausführung auf dem Host auftreten.  Dies liegt daran, dass bei virtualisierungsfähigen Anwendungen das Hostbetriebssystem genau wie die Gastbetriebssysteme oberhalb der Hyper-V-Virtualisierungsschicht ausgeführt wird. Im Gegensatz zu Gastbetriebssystemen kann das Hostbetriebssystem direkt auf sämtliche Hardware zugreifen. Dadurch können auch Anwendungen mit speziellen Hardwareanforderungen problemlos im Hostbetriebssystem ausgeführt werden.

## <a name="next-step"></a>Nächster Schritt

[Installieren von Hyper-V unter Windows 10](../quick-start/enable-hyper-v.md)
