---
title: "Einführung in Hyper-V unter Windows 10"
description: "Einführung in Hyper-V, Virtualisierung und verwandte Technologien."
keywords: Windows10, Hyper-V
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
ms.openlocfilehash: 307cd592a9deda41fd2a892d49eadbc5ae436d84
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/21/2017
---
# Einführung in Hyper-V unter Windows 10

> Hyper-V ersetzt Microsoft Virtual PC. 

Als Entwickler von Software, IT-Experte oder Technologiefan müssen Sie häufig mehrere Betriebssysteme ausführen. Anstatt jedem Ihrer Computer physische Hardware dediziert zuzuweisen, ermöglicht Ihnen Hyper-V das Ausführen eines Betriebssystems oder eines Computersystems als virtuellen Computer auf Ihrem Windows-Computer.  

![](media/HyperVNesting.png)

Hyper-V bietet insbesondere Hardwarevirtualisierung an.  Das bedeutet, dass jeder virtuelle Computer auf virtueller Hardware ausgeführt wird.  Mit Hyper-V können Sie virtuelle Festplatten, virtuelle Switches und zahlreiche andere virtuelle Geräte erstellen, die alle virtuellen Computern hinzugefügt werden können.

## Gründe für die Virtualisierung

Die Virtualisierung ermöglicht folgendes:  
* Führen Sie Software aus, die eine ältere Version von Windows oder andere Betriebssysteme als Windows erfordert. 

* Experimentieren Sie mit anderen Betriebssystemen. Hyper-V erleichtert das Inbetriebnehmen und Entfernen verschiedener Betriebssysteme.

* Testen Sie Software unter mehreren Betriebssystemen mithilfe mehrerer virtueller Computer. Mit Hyper-V können Sie alle auf einem einzelnen Desktop- oder Laptopcomputer ausführen. Diese virtuellen Computer können exportiert und anschließend in ein anderes Hyper-V-System, einschließlich Azure, importiert werden.

* Sie können Probleme mit virtuellen Computern in einer beliebigen Hyper-V-Umgebung beheben. Sie können einen virtuellen Computer aus Ihrer Produktionsumgebung exportieren, auf Ihrem Desktopcomputer, auf dem Hyper-V ausgeführt wird, öffnen, die Probleme des virtuellen Computers beheben und ihn dann zurück in die Produktionsumgebung exportieren. 

* Mithilfe eines virtuellen Netzwerks können Sie eine Umgebung mit mehreren Computern zu Test-, Entwicklungs- und Demonstrationszwecken erstellen und gleichzeitig sicherstellen, dass das Produktionsnetzwerk nicht beeinflusst wird.

## Systemanforderungen
Hyper-V ist in 64-Bit-Versionen von Windows Professional, Enterprise, und Education von Windows 8 und höher verfügbar.  Die Funktion steht für Windows Home Edition nicht zur Verfügung.  

>  Ein Upgrade von Windows10 Home auf Windows10 Professional ist unter **Einstellungen** > **Update und Sicherheit** > **Aktivierung** möglich. Hier können Sie den Store besuchen und ein Upgrade erwerben.

Die meisten Computer führen Hyper-V aus, virtuelle Computer benötigen allerdings erhebliche Ressourcen da sie ein vollständiges Betriebssystem ausführen.  Sie können auf einem Computer mit 4GB RAM in der Regel einen oder mehrere virtuelle Computer ausführen, obwohl Sie für zusätzliche virtuelle Computer oder zum Installieren und Ausführen von ressourcenintensiver Software wie Spielen, Videos oder -Engineering-Designsoftware weitere Ressourcen benötigen. 

Müssen Sie Ihrem Computer Adressübersetzung der zweiten Ebene (SLAT) hinzufügen. Diese ist in der aktuellen Generation der 64-Bit-Prozessoren von Intel und AMD vorhanden.  Sie benötigen auch eine 64-Bit-Version von Windows.

Weitere Informationen zu den Systemanforderungen von Hyper-V und der Überprüfung, ob Hyper-V auf Ihrem Computer ausgeführt wird, finden Sie unter [Verweis auf Hyper-V‑Anforderungen](..\reference\hyper-v-requirements.md).

## Auf einem virtuellen Computer ausführbare Betriebssysteme
Der Begriff "Gast" bezieht sich auf einer virtuellen Maschine und "Host" bezieht sich auf den Computer, auf dem virtuellen Computer ausgeführt wird. Hyper-V unter Windows unterstützt viele verschiedene Gastbetriebssysteme, einschließlich verschiedener Versionen von Linux, FreeBSD und Windows. 

Zur Erinnerung müssen Sie eine gültige Lizenz für alle Betriebssysteme verfügen, die in den virtuellen Computern verwendet. 

Informationen darüber, welche Betriebssysteme als Gäste in Hyper-V unter Windows unterstützt werden, finden Sie unter [Unterstützte Windows-Gastbetriebssysteme](supported-guest-os.md) und [unterstützte Linux Gastbetriebssysteme](https://technet.microsoft.com/library/dn531030.aspx). 


## Unterschiede zwischen Hyper-V unter Windows und Hyper-V unter Windows Server
Es gibt einige Features, die in Hyper-V unter Windows und Hyper-V unter Windows Server unterschiedlich funktionieren.. 

Das Speicher-Management-Modell unterscheidet sich für Hyper-V unter Windows. Hyper-V-Speicher erfolgt auf einem Server unter der Annahme, die nur die virtuellen Maschinen auf dem Server ausgeführt werden. In Hyper-V unter Windows wird Arbeitsspeicher in der Erwartung verwaltet, dass die meisten Clientcomputer sowohl Software auf dem Host als auch virtuelle Computer ausführen. Beispielsweise kann ein Entwickler sowohl Visual Studio als auch mehrere virtuelle Computer auf dem gleichen Computer ausführen.

Einige Features von Hyper-V unter Windows Server sind in Hyper-V unter Windows nicht enthalten. Dazu gehören:

* Virtualisieren von GPUs mit RemoteFX 
* Live-Migration virtueller Maschinen von einem Host zum anderen
* Hyper-V-Replikat
* Virtueller Fibre Channel
* SR-IOV-Netzwerke
* Gemeinsam genutzt. VHDX

## Einschränkungen
Mithilfe der Virtualisierung begrenzt ist. Features oder Anwendungen, die von bestimmter Hardware abhängig sind, funktionieren ebenfalls nicht auf einem virtuellen Computer. Beispielsweise sind Spiele oder Anwendungen, die eine Verarbeitung mit GPUs erfordern, ggf. nicht gut geeignet. Außerdem können bei Anwendungen, die Hochpräzisions-Ereigniszeitgeber im Bereich von weniger als 10ms benötigen (latenzempfindliche Apps, beispielsweise zum Mischen von Livemusik), Probleme auftreten, wenn sie auf einem virtuellen Computer ausgeführt werden.

Darüber hinaus können bei Hyper-V-fähigen Anwendungen mit hohen Anforderungen an Latenz und Präzision auch Probleme bei der Ausführung auf dem Host auftreten.  Dies liegt daran, dass bei virtualisierungsfähigen Anwendungen das Hostbetriebssystem genau wie die Gastbetriebssysteme oberhalb der Hyper-V-Virtualisierungsschicht ausgeführt wird. Im Gegensatz zu Gastbetriebssystemen kann das Hostbetriebssystem direkt auf sämtliche Hardware zugreifen. Dadurch können auch Anwendungen mit speziellen Hardwareanforderungen problemlos im Hostbetriebssystem ausgeführt werden.

## Nächster Schritt
[Installieren von Hyper-V unter Windows10](..\quick-start\enable-hyper-v.md) 
