---
title: Verwalten von Hyper-V-Integrationsdiensten
description: Verwalten von Hyper-V-Integrationsdiensten
keywords: Windows 10, Hyper-V, Integrationsdienste, Integrationskomponenten
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 9cafd6cb-dbbe-4b91-b26c-dee1c18fd8c2
redirect_url: https://technet.microsoft.com/windows-server-docs/compute/hyper-v/manage/manage-Hyper-V-integration-services
ms.openlocfilehash: 83bcc4c2f47e2a3921be257f45a3a0e22dcba89a
ms.sourcegitcommit: fd6c5ec419aae425af7ce6c6a44d59c98f62502a
ms.translationtype: HT
ms.contentlocale: de-DE
---
# <a name="managing-hyper-v-integration-services"></a>Verwalten von Hyper-V-Integrationsdiensten

Integrationsdienste (oft als „Integrationskomponenten“ bezeichnet) ermöglichen dem virtuellen Computer das Kommunizieren mit dem Hyper-V-Host. Viele dieser Dienste stellen Vorteile dar (z.B. Kopieren von Gastdateien), während andere für das problemlose Funktionieren des virtuellen Computers recht wichtig sein können (z.B. Zeitsynchronisierung).

Dieser Artikel enthält ausführliche Informationen zum Verwalten von Integrationsdiensten mithilfe von Hyper-V-Manager und PowerShell in Windows 10.  

Weitere Informationen zu den einzelnen Integrationsdiensten finden Sie unter [Integrationsdienste](../reference/integration-services.md).

## <a name="enable-or-disable-integration-services-using-hyper-v-manager"></a>Aktivieren und Deaktivieren von Integrationsdiensten mit Hyper-V-Manager

1. Wählen Sie einen virtuellen Computer aus, und öffnen Sie „Einstellungen“.
  
2. Wechseln Sie im Fenster „Einstellungen“ des virtuellen Computers unter „Verwaltung“ zur Registerkarte „Integrationsdienste“.
  
  Hier sehen Sie alle Integrationsdienste, die auf diesem Hyper-V-Host verfügbar sind.  Wichtig ist der Hinweis, dass das Gastbetriebssystem ggf. nicht alle aufgeführten Integrationsdienste unterstützt. Um die Versionsinformationen eines Gastbetriebssystems zu ermitteln, melden Sie sich am Gastbetriebssystem an und führen an der Eingabeaufforderung den folgenden Befehl aus.

REG QUERY "HKLM\Software\Microsoft\Virtual Machine\Auto" /v IntegrationServicesVersion

## <a name="enable-or-disable-integration-services-using-powershell"></a>Aktivieren und Deaktivieren von Integrationsdiensten mit PowerShell

Integrationsdienste können auch mit PowerShell durch Ausführen von [`Enable-VMIntegrationService`](https://technet.microsoft.com/en-us/library/hh848500.aspx) bzw. [`Disable-VMIntegrationService`](https://technet.microsoft.com/en-us/library/hh848488.aspx) aktiviert und deaktiviert werden.

In diesem Beispiel aktivieren und deaktivieren wir den Integrationsdienst für das Kopieren von Gastdateien auf dem zuvor erwähnten virtuellen Computer „demovm“.

1. Prüfen, welche Integrationsdienste ausgeführt werden
  
  ``` PowerShell
  Get-VMIntegrationService -VMName "DemoVM"
  ```

  Die Ausgabe sieht folgendermaßen aus:  
  ``` PowerShell
  VMName      Name                    Enabled PrimaryStatusDescription SecondaryStatusDescription
  ------      ----                    ------- ------------------------ --------------------------
  DemoVM      Guest Service Interface False   OK
  DemoVM      Heartbeat               True    OK                       OK
  DemoVM      Key-Value Pair Exchange True    OK
  DemoVM      Shutdown                True    OK
  DemoVM      Time Synchronization    True    OK
  DemoVM      VSS                     True    OK
  ```

2. Aktivieren des `Guest Service Interface`-Integrationsdiensts

   ``` PowerShell
   Enable-VMIntegrationService -VMName "DemoVM" -Name "Guest Service Interface"
   ```
   
   Falls Sie `Get-VMIntegrationService -VMName "DemoVM"` ausführen, sehen Sie, dass der Integrationsdienst für den Gastschnittstellendienst aktiviert ist.
 
3. Deaktivieren des `Guest Service Interface`-Integrationsdiensts

   ``` PowerShell
   Disable-VMIntegrationService -VMName "DemoVM" -Name "Guest Service Interface"
   ```
   
Integrationsdienste sind so konzipiert, dass sie für ein ordnungsgemäßes Funktionieren sowohl im Host- als auch im Gastsystem aktiviert sein müssen.  In Windows-Gastbetriebssystemen sind Integrationsdienste standardmäßig aktiviert, können aber deaktiviert werden.  Dies wird im nächsten Abschnitt erläutert.


## <a name="manage-integration-services-from-guest-os-windows"></a>Verwalten von Integrationsdiensten im Gastbetriebssystem (Windows)

> **Hinweis:** Durch das Deaktivieren von Integrationsdiensten kann die Fähigkeit des Hosts zum Verwalten Ihres virtuellen Computers gravierend eingeschränkt werden.  Integrationsdienste müssen für einen ordnungsgemäßen Betrieb auf Host und Gast aktiviert sein.

Integrationsdienste werden unter Windows als Dienste angezeigt. Zum Aktivieren oder Deaktivieren eines Integrationsdiensts innerhalb des virtuellen Computers öffnen Sie den Windows-Dienste-Manager.

![](../user-guide/media/HVServices.png) 

Suchen Sie die Dienste, die „Hyper-V“ im Namen enthalten. Klicken Sie mit der rechten Maustaste auf den Dienst, den Sie aktivieren oder deaktivieren bzw. starten oder beenden möchten.

Um alle Integrationsdienste mithilfe von PowerShell anzuzeigen, führen Sie Folgendes aus:

```PowerShell
Get-Service -Name vm*
```

Als Rückgabe erhalten Sie eine Liste, die ungefähr so aussieht:

```PowerShell
Status   Name               DisplayName
------   ----               -----------
Running  vmicguestinterface Hyper-V Guest Service Interface
Running  vmicheartbeat      Hyper-V Heartbeat Service
Running  vmickvpexchange    Hyper-V Data Exchange Service
Running  vmicrdv            Hyper-V Remote Desktop Virtualizati...
Running  vmicshutdown       Hyper-V Guest Shutdown Service
Running  vmictimesync       Hyper-V Time Synchronization Service
Stopped  vmicvmsession      Hyper-V VM Session Service
Running  vmicvss            Hyper-V Volume Shadow Copy Requestor
```

Starten oder Beenden von Diensten mit [`Start-Service`](https://technet.microsoft.com/en-us/library/hh849825.aspx) oder [`Stop-Service`](https://technet.microsoft.com/en-us/library/hh849790.aspx).

Um z.B. PowerShell Direct zu deaktivieren, können Sie `Stop-Service -Name vmicvmsession` ausführen.

Standardmäßig sind alle Integrationsdienste im Gastbetriebssystem aktiviert.

## <a name="manage-integration-services-from-guest-os-linux"></a>Verwalten von Integrationsdiensten im Gastbetriebssystem (Linux)

Linux-Integrationsdienste werden in der Regel über den Linux-Kernel bereitgestellt.

Prüfen Sie, ob die Integrationsdiensttreiber und -daemons ausgeführt werden, indem Sie in Ihrem Linux-Gastbetriebssystem die folgenden Befehle ausführen.

1. Der Treiber für Linux-Integrationsdienste heißt „hv_utils“.  Führen Sie Folgendes aus, um festzustellen, ob er geladen ist.

  ``` BASH
  lsmod | grep hv_utils
  ``` 
  
  Die Ausgabe sollte wie folgt aussehen:  
  
  ``` BASH
  Module                  Size   Used by
  hv_utils               20480   0
  hv_vmbus               61440   8 hv_balloon,hyperv_keyboard,hv_netvsc,hid_hyperv,hv_utils,hyperv_fb,hv_storvsc
  ```

2. Führen Sie den folgenden Befehl im Linux-Gastbetriebssystem aus, um zu prüfen, ob die erforderlichen Daemons ausgeführt werden.
  
  ``` BASH
  ps -ef | grep hv
  ```
  
  Die Ausgabe sollte wie folgt aussehen:  
  
  ``` BASH
  root       236     2  0 Jul11 ?        00:00:00 [hv_vmbus_con]
  root       237     2  0 Jul11 ?        00:00:00 [hv_vmbus_ctl]
  ...
  root       252     2  0 Jul11 ?        00:00:00 [hv_vmbus_ctl]
  root      1286     1  0 Jul11 ?        00:01:11 /usr/lib/linux-tools/3.13.0-32-generic/hv_kvp_daemon
  root      9333     1  0 Oct12 ?        00:00:00 /usr/lib/linux-tools/3.13.0-32-generic/hv_kvp_daemon
  root      9365     1  0 Oct12 ?        00:00:00 /usr/lib/linux-tools/3.13.0-32-generic/hv_vss_daemon
  scooley  43774 43755  0 21:20 pts/0    00:00:00 grep --color=auto hv          
  ```
  
  Führen Sie Folgendes aus, um festzustellen, welche Daemons verfügbar sind:
  ``` BASH
  compgen -c hv_
  ```
  
  Die Ausgabe sollte wie folgt aussehen:
  
  ``` BASH
  hv_vss_daemon
  hv_get_dhcp_info
  hv_get_dns_info
  hv_set_ifconfig
  hv_kvp_daemon
  hv_fcopy_daemon     
  ```
  
  Daemons für Integrationsdienste, die ggf. angezeigt werden:  
  * **`hv_vss_daemon`**: Dieser Daemon ist zum Erstellen von Sicherungen aktiver virtueller Linux-Computer erforderlich.
  * **`hv_kvp_daemon`**: Dieser Daemon ermöglicht das Festlegen und Abfragen interner und externer Schlüsselwertpaare.
  * **`hv_fcopy_daemon`**: Dieser Daemon implementiert einen Dateikopierdienst zwischen Host und Gast.

> **Hinweis:** Wenn die genannten Daemons für Integrationsdienste nicht verfügbar sind, werden sie möglicherweise auf Ihrem System nicht unterstützt oder sind nicht installiert.  Weitere distributionsbezogene Informationen finden Sie [hier](https://technet.microsoft.com/en-us/library/dn531030.aspx).  

In diesem Beispiel beenden und starten wir den KVP-Daemon `hv_kvp_daemon`.

Beenden Sie den Prozess des Daemons mithilfe der PID (Prozess-ID), die sich in der zweiten Spalte der obigen Ausgabe befindet.  Alternativ können Sie den richtigen Prozess mithilfe von `pidof` finden.  Da Hyper-V-Daemons als Root ausgeführt werden, benötigen Sie Rootberechtigungen.

``` BASH
sudo kill -15 `pidof hv_kvp_daemon`
```

Wenn Sie `ps -ef | hv` nochmals ausführen, sehen Sie, dass kein `hv_kvp_daemon`-Prozess mehr vorhanden ist.

Um den Daemon erneut zu starten, führen Sie den Daemon als Root aus.

``` BASH
sudo hv_kvp_daemon
``` 

Wenn Sie `ps -ef | hv` nochmals ausführen, sehen Sie, dass ein `hv_kvp_daemon`-Prozess mit einer neuen Prozess-ID vorhanden ist.


## <a name="integration-service-maintenance"></a>Wartung von Integrationsdiensten

Die Wartung von Integrationsdiensten in Windows 10 erfolgt standardmäßig, sofern Ihre virtuellen Computer wichtige Updates von Windows Update erhalten können.  

Indem die Integrationsdienste auf dem neuesten Stand gehalten werden, können die bestmögliche Leistung sowie die besten Funktionen für virtuelle Computer bereitgestellt werden.

**Für auf Windows10-Hosts ausgeführte virtuelle Computer:**

> **Hinweis:** Die ISO-Imagedatei „vmguest.iso“ wird nicht mehr zum Aktualisieren von Integrationskomponenten benötigt. Sie ist nicht in Hyper-V unter Windows 10 enthalten.

| Gastbetriebssystem | Updatemechanismus | Anmerkungen |
|:---------|:---------|:---------|
| Windows 10 | Windows Update | |
| Windows8.1 | Windows Update | |
| Windows 8 | Windows Update | Benötigt den Integrationsdienst „Datenaustausch“.* |
| Windows 7 | Windows Update | Benötigt den Integrationsdienst „Datenaustausch“.* |
| Windows Vista (SP 2) | Windows Update | Benötigt den Integrationsdienst „Datenaustausch“.* |
| - | | |
| Windows Server 2012 R2 | Windows Update | |
| Windows Server 2012 | Windows Update | Benötigt den Integrationsdienst „Datenaustausch“.* |
| Windows Server 2008 R2 (SP 1) | Windows Update | Benötigt den Integrationsdienst „Datenaustausch“.* |
| Windows Server 2008 (SP 2) | Windows Update | Erweiterte Unterstützung nur in Server 2016 ([Weitere Informationen](https://support.microsoft.com/en-us/lifecycle?p1=12925)). |
| Windows Home Server2011 | Windows Update | Nicht in Server 2016 unterstützt ([Weitere Informationen](https://support.microsoft.com/en-us/lifecycle?p1=15820)). |
| Windows Small Business Server2011 | Windows Update | Nicht im grundlegenden Support enthalten ([Weitere Informationen](https://support.microsoft.com/en-us/lifecycle?p1=15817)). |
| - | | |
| Linux-Gastbetriebssysteme | Paket-Manager | Linux-Integrationskomponenten sind in die Distribution integriert. Es können jedoch optionale Updates verfügbar sein. ******** |

>  \* Wenn der Integrationsdienst „Datenaustausch“ nicht aktiviert werden kann, stehen die Integrationskomponenten für diese Gastbetriebssysteme als CAB-Dateien im Download Center [hier](https://support.microsoft.com/en-us/kb/3071740) zur Verfügung.  
  Anweisungen zum Ausführen einer CAB-Datei finden Sie [hier](http://blogs.technet.com/b/virtualization/archive/2015/07/24/integration-components-available-for-virtual-machines-not-connected-to-windows-update.aspx).


**Für auf Windows8.1-Hosts ausgeführte virtuelle Computer:**

| Gastbetriebssystem | Updatemechanismus | Anmerkungen |
|:---------|:---------|:---------|
| Windows 10 | Windows Update | |
| Windows8.1 | Windows Update | |
| Windows 8 | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows 7 | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Vista (SP 2) | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows XP (SP 2, SP 3) | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| - | | |
| Windows Server 2012 R2 | Windows Update | |
| Windows Server 2012 | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server 2008 R2 | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server 2008 (SP 2) | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Home Server2011 | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Small Business Server2011 | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server 2003 R2 (SP 2) | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server 2003 (SP 2) | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| - | | |
| Linux-Gastbetriebssysteme | Paket-Manager | Linux-Integrationskomponenten sind in die Distribution integriert. Es können jedoch optionale Updates verfügbar sein. ** |


**Für auf Windows8-Hosts ausgeführte virtuelle Computer:**

| Gastbetriebssystem | Updatemechanismus | Anmerkungen |
|:---------|:---------|:---------|
| Windows8.1 | Windows Update | |
| Windows 8 | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows 7 | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Vista (SP 2) | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows XP (SP 2, SP 3) | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| - | | |
| Windows Server 2012 R2 | Windows Update | |
| Windows Server 2012 | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server 2008 R2 | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4).|
| Windows Server 2008 (SP 2) | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Home Server2011 | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Small Business Server2011 | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server 2003 R2 (SP 2) | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| Windows Server 2003 (SP 2) | Integrationsdienste-Datenträger | Anweisungen finden Sie [hier](https://technet.microsoft.com/en-us/library/hh846766.aspx#BKMK_step4). |
| - | | |
| Linux-Gastbetriebssysteme | Paket-Manager | Linux-Integrationskomponenten sind in die Distribution integriert. Es können jedoch optionale Updates verfügbar sein. ** |

 > ** Weitere Informationen zu Linux-Gastbetriebssystemen finden Sie [hier](https://technet.microsoft.com/en-us/library/dn531030.aspx). 
