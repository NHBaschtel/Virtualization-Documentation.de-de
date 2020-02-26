---
title: Hyper-V-Integrationsdienste
description: Verweis für Hyper-V-Integrationsdienste
keywords: Windows 10, Hyper-V, Integrationsdienste, Integrationskomponenten
author: scooley
ms.date: 05/25/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 18930864-476a-40db-aa21-b03dfb4fda98
ms.openlocfilehash: 6568b68a77fc5506b58249caea44ec78e3e44de2
ms.sourcegitcommit: 16744984ede5ec94cd265b6bff20aee2f782ca88
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 02/18/2020
ms.locfileid: "77439567"
---
# <a name="hyper-v-integration-services"></a>Hyper-V-Integrationsdienste

Integrationsdienste (oft als „Integrationskomponenten“ bezeichnet) ermöglichen dem virtuellen Computer das Kommunizieren mit dem Hyper-V-Host. Viele dieser Dienste stellen Vorteile dar, während andere sehr wichtig für die problemlose Funktion des virtuellen Computers sein können.

Dieser Artikel stellt eine Referenz für jeden in Windows verfügbaren Integrationsdienst dar.  Er fungiert auch als Ausgangspunkt für alle Informationen im Zusammenhang mit bestimmten Integrationsdiensten oder ihren Verläufen.

**Benutzerhandbücher:**  
* [Verwalten von Integrationsdiensten](https://docs.microsoft.com/windows-server/virtualization/hyper-v/manage/Manage-Hyper-V-integration-services)


## <a name="quick-reference"></a>Kurzreferenz

| Name | Name des Windows-Diensts | Name des Linux-Daemons |  Beschreibung | Auswirkung auf die VM, wenn deaktiviert |
|:---------|:---------|:---------|:---------|:---------|
| [Hyper-V-Taktdienst](#hyper-v-heartbeat-service) |  vmicheartbeat | hv_utils | Berichtet, dass der virtuelle Computer fehlerfrei ausgeführt wird. | Variiert |
| [Hyper-V-Dienst zum Herunterfahren des Gasts](#hyper-v-guest-shutdown-service) | vmicshutdown | hv_utils |  Ermöglicht es dem Host, das Herunterfahren des virtuellen Computers auszulösen. | **Hochrangiger** |
| [Hyper-V-Dienst für Zeitsynchronisierung](#hyper-v-time-synchronization-service) | vmictimesync | hv_utils | Synchronisiert die Uhr des virtuellen Computers mit der Uhr des Hostcomputers. | **Hochrangiger** |
| [Hyper-V-Datenaustausch Dienst (KVP)](#hyper-v-data-exchange-service-kvp) | vmickvpexchange | hv_kvp_daemon | Bietet ein Verfahren zum Austausch von grundlegenden Metadaten zwischen dem virtuellen Computer und dem Host. | Mittel |
| [Hyper-V-Volumeschattenkopie-Anforderer](#hyper-v-volume-shadow-copy-requestor) | vmicvss | hv_vss_daemon | Ermöglicht dem Volumeschattenkopie-Dienst, den virtuellen Computer zu sichern, ohne ihn herunterzufahren. | Variiert |
| [Hyper-V-Gastschnittstelle](#hyper-v-powershell-direct-service) | vmicguestinterface | hv_fcopy_daemon | Stellt eine Schnittstelle für den Hyper-V-Host zum Kopieren von Dateien zu oder von dem virtuellen Computer bereit. | Niedrig |
| [Hyper-V PowerShell Direct Service](#hyper-v-powershell-direct-service) | vmicvmsession | nicht verfügbar | Bietet eine Möglichkeit zum Verwalten der virtuellen Computer mit PowerShell ohne eine Netzwerkverbindung. | Niedrig |  


## <a name="hyper-v-heartbeat-service"></a>Hyper-V-Taktdienst

**Name des Windows-Diensts:** vmicheartbeat  
**Name des Linux-Daemons:** hv_utils  
**Beschreibung:** teilt dem Hyper-V-Host mit, dass auf dem virtuellen Computer ein Betriebssystem installiert ist und dass er ordnungsgemäß gestartet wurde.  
**Hinzugefügt in:** Windows Server 2012, Windows 8  
**Auswirkung:** Wenn er deaktiviert ist, ist der virtuelle Computer nicht in der Lage zu berichten, dass das Betriebssystem im virtuellen Computer ordnungsgemäß ausgeführt wird.  Das kann Auswirkungen auf einige Überwachungsdiagnosen und Diagnosen aufseiten des Hosts haben.  

Der Taktdienst ermöglicht die Beantwortung grundlegender Fragen wie „Wurde der virtuelle Computer gestartet?“.  

Wenn Hyper-V berichtet, dass der Status eines virtuellen Computers „wird ausgeführt“ lautet (siehe nachfolgendes Beispiel), bedeutet dies, dass Hyper-V Ressourcen für den virtuellen Computer reserviert hat. Es bedeutet nicht, dass ein Betriebssystem installiert wurde oder funktioniert.  Hier ist der Takt hilfreich.  Der Taktdienst berichtet Hyper-V, dass das Betriebssystem im virtuellen Computer gestartet wurde.  

### <a name="check-heartbeat-with-powershell"></a>Überprüfen des Takts mit PowerShell

Führen Sie [Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps) als Administrator aus, um den Takt des virtuellen Computers zu sehen:
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



## <a name="hyper-v-guest-shutdown-service"></a>Hyper-V-Dienst zum Herunterfahren des Gasts

**Name des Windows-Diensts:** vmicshutdown  
**Name des Linux-Daemons:** hv_utils  
**Beschreibung:** ermöglicht dem Hyper-V-Host, das Herunterfahren des virtuellen Computers anzufordern.  Der Host kann die Abschaltung des virtuellen Computers immer erzwingen, was aber aber mehr dem Ziehen des Netzsteckers als dem normalen Herunterfahren entspricht.  
**Hinzugefügt in:** Windows Server 2012, Windows 8  
**Auswirkung:** **Wenn diese** Option deaktiviert ist, kann der Host kein freundliches Herunterfahren innerhalb des virtuellen Computers auslöst.  Alle Herunterfahr Vorgänge sind eine harte Stromversorgung, was zu Datenverlusten oder Daten Beschädigungen führen kann.  


## <a name="hyper-v-time-synchronization-service"></a>Hyper-V-Dienst für Zeitsynchronisierung

**Name des Windows-Diensts:** vmictimesync  
**Name des Linux-Daemons:** hv_utils  
**Beschreibung:** synchronisiert die Systemuhr des virtuellen Computers mit der Systemuhr des physischen Computers.  
**Hinzugefügt in:** Windows Server 2012, Windows 8  
**Auswirkung:** **hohe Beeinträchtigung** , wenn diese deaktiviert ist, wird die Uhr des virtuellen Computers nicht mehr abweichen.  


## <a name="hyper-v-data-exchange-service-kvp"></a>Hyper-V-Datenaustauschdienst (KVP)

**Name des Windows-Diensts:** vmickvpexchange  
**Name des Linux-Daemons:** hv_kvp_daemon  
**Beschreibung:** bietet einen Mechanismus zum Austausch von grundlegenden Metadaten zwischen dem virtuellen Computer und dem Host.  
**Hinzugefügt in:** Windows Server 2012, Windows 8  
**Auswirkung:** Falls deaktiviert, erhalten virtuelle Computer, auf denen Windows 8 oder Windows Server 2012 oder früher ausgeführt wird, keine Updates zu Hyper-V-Integrationsdiensten.  Das Deaktivieren des Datenaustauschs kann auch Auswirkungen auf einige Überwachungsdiagnosen und Diagnosen aufseiten des Hosts haben.  

Der Datenaustauschdienst (manchmal auch als KVP bezeichnet) gibt kleine Mengen Computerinformationen zwischen virtuellen Computern und dem Hyper-V-Host mithilfe von Schlüssel-Wert-Paaren (key-value pairs; KVP) durch die Windows-Registrierung frei.  Der gleiche Mechanismus kann auch verwendet werden, um benutzerdefinierte Daten zwischen dem virtuellen Computer und dem Host freizugeben.

Schlüssel-Wert-Paare bestehen aus einem „Schlüssel“ und einem „Wert“. Sowohl der Schlüssel als auch der Wert sind Zeichenfolgen, es werden keine weiteren Datentypen unterstützt. Wenn ein Schlüssel-Wert-Paar erstellt oder geändert wird, ist das für den Gast und Host sichtbar. Die Informationen des Schlüssel-Wert-Paars werden über den Hyper-V-VMBus übertragen und benötigen keinerlei Netzwerkverbindung zwischen dem Gast und dem Hyper-V-Host. 

Der Datenaustauschdienst ist ein großartiges Tool, um Informationen zum virtuellen Computer zu sichern. Verwenden Sie für interaktive Datenfreigaben oder Datenübertragungen [PowerShell Direct](#hyper-v-powershell-direct-service). 


**Benutzerhandbücher:**  
* [Freigeben von Informationen zwischen dem Host und dem Gast auf Hyper-V mithilfe von Schlüssel-Wert-Paaren](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn798287(v=ws.11)).  


## <a name="hyper-v-volume-shadow-copy-requestor"></a>Hyper-V-Volumeschattenkopie-Anforderer

**Name des Windows-Diensts:** vmicvss  
**Name des Linux-Daemons:** hv_vss_daemon  
**Beschreibung:** ermöglicht dem Volumeschattenkopie-Dienst, Anwendungen und Daten auf dem virtuellen Computer zu sichern.  
**Hinzugefügt in:** Windows Server 2012, Windows 8  
**Auswirkung:** Wenn der Dienst deaktiviert ist, ist es nicht möglich den virtuellen Computer zu sichern, während er ausgeführt wird (mithilfe von VSS).  

Der Integrationsdienst für den Volumeschattenkopie-Anforderer wird für den Volumeschattenkopie-Dienst (Volume Shadow Copy Service; ([VSS](https://docs.microsoft.com/windows/desktop/VSS/overview-of-processing-a-backup-under-vss)) benötigt).  Der Volumeschattenkopie-Dienst (Volume Shadow Copy Service; VSS) erfasst und kopiert Images für die Sicherung der laufenden Systeme, insbesondere der Server, ohne dass die Leistung und Stabilität der von ihnen gebotenen Dienste übermäßig beeinträchtigt werden.  Dieser Integrationsdienst ermöglicht dies durch die Koordination der Arbeitsauslastungen des virtuellen Computers mit dem Sicherungsprozess des Hosts.

Weitere Informationen zur Volumeschattenkopie erhalten Sie [hier](https://docs.microsoft.com/previous-versions/windows/desktop/virtual/backing-up-and-restoring-virtual-machines).


## <a name="hyper-v-guest-service-interface"></a>Hyper-V-Gastschnittstelle

**Name des Windows-Diensts:** vmicguestinterface  
**Name des Linux-Daemons:** hv_fcopy_daemon  
**Beschreibung:** stellt eine Schnittstelle für den Hyper-V-Host zum Kopieren von Dateien zu oder vom virtuellen Computer in beide Richtungen bereit.  
**Hinzugefügt in:** Windows Server 2012 R2, Windows 8.1  
**Auswirkung:** Wenn der Dienst deaktiviert ist, kann der Host keine Dateien zum oder vom Gast mithilfe von `Copy-VMFile` kopieren.  Erfahren Sie mehr über das [Cmdlet Copy-VMFile](https://docs.microsoft.com/powershell/module/hyper-v/copy-vmfile?view=win10-ps).  

**Anmerkungen:**  
Standardmäßig deaktiviert  Weitere Informationen finden Sie unter [PowerShell Direct mithilfe von Copy-Item](../user-guide/powershell-direct.md#copy-files-with-new-pssession-and-copy-item). 


## <a name="hyper-v-powershell-direct-service"></a>Hyper-V-Dienst PowerShell Direct

**Name des Windows-Diensts:** vmicvmsession  
**Name des Linux-Daemons:** nicht zutreffend  
**Beschreibung:** bietet einen Mechanismus zum Verwalten eines virtuellen Computers mit PowerShell über VM-Sitzungen ohne ein virtuelles Netzwerk.    
**Hinzugefügt in:** Windows Server TP3, Windows 10  
**Auswirkung:** Das Deaktivieren dieses Diensts verhindert, dass der Host mittels PowerShell Direct eine Verbindung mit dem virtuellen Computer herstellen kann.  

**Anmerkungen:**  
Der Name des Dienstes war ursprünglich „Hyper-V-VM-Sitzungsdienst“.  
PowerShell Direct befindet sich in der aktiven Entwicklung und ist nur auf Windows 10/Windows Server Technical Preview 3 oder späteren Hosts/Gästen verfügbar.

PowerShell Direct ermöglicht die Verwaltung von PowerShell innerhalb eines virtuellen Computers vom Hyper-V-Host aus, unabhängig von der Netzwerkkonfiguration oder den Remoteverwaltungseinstellungen auf dem Hyper-V-Host oder dem virtuellen Computer. Dies erleichtert Hyper-V-Administratoren die Automatisierung und Skripterstellung für Verwaltungs- und Konfigurationsaufgaben.

[Weitere Informationen zu PowerShell Direct](../user-guide/powershell-direct.md).  

**Benutzerhandbücher:**  
* [Ausführen eines Skripts auf einem virtuellen Computer](../user-guide/powershell-direct.md#run-a-script-or-command-with-invoke-command)
* [Kopieren von Dateien in und von einer virtuellen Maschine](../user-guide/powershell-direct.md#copy-files-with-new-pssession-and-copy-item)
