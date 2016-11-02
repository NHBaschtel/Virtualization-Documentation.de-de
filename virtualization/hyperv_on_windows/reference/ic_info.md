---
title: Hyper-V-Integrationsdienste
description: "Verweis für Hyper-V-Integrationsdienste"
keywords: Windows 10, Hyper-V, Integrationsdienste, Integrationskomponenten
author: scooley
ms.date: 05/25/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 18930864-476a-40db-aa21-b03dfb4fda98
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: 71ec8f80e721a0c66dc8709473a427792df1cf24

---

# Hyper-V-Integrationsdienste

Integrationsdienste (oft als „Integrationskomponenten“ bezeichnet) ermöglichen dem virtuellen Computer das Kommunizieren mit dem Hyper-V-Host. Viele dieser Dienste stellen Vorteile dar, während andere sehr wichtig für die problemlose Funktion des virtuellen Computers sein können.

Dieser Artikel stellt eine Referenz für jeden in Windows verfügbaren Integrationsdienst dar.  Er fungiert auch als Ausgangspunkt für alle Informationen im Zusammenhang mit bestimmten Integrationsdiensten oder ihren Verläufen.

**Benutzerhandbücher:**  
* [Aktivieren/Deaktivieren von Integrationsdiensten vom Hyper-V-Host aus](../user_guide/managing_ics.md#enable-or-disable-integration-services-using-powershell)
* Aktivieren/Deaktivieren von Integrationsdiensten im virtuellen Computer
  * [Windows](../user_guide/managing_ics.md#manage-integration-services-from-guest-os-windows)
  * [Linux](../user_guide/managing_ics.md#manage-integration-services-from-guest-os-linux)
* [Updates und Wartung von Integrationsdiensten](../user_guide/managing_ics.md#integration-service-maintenance)


## Kurzübersicht

| Name | Name des Windows-Diensts | Name des Linux-Daemons |  Beschreibung | Auswirkung auf die VM, wenn deaktiviert |
|:---------|:---------|:---------|:---------|:---------|
| [Hyper-V Taktdienst](#hyper-v-heartbeat-service) |  vmicheartbeat | hv_utils | Berichtet, dass der virtuelle Computer fehlerfrei ausgeführt wird. | Variiert |
| [Hyper-V-Dienst zum Herunterfahren des Gasts](#hyper-v-guest-shutdown-service) | vmicshutdown | hv_utils |  Ermöglicht es dem Host, das Herunterfahren des virtuellen Computers auszulösen. | **Hoch** |
| [Hyper-V-Zeitsynchronisierungsdienst](#hyper-v-time-synchronization-service) | vmictimesync | hv_utils | Synchronisiert die Uhr des virtuellen Computers mit der Uhr des Hostcomputers. | **Hoch** |
| [Hyper-V-Datenaustauschdienst (KVP)](#hyper-v-data-exchange-service-kvp) | vmickvpexchange | hv_kvp_daemon | Bietet ein Verfahren zum Austausch von grundlegenden Metadaten zwischen dem virtuellen Computer und dem Host. | Mittel |
| [Hyper-V-Volumeschattenkopie-Anforderer](#hyper-v-volume-shadow-copy-requestor) | vmicvss | hv_vss_daemon | Ermöglicht dem Volumeschattenkopie-Dienst, den virtuellen Computer zu sichern, ohne ihn herunterzufahren. | Variiert |
| [Hyper-V-Gastdienstschnittstelle](#hyper-v-powershell-direct-service) | vmicguestinterface | hv_fcopy_daemon | Stellt eine Schnittstelle für den Hyper-V-Host zum Kopieren von Dateien zu oder von dem virtuellen Computer bereit. | Niedrig |
| [Hyper-V-Dienst PowerShell Direct](#hyper-v-powershell-direct-service) | vmicvmsession | nicht verfügbar | Bietet eine Möglichkeit zum Verwalten der virtuellen Computer mit PowerShell ohne eine Netzwerkverbindung. | Niedrig |  


## Hyper-V Taktdienst

**Name des Windows-Diensts:** vmicheartbeat  
**Name des Linux-Daemons:** hv_utils  
**Beschreibung:** teilt dem Hyper-V-Host mit, dass auf dem virtuellen Computer ein Betriebssystem installiert ist und dass er ordnungsgemäß gestartet wurde.  
**Hinzugefügt in:** Windows Server 2012, Windows 8  
**Auswirkung:** Wenn er deaktiviert ist, ist der virtuelle Computer nicht in der Lage zu berichten, dass das Betriebssystem im virtuellen Computer ordnungsgemäß ausgeführt wird.  Das kann Auswirkungen auf einige Überwachungsdiagnosen und Diagnosen aufseiten des Hosts haben.  

Der Taktdienst ermöglicht die Beantwortung grundlegender Fragen wie „Wurde der virtuelle Computer gestartet?“.  

Wenn Hyper-V berichtet, dass der Status eines virtuellen Computers „wird ausgeführt“ lautet (siehe nachfolgendes Beispiel), bedeutet dies, dass Hyper-V Ressourcen für den virtuellen Computer reserviert hat. Es bedeutet nicht, dass ein Betriebssystem installiert wurde oder funktioniert.  Hier ist der Takt hilfreich.  Der Taktdienst berichtet Hyper-V, dass das Betriebssystem im virtuellen Computer gestartet wurde.  

### Überprüfen des Takts mit PowerShell

Führen Sie [Get-VM](https://technet.microsoft.com/en-us/library/hh848479.aspx) als Administrator aus, um den Takt des virtuellen Computers zu sehen:
``` PowerShell
Get-VM -VMName $VMName | select Name, State, Status
```

Die Ausgabe sollte etwa wie folgt aussehen:
```
Name    State    Status
----    -----    ------
DemoVM  Running  Operating normally
```

Das Feld `Status` wird vom Taktdienst bestimmt.



## Hyper-V-Dienst zum Herunterfahren des Gasts

**Name des Windows-Diensts:** vmicshutdown  
**Name des Linux-Daemons:** hv_utils  
**Beschreibung:** ermöglicht dem Hyper-V-Host, das Herunterfahren des virtuellen Computers anzufordern.  Der Host kann die Abschaltung des virtuellen Computers immer erzwingen, was aber aber mehr dem Ziehen des Netzsteckers als dem normalen Herunterfahren entspricht.  
**Hinzugefügt in:** Windows Server 2012, Windows 8  
**Auswirkung:** **Gravierende Auswirkungen** Wenn der Dienst deaktiviert ist, kann der Host kein normales Herunterfahren im virtuellen Computer auslösen.  Alle Herunterfahrvorgänge sind erzwungene Systemabschaltungen, was zu Datenverlusten oder Datenbeschädigungen führen kann.  


## Hyper-V-Zeitsynchronisierungsdienst

**Name des Windows-Diensts:** vmictimesync  
**Name des Linux-Daemons:** hv_utils  
**Beschreibung:** synchronisiert die Systemuhr des virtuellen Computers mit der Systemuhr des physischen Computers.  
**Hinzugefügt in:** Windows Server 2012, Windows 8  
**Auswirkung:** **Gravierende Auswirkungen** Wenn der Dienst deaktiviert ist, weicht die Uhr des virtuellen Computers unkontrolliert ab.  


## Hyper-V-Datenaustauschdienst (KVP)

**Name des Windows-Diensts:** vmickvpexchange  
**Name des Linux-Daemons:** hv_kvp_daemon  
**Beschreibung:** bietet einen Mechanismus zum Austausch von grundlegenden Metadaten zwischen dem virtuellen Computer und dem Host.  
**Hinzugefügt in:** Windows Server 2012, Windows 8  
**Auswirkung:** Falls deaktiviert, erhalten virtuelle Computer, auf denen Windows 8 oder Windows Server 2012 oder früher ausgeführt wird, keine Updates zu Hyper-V-Integrationsdiensten.  Das Deaktivieren des Datenaustauschs kann auch Auswirkungen auf einige Überwachungsdiagnosen und Diagnosen aufseiten des Hosts haben.  

Der Datenaustauschdienst (manchmal auch als KVP bezeichnet) gibt kleine Mengen Computerinformationen zwischen virtuellen Computern und dem Hyper-V-Host mithilfe von Schlüssel-Wert-Paaren (key-value pairs; KVP) durch die Windows-Registrierung frei.  Der gleiche Mechanismus kann auch verwendet werden, um benutzerdefinierte Daten zwischen dem virtuellen Computer und dem Host freizugeben.

Schlüssel-Wert-Paare bestehen aus einem „Schlüssel“ und einem „Wert“. Sowohl der Schlüssel als auch der Wert sind Zeichenfolgen, es werden keine weiteren Datentypen unterstützt. Wenn ein Schlüssel-Wert-Paar erstellt oder geändert wird, ist das für den Gast und Host sichtbar. Die Informationen des Schlüssel-Wert-Paars werden über den Hyper-V-VMBus übertragen und benötigen keinerlei Netzwerkverbindung zwischen dem Gast und dem Hyper-V-Host. 

Der Datenaustauschdienst ist ein großartiges Tool, um Informationen zum virtuellen Computer zu sichern. Verwenden Sie für interaktive Datenfreigaben oder Datenübertragungen [PowerShell Direct](#hyper-v-powershell-direct-service). 


**Benutzerhandbücher:**  
* [Freigeben von Informationen zwischen dem Host und dem Gast auf Hyper-V mithilfe von Schlüssel-Wert-Paaren](https://technet.microsoft.com/en-us/library/dn798287.aspx).  


## Hyper-V-Volumeschattenkopie-Anforderer

**Name des Windows-Diensts:** vmicvss  
**Name des Linux-Daemons:** hv_vss_daemon  
**Beschreibung:** ermöglicht dem Volumeschattenkopie-Dienst, Anwendungen und Daten auf dem virtuellen Computer zu sichern.  
**Hinzugefügt in:** Windows Server 2012, Windows 8  
**Auswirkung:** Wenn der Dienst deaktiviert ist, ist es nicht möglich den virtuellen Computer zu sichern, während er ausgeführt wird (mithilfe von VSS).  

Der Integrationsdienst für den Volumeschattenkopie-Anforderer wird für den Volumeschattenkopie-Dienst (Volume Shadow Copy Service; ([VSS](https://msdn.microsoft.com/en-us/library/aa384589.aspx)) benötigt).  Der Volumeschattenkopie-Dienst (Volume Shadow Copy Service; VSS) erfasst und kopiert Images für die Sicherung der laufenden Systeme, insbesondere der Server, ohne dass die Leistung und Stabilität der von ihnen gebotenen Dienste übermäßig beeinträchtigt werden.  Dieser Integrationsdienst ermöglicht dies durch die Koordination der Arbeitsauslastungen des virtuellen Computers mit dem Sicherungsprozess des Hosts.

Weitere Informationen zur Volumeschattenkopie erhalten Sie [hier](https://msdn.microsoft.com/en-us/library/dd405549.aspx).


## Hyper-V-Gastdienstschnittstelle

**Name des Windows-Diensts:** vmicguestinterface  
**Name des Linux-Daemons:** hv_fcopy_daemon  
**Beschreibung:** stellt eine Schnittstelle für den Hyper-V-Host zum Kopieren von Dateien zu oder vom virtuellen Computer in beide Richtungen bereit.  
**Hinzugefügt in:** Windows Server 2012 R2, Windows 8.1  
**Auswirkung:** Wenn der Dienst deaktiviert ist, kann der Host keine Dateien zum oder vom Gast mithilfe von `Copy-VMFile` kopieren.  Erfahren Sie mehr über das [Cmdlet Copy-VMFile](https://technet.microsoft.com/library/dn464282.aspx).  

**Hinweise:**  
Standardmäßig deaktiviert  Weitere Informationen finden Sie unter [PowerShell Direct mithilfe von Copy-Item](../user_guide/vmsession.md#copy-files-with-new-pssession-and-copy-item). 


## Hyper-V-Dienst PowerShell Direct

**Name des Windows-Diensts:** vmicvmsession  
**Name des Linux-Daemons:** nicht zutreffend  
**Beschreibung:** bietet einen Mechanismus zum Verwalten eines virtuellen Computers mit PowerShell über VM-Sitzungen ohne ein virtuelles Netzwerk.    
**Hinzugefügt in:** Windows Server TP3, Windows 10  
**Auswirkung:** Das Deaktivieren dieses Diensts verhindert, dass der Host mittels PowerShell Direct eine Verbindung mit dem virtuellen Computer herstellen kann.  

**Hinweise:**  
Der Name des Dienstes war ursprünglich „Hyper-V-VM-Sitzungsdienst“.  
PowerShell Direct befindet sich in der aktiven Entwicklung und ist nur auf Windows 10/Windows Server Technical Preview 3 oder späteren Hosts/Gästen verfügbar.  


PowerShell Direct ermöglicht die Verwaltung von PowerShell innerhalb eines virtuellen Computers vom Hyper-V-Host aus, unabhängig von der Netzwerkkonfiguration oder den Remoteverwaltungseinstellungen auf dem Hyper-V-Host oder dem virtuellen Computer. Dies erleichtert Hyper-V-Administratoren die Automatisierung und Skripterstellung für Verwaltungs- und Konfigurationsaufgaben.

[Weitere Informationen zu PowerShell Direct](../user_guide/vmsession.md).  

**Benutzerhandbücher:**  
* [Ausführen eines Skripts in einem virtuellen Computer](../user_guide/vmsession.md#run-a-script-or-command-with-invoke-command)
* [Kopieren von Dateien zu und aus einem virtuellen Computer](../user_guide/vmsession.md#copy-files-with-new-pssession-and-copy-item)



<!--HONumber=Oct16_HO4-->


