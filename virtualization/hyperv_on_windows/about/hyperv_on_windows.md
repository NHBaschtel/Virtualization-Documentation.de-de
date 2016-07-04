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
ms.sourcegitcommit: 9f1073ecfa4cf836295de81f2bcf9622274ca034
ms.openlocfilehash: ea19b576219755e09a4064d0fa6bfde5367967e1

---

# Einführung in Hyper-V unter Windows 10

Als Entwickler von Software, IT-Experte oder Technologiefan müssen Sie häufig mehrere Betriebssysteme ausführen, gelegentlich sogar auf vielen verschiedenen Computern. Möglicherweise haben Sie jedoch keinen Zugang zu einer umfassenden Labumgebung, in der all diese Computer gehostet werden könnten. Anstatt nun für jeden einzelnen Computer dedizierte physische Hardwarekomponenten zu verwenden, können Sie die Computer als *virtuelle Hyper-V-Computer* (VMs) ausführen und so die Virtualisierungstechnologie nutzen, um Platz und Zeit zu sparen.

> Microsoft Virtual PC wird im April 2017 eingestellt. Hyper-V unter Windows 10 wird die unterstützte Ablösung. 

## Verwendet für die Virtualisierung
Virtualisierung kann jeder Umgebung mit mehreren Test bestehend aus vielen Betriebssystemen, Software-Konfigurationen und Hardwarekonfigurationen problemlos zu verwalten.  Hyper-V bietet Virtualisierung auf Windows sowie einen einfachen Mechanismus, wechseln Sie schnell zwischen diesen Umgebungen, ohne dass zusätzliche Hardwarekosten anfallen.    

Hyper-V kann zu unterschiedlichen Zwecken verwendet werden. Beispiel:

- Eine Umgebung bestehend aus mehreren virtuellen Computern kann auf einem einzelnen Desktop oder Laptop-Computer erstellt werden. Nachdem die Tests abgeschlossen sind, können diese virtuellen Computer exportiert und anschließend in einem anderen Hyper-V-System importiert.

- Entwickler können Hyper-V auf dem Computer zum Testen von Software auf mehreren Betriebssystemen verwenden. Wenn Sie über eine Anwendung verfügen, die unter Windows 8, Windows 7 und auf einem Linux-Betriebssystem getestet werden muss, können Sie in Ihrem Entwicklungssystem mehrere virtuelle Computer erstellen – eines mit jedem dieser Betriebssysteme.

- Sie können Hyper-V unter Windows 10 verwenden, um Probleme mit virtuellen Computern in jeder Hyper-V-Bereitstellung zu beheben. Sie können exportieren ein virtuellen Computers von der produktionsumgebung, auf dem Desktop mit Hyper-V zu öffnen, führen Sie die Problembehandlung erforderlich und dann wieder in die produktionsumgebung zu exportieren. 

- Mithilfe eines virtuellen Netzwerks können Sie eine Umgebung mit mehreren Computern zu Test-, Entwicklungs- und Demonstrationszwecken erstellen und gleichzeitig sicherstellen, dass das Produktionsnetzwerk nicht beeinflusst wird.

- Technologiefans können Hyper-V zum Experimentieren mit anderen Betriebssystemen verwenden. Hyper-V erleichtert und Beenden von verschiedenen Betriebssystemen.

- Hyper-V können auf einem Laptop für ältere Versionen von Windows oder nicht-Windows-Betriebssystemen zu veranschaulichen. 


## System requirements (Systemanforderungen)
Hyper-V ist ein 64-Bit-System, Second Level Address Translation (SLAT) ist, erforderlich. SLAT ist eine Funktion der aktuellen Generation der 64-Bit-Prozessoren von Intel und AMD. Sie benötigen auch einen 64-Bit-Version von Windows 8 oder höher und mindestens 4 GB RAM. Hyper-V unterstützt die Erstellung von 32-Bit- und 64-Bit-Betriebssysteme auf VMs.

Hyper-V dynamic Memory kann erforderlichen Arbeitsspeichers durch den virtuellen Computer reserviert und dynamisch aufgehoben werden (Sie geben ein Minimum und Maximum) und Freigeben von ungenutztem Speicher zwischen virtuellen Computern. Sie können drei oder vier virtuelle Computer auf einem physischen Computer mit 4 GB RAM ausführen. Für fünf oder mehr virtuelle Computer benötigen Sie jedoch mehr RAM. Am anderen Ende des Spektrums können Sie auch große virtuelle Computer mit 32 Prozessoren und 512 GB RAM, abhängig von der physischen Hardware erstellen.

## Betriebssysteme, die auf einem virtuellen Computer ausgeführt werden kann
Der Begriff "Gast" bezieht sich auf einer virtuellen Maschine und "Host" bezieht sich auf den Computer, auf dem virtuellen Computer ausgeführt wird. Hyper-V unter Windows unterstützt viele verschiedene Gastbetriebssysteme, einschließlich verschiedener Versionen von Linux, FreeBSD und Windows. Informationen darüber, welche Betriebssysteme als Gäste in Hyper-V unter Windows unterstützt werden, finden Sie unter [Unterstützte Windows-Gastbetriebssysteme](supported_guest_os.md) und [Linux- und FreeBSD-VMs unter Hyper-V](https://technet.microsoft.com/library/dn531030.aspx). 

## Unterschiede zwischen Hyper-V auf Windows und Hyper-V auf WindowsServer
Es gibt einige Features, die in Hyper-V unter Windows und Hyper-V unter Windows Server unterschiedlich funktionieren.. Dazu gehören u. a.:

- Das Speicher-Management-Modell unterscheidet sich für Hyper-V unter Windows. Hyper-V-Speicher erfolgt auf einem Server unter der Annahme, die nur die virtuellen Maschinen auf dem Server ausgeführt werden. In Hyper-V unter Windows wird Arbeitsspeicher in der Erwartung verwaltet, dass die meisten Clientcomputer sowohl Software als auch virtuelle Computer ausführen. Beispielsweise kann ein Entwickler Visual Studio als auch mehrere virtuelle Maschinen auf dem gleichen Computer ausgeführt werden.

- SR-IOV auf einem 64-Bit-Gast funktioniert normal, aber die 32-Bit nicht und wird nicht unterstützt.

### In Windows Hyper-V nicht verfügbare Windows Server-Features
Einige Features von Hyper-V unter Windows Server sind in Hyper-V unter Windows nicht enthalten. Dazu gehören u. a.:

- Virtualisieren von GPUs mit RemoteFX 

- Live-Migration virtueller Maschinen von einem Host zum anderen

- Hyper-V-Replikat

- Virtueller Fibre Channel

- SR-IOV-Netzwerke

- Gemeinsam genutzt. VHDX

> **Warnung**: In Hyper-V ausgeführte virtuelle Computer verarbeiten den Wechsel von einer drahtgebundenen zu einer drahtlosen Verbindung nicht automatisch. Sie müssen die Einstellungen des Netzwerkadapters für den virtuellen Computer manuell ändern.

## Einschränkungen
Mithilfe der Virtualisierung begrenzt ist. Features oder Apps, die auf bestimmte Hardware abhängig sind funktioniert ebenfalls auf einem virtuellen Computer nicht. Z. B. Spiele oder Programme, die Verarbeitung mit GPUs erfordern (ohne Software fallback) funktionieren möglicherweise nicht gut. Außerdem können bei Anwendungen, die Hochpräzisions-Ereigniszeitgeber im Bereich von weniger als 10 ms benötigen (latenzempfindliche Apps, beispielsweise zum Mischen von Livemusik), Probleme auftreten, wenn sie auf einem virtuellen Computer ausgeführt werden.

Darüber hinaus können bei virtualisierungsfähigen Anwendungen mit hohen Anforderungen an Latenz und Präzision auch Probleme bei der Ausführung im Hostbetriebssystem auftreten. (Dies liegt daran, dass bei virtualisierungsfähigen Anwendungen das Hostbetriebssystem genau wie die Gastbetriebssysteme oberhalb der Hyper-V-Virtualisierungsschicht ausgeführt wird. Im Gegensatz zu Gastbetriebssystemen kann das Hostbetriebssystem direkt auf sämtliche Hardware zugreifen. Dadurch können Anwendungen mit speziellen Hardwareanforderungen problemlos im Hostbetriebssystem ausgeführt werden.)

Zur Erinnerung müssen Sie eine gültige Lizenz für alle Betriebssysteme verfügen, die in den virtuellen Computern verwendet.

## Nächster Schritt
[Exemplarische Vorgehensweise: Hyper-V unter Windows 10](..\quick_start\walkthrough.md) 

Machen Sie sich mit den [Neuerungen](whats_new.md) in Hyper-V für Windows Server 10 vertraut.




<!--HONumber=Jun16_HO4-->


