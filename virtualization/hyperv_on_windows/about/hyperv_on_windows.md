---
title: "Einführung in Hyper-V unter Windows 10"
description: "Einführung in Hyper-V unter Windows 10."
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
translationtype: Human Translation
ms.sourcegitcommit: 53539a0325b3f07e542ca1dd0a4352239e8a65b3
ms.openlocfilehash: 25295b8a2888e25090439a3490c9ff7c3214f23a

---

# Einführung in Hyper-V unter Windows 10

Als Entwickler von Software, IT-Experte oder Technologiefan müssen Sie häufig mehrere Betriebssysteme ausführen.  Anstatt jedem Ihrer Computer physische Hardware dediziert zuzuweisen, ermöglicht Ihnen Hyper-V das Ausführen mehrerer Betriebssysteme auf Ihrem Windows-Computer als virtuelle Computer (virtuelle Maschinen, VMs).

> Microsoft Virtual PC wird im April 2017 eingestellt. Hyper-V unter Windows 10 Enterprise und Windows 10 Professional ist der unterstützte Nachfolger.  

## Gründe für die Virtualisierung
Virtualisierung ermöglicht allen Benutzern das Ausführen mehrerer Betriebssysteme, Software- und Hardwarekonfigurationen auf demselben physischen Computer.  Hyper-V bietet sowohl die Virtualisierung als auch die Tools zum Verwalten Ihrer virtuellen Computer.

Hyper-V kann zu unterschiedlichen Zwecken verwendet werden. Beispiel:

* Führen Sie Software aus, die eine ältere Version von Windows oder andere Betriebssysteme als Windows erfordert. 

* Experimentieren Sie mit anderen Betriebssystemen. Hyper-V erleichtert das Inbetriebnehmen und Entfernen verschiedener Betriebssysteme.

* Testen Sie Software unter mehreren Betriebssystemen mithilfe mehrerer virtueller Computer. Mit Hyper-V können Sie alle auf einem einzelnen Desktop- oder Laptopcomputer ausführen. Diese virtuellen Computer können exportiert und anschließend in ein anderes Hyper-V-System, einschließlich Azure, importiert werden.

* Sie können Probleme mit virtuellen Computern in einer beliebigen Hyper-V-Umgebung beheben. Sie können einen virtuellen Computer aus Ihrer Produktionsumgebung exportieren, auf Ihrem Desktopcomputer, auf dem Hyper-V ausgeführt wird, öffnen, die Probleme des virtuellen Computers beheben und ihn dann zurück in die Produktionsumgebung exportieren. 

* Mithilfe eines virtuellen Netzwerks können Sie eine Umgebung mit mehreren Computern zu Test-, Entwicklungs- und Demonstrationszwecken erstellen und gleichzeitig sicherstellen, dass das Produktionsnetzwerk nicht beeinflusst wird.

## System requirements (Systemanforderungen)
Hyper-V ist nur in den Editionen Professional, Enterprise, und Education von Windows 8 und höher verfügbar.

Hyper-V erfordert ein 64-Bit-System mit Second Level Address Translation (SLAT). SLAT ist eine Funktion der aktuellen Generation der 64-Bit-Prozessoren von Intel und AMD.  Sie benötigen auch eine 64-Bit-Version von Windows.  
Allerdings unterstützt Hyper-V sowohl 32-Bit- als auch 64-Bit-Betriebssysteme innerhalb der virtuellen Computer.

Auf einem Host mit 4 GB RAM können Sie drei bis vier virtuelle Computer ausführen. Für weitere virtuelle Computer benötigen Sie allerdings weitere Ressourcen. Am anderen Ende des Spektrums können Sie auch, abhängig von der physischen Hardware, große virtuelle Computer mit 32 Prozessoren und 512 GB RAM erstellen.

Weitere Informationen zu den Systemanforderungen von Hyper-V und zum Überprüfen, ob Hyper-V auf Ihrem Computer ausgeführt wird, finden Sie unter [Exemplarische Vorgehensweise: Systemanforderungen von Hyper-V unter Windows 10](..\quick_start\walkthrough_install.md).


## Betriebssysteme, die auf einem virtuellen Computer ausgeführt werden kann
Der Begriff "Gast" bezieht sich auf einer virtuellen Maschine und "Host" bezieht sich auf den Computer, auf dem virtuellen Computer ausgeführt wird. Hyper-V unter Windows unterstützt viele verschiedene Gastbetriebssysteme, einschließlich verschiedener Versionen von Linux, FreeBSD und Windows. 

Zur Erinnerung müssen Sie eine gültige Lizenz für alle Betriebssysteme verfügen, die in den virtuellen Computern verwendet. 

Informationen darüber, welche Betriebssysteme als Gäste in Hyper-V unter Windows unterstützt werden, finden Sie unter [Unterstützte Windows-Gastbetriebssysteme](supported_guest_os.md) und [Linux- und FreeBSD-VMs unter Hyper-V](https://technet.microsoft.com/library/dn531030.aspx). 


## Unterschiede zwischen Hyper-V auf Windows und Hyper-V auf WindowsServer
Es gibt einige Features, die in Hyper-V unter Windows und Hyper-V unter Windows Server unterschiedlich funktionieren.. 

Das Speicher-Management-Modell unterscheidet sich für Hyper-V unter Windows. Hyper-V-Speicher erfolgt auf einem Server unter der Annahme, die nur die virtuellen Maschinen auf dem Server ausgeführt werden. In Hyper-V unter Windows wird Arbeitsspeicher in der Erwartung verwaltet, dass die meisten Clientcomputer sowohl Software auf dem Host als auch virtuelle Computer ausführen. Beispielsweise kann ein Entwickler Visual Studio als auch mehrere virtuelle Maschinen auf dem gleichen Computer ausgeführt werden.

### Nur in Windows Server verfügbare Hyper-V-Features
Einige Features von Hyper-V unter Windows Server sind in Hyper-V unter Windows nicht enthalten. Dazu gehören:

* Virtualisieren von GPUs mit RemoteFX 
* Live-Migration virtueller Maschinen von einem Host zum anderen
* Hyper-V-Replikat
* Virtueller Fibre Channel
* SR-IOV-Netzwerke
* Gemeinsam genutzt. VHDX

## Einschränkungen
Mithilfe der Virtualisierung begrenzt ist. Features oder Anwendungen, die von bestimmter Hardware abhängig sind, funktionieren ebenfalls nicht auf einem virtuellen Computer. Beispielsweise sind Spiele oder Anwendungen, die eine Verarbeitung mit GPUs erfordern, ggf. nicht gut geeignet. Außerdem können bei Anwendungen, die Hochpräzisions-Ereigniszeitgeber im Bereich von weniger als 10 ms benötigen (latenzempfindliche Apps, beispielsweise zum Mischen von Livemusik), Probleme auftreten, wenn sie auf einem virtuellen Computer ausgeführt werden.

Darüber hinaus können bei Hyper-V-fähigen Anwendungen mit hohen Anforderungen an Latenz und Präzision auch Probleme bei der Ausführung auf dem Host auftreten.  Dies liegt daran, dass bei virtualisierungsfähigen Anwendungen das Hostbetriebssystem genau wie die Gastbetriebssysteme oberhalb der Hyper-V-Virtualisierungsschicht ausgeführt wird. Im Gegensatz zu Gastbetriebssystemen kann das Hostbetriebssystem direkt auf sämtliche Hardware zugreifen. Dadurch können Anwendungen mit speziellen Hardwareanforderungen problemlos im Hostbetriebssystem ausgeführt werden.

## Nächster Schritt
[Exemplarische Vorgehensweise: Installieren von Hyper-V unter Windows 10](..\quick_start\walkthrough_install.md) 

Machen Sie sich mit den [Neuerungen](whats_new.md) in Hyper-V für Windows Server 10 vertraut.




<!--HONumber=Jul16_HO2-->


