---
title: &1398555736 Verwalten von Hyper-V-Integrationsdiensten
description: Verwalten von Hyper-V-Integrationsdiensten
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &740518608 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 9cafd6cb-dbbe-4b91-b26c-dee1c18fd8c2
---

# Verwalten von Hyper-V-Integrationsdiensten

Integrationsdienste (oft als „Integrationskomponenten“ bezeichnet) ermöglichen dem virtuellen Computer das Kommunizieren mit dem Hyper-V-Host. Viele dieser Dienste sind bloß zweckmäßig (z. B. Kopieren von Gastdateien), während andere für das ordnungsgemäße Funktionieren des Gastbetriebssystems wichtig sein können (z. B. Zeitsynchronisierung).

Dieser Artikel enthält ausführliche Informationen zum Verwalten von Integrationsdiensten mithilfe von Hyper-V-Manager und PowerShell in Windows 10. Weitere Informationen zu den einzelnen Integrationsdiensten finden Sie unter <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Integrationsdienste</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

## Aktivieren und Deaktivieren von Integrationsdiensten mit Hyper-V-Manager

1. Wählen Sie einen virtuellen Computer aus, und öffnen Sie „Einstellungen“.
  <g id="1" ctype="x-linkText"></g>

2. Wechseln Sie im Fenster „Einstellungen“ des virtuellen Computers unter „Verwaltung“ zur Registerkarte „Integrationsdienste“.

  <g id="1" ctype="x-linkText"></g>

  Hier sehen Sie alle Integrationsdienste, die auf diesem Hyper-V-Host verfügbar sind. Wichtig ist der Hinweis, dass das Gastbetriebssystem ggf. nicht alle aufgeführten Integrationsdienste unterstützt.

## Aktivieren und Deaktivieren von Integrationsdiensten mit PowerShell

Integrationsdienste können mit PowerShell aktiviert und deaktiviert werden, indem [<g id="2" ctype="x-code">Enable-VMIntegrationService</g>](https://technet.microsoft.com/de-de/library/hh848500.aspx) und [<g id="4" ctype="x-code">Disable-VMIntegrationService</g>](https://technet.microsoft.com/de-de/library/hh848488.aspx) ausgeführt werden.

In diesem Beispiel aktivieren und deaktivieren wir den Integrationsdienst für das Kopieren von Gastdateien auf dem zuvor erwähnten virtuellen Computer „demovm“.

1. Prüfen, welche Integrationsdienste ausgeführt werden

  ``` PowerShell
  Get-VMIntegrationService -VMName "demovm"
  ```

  Die Ausgabe sieht folgendermaßen aus:
  ``` PowerShell
  VMName      Name                    Enabled PrimaryStatusDescription SecondaryStatusDescription
  ------      ----                    ------- ------------------------ --------------------------
  demovm      Guest Service Interface False   OK
  demovm      Heartbeat               True    OK                       OK
  demovm      Key-Value Pair Exchange True    OK
  demovm      Shutdown                True    OK
  demovm      Time Synchronization    True    OK
  demovm      VSS                     True    OK
  ```

2. Aktivieren des Integrationsdiensts <g id="2" ctype="x-code">Guest Service Interface</g>

   ``` PowerShell
   Enable-VMIntegrationService -VMName "demovm" -Name "Guest Service Interface"
   ```

   Wenn Sie <g id="2" ctype="x-code">Get-VMIntegrationService -VMName "demovm"</g> ausführen, sehen Sie, dass der Integrationsdienst „Guest Service Interface“ aktiviert ist.

3. Deaktivieren des Integrationsdiensts <g id="2" ctype="x-code">Guest Service Interface</g>

   ``` PowerShell
   Disable-VMIntegrationService -VMName "demovm" -Name "Guest Service Interface"
   ```

Integrationsdienste sind so konzipiert, dass sie für ein ordnungsgemäßes Funktionieren sowohl im Host- als auch im Gastsystem aktiviert sein müssen. In Windows-Gastbetriebssystemen sind Integrationsdienste standardmäßig aktiviert, können aber deaktiviert werden. Dies wird im nächsten Abschnitt erläutert.


## Verwalten von Integrationsdiensten im Gastbetriebssystem (Windows)

> <g id="1" ctype="x-strong">Hinweis:</g> Durch das Deaktivieren von Integrationsdiensten kann die Fähigkeit des Hosts zum Verwalten Ihres virtuellen Computers gravierend eingeschränkt werden. Integrationsdienste müssen für einen ordnungsgemäßen Betrieb auf Host und Gast aktiviert sein.

Integrationsdienste werden unter Windows als Dienste angezeigt. Zum Aktivieren oder Deaktivieren eines Integrationsdiensts innerhalb des virtuellen Computers öffnen Sie den Windows-Dienste-Manager.

<g id="1" ctype="x-linkText"></g>

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

Starten oder beenden Sie Dienste mit [<g id="2" ctype="x-code">Start-Service</g>](https://technet.microsoft.com/de-de/library/hh849825.aspx) bzw. [<g id="4" ctype="x-code">Stop-Service</g>](https://technet.microsoft.com/de-de/library/hh849790.aspx).

Um z. B. PowerShell Direct zu deaktivieren, können Sie <g id="2" ctype="x-code">Stop-Service -Name vmicvmsession</g> ausführen.

Standardmäßig sind alle Integrationsdienste im Gastbetriebssystem aktiviert.

## Verwalten von Integrationsdiensten im Gastbetriebssystem (Linux)

Linux-Integrationsdienste werden in der Regel über den Linux-Kernel bereitgestellt.

Prüfen Sie, ob die Integrationsdiensttreiber und -daemons ausgeführt werden, indem Sie in Ihrem Linux-Gastbetriebssystem die folgenden Befehle ausführen.

1. Der Treiber für Linux-Integrationsdienste heißt „hv_utils“. Führen Sie Folgendes aus, um festzustellen, ob er geladen ist.

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
  * **<g id="2" ctype="x-code">hv_vss_daemon</g>**: Dieser Daemon ist zum Erstellen von Sicherungen aktiver virtueller Linux-Computer erforderlich.
  * **<g id="2" ctype="x-code">hv_kvp_daemon</g>**: Dieser Daemon ermöglicht das Festlegen und Abfragen interner und externer Schlüsselwertpaare.
  * **<g id="2" ctype="x-code">hv_fcopy_daemon</g>**: Dieser Daemon implementiert einen Dateikopierdienst zwischen Host und Gast.

> <g id="1" ctype="x-strong">Hinweis:</g> Wenn die genannten Daemons für Integrationsdienste nicht verfügbar sind, werden sie möglicherweise auf Ihrem System nicht unterstützt oder sind nicht installiert. Weitere distributionsbezogene Informationen finden Sie <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">hier</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

In diesem Beispiel beenden und starten wir den KVP-Daemon <g id="2" ctype="x-code">hv_kvp_daemon</g>.

Beenden Sie den Prozess des Daemons mithilfe der PID (Prozess-ID), die sich in der zweiten Spalte der obigen Ausgabe befindet. Alternativ können Sie den richtigen Prozess mithilfe von <g id="2" ctype="x-code">pidof</g> finden. Da Hyper-V-Daemons als Root ausgeführt werden, benötigen Sie Rootberechtigungen.

``` BASH
sudo kill -15 `pidof hv_kvp_daemon`
```

Wenn Sie <g id="2" ctype="x-code">ps -ef | hv</g> nochmals ausführen, sehen Sie, dass der Prozess <g id="4" ctype="x-code">hv_kvp_daemon</g> nicht mehr vorhanden ist.

Um den Daemon erneut zu starten, führen Sie den Daemon als Root aus.

``` BASH
sudo hv_kvp_daemon
```

Wenn Sie <g id="2" ctype="x-code">ps -ef | hv</g> nochmals ausführen, sehen Sie, dass der Prozess <g id="4" ctype="x-code">hv_kvp_daemon</g> eine neue Prozess-ID hat.


## Wartung von Integrationsdiensten

Halten Sie Integrationsdienste auf dem neuesten Stand, um sich die bestmögliche Leistung und neuesten Features für virtuelle Computer zu sichern.

<g id="1" ctype="x-strong">Für auf Windows 10-Hosts ausgeführte virtuelle Computer:</g>

> <g id="1" ctype="x-strong">Hinweis:</g> Die ISO-Imagedatei „vmguest.iso“ wird nicht mehr zum Aktualisieren von Integrationskomponenten benötigt. Sie ist nicht in Hyper-V unter Windows 10 enthalten.

| Gastbetriebssystem| Updatemechanismus| Anmerkungen|
|:---------|:---------|:---------|
| Windows 10| Windows Update| |
| Windows 8.1| Windows Update| |
| Windows 8| Windows Update| Benötigt den Integrationsdienst „Datenaustausch“.<g id="2" ctype="x-strong">*</g>|
| Windows 7| Windows Update| Benötigt den Integrationsdienst „Datenaustausch“.<g id="2" ctype="x-strong">*</g>|
| Windows Vista (SP 2)| Windows Update| Benötigt den Integrationsdienst „Datenaustausch“.<g id="2" ctype="x-strong">*</g>|
| –| | |
| Windows Server 2012 R2| Windows Update| |
| Windows Server 2012| Windows Update| Benötigt den Integrationsdienst „Datenaustausch“.<g id="2" ctype="x-strong">*</g>|
| Windows Server 2008 R2 (SP 1)| Windows Update| Benötigt den Integrationsdienst „Datenaustausch“.<g id="2" ctype="x-strong">*</g>|
| Windows Server 2008 (SP 2)| Windows Update| Erweiterte Unterstützung nur in Server 2016 (<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Weitere Informationen</g><g id="2CapsExtId3" ctype="x-title"></g></g>.|
| Windows Home Server 2011| Windows Update| Nicht in Server 2016 unterstützt (<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Weitere Informationen</g><g id="2CapsExtId3" ctype="x-title"></g></g>.|
| Windows Small Business Server 2011| Windows Update| Nicht im grundlegenden Support enthalten (<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Weitere Informationen</g><g id="2CapsExtId3" ctype="x-title"></g></g>.|
| -| | |
| Linux-Gastbetriebssysteme| Paket-Manager| Linux-Integrationskomponenten sind in die Distribution integriert. Es können jedoch optionale Updates verfügbar sein.<g id="1" ctype="x-strong">****</g>|

<g id="1" ctype="x-strong">\*</g> Wenn der Integrationsdienst „Datenaustausch“ nicht aktiviert werden kann, stehen die Integrationskomponenten für diese Gastbetriebssysteme als CAB-Dateien im Downloadcenter <g id="3CapsExtId1" ctype="x-link"><g id="3CapsExtId2" ctype="x-linkText">hier</g><g id="3CapsExtId3" ctype="x-title"></g></g> zur Verfügung. Anweisungen zum Ausführen einer CAB-Datei finden Sie <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">hier</g><g id="2CapsExtId3" ctype="x-title"></g></g>.


<g id="1" ctype="x-strong">Für auf Windows 8.1-Hosts ausgeführte virtuelle Computer:</g>

| Gastbetriebssystem| Updatemechanismus| Anmerkungen|
|:---------|:---------|:---------|
| Windows 10| Windows Update| |
| Windows 8.1| Windows Update| |
| Windows 8| Integrationsdienste-Datenträger| |
| Windows 7| Integrationsdienste-Datenträger| |
| Windows Vista (SP 2)| Integrationsdienste-Datenträger| |
| Windows XP (SP 2, SP 3)| Integrationsdienste-Datenträger| |
| –| | |
| Windows Server 2012 R2| Windows Update| |
| Windows Server 2012| Integrationsdienste-Datenträger| |
| Windows Server 2008 R2| Integrationsdienste-Datenträger| |
| Windows Server 2008 (SP 2)| Integrationsdienste-Datenträger| |
| Windows Home Server 2011| Integrationsdienste-Datenträger| |
| Windows Small Business Server 2011| Integrationsdienste-Datenträger| |
| Windows Server 2003 R2 (SP 2)| Integrationsdienste-Datenträger| |
| Windows Server 2003 (SP 2)| Integrationsdienste-Datenträger| |
| -| | |
| Linux-Gastbetriebssysteme| Paket-Manager| Linux-Integrationskomponenten sind in die Distribution integriert. Es können jedoch optionale Updates verfügbar sein.<g id="1" ctype="x-strong">****</g>|


<g id="1" ctype="x-strong">Für auf Windows 8-Hosts ausgeführte virtuelle Computer:</g>

| Gastbetriebssystem| Updatemechanismus| Anmerkungen|
|:---------|:---------|:---------|
| Windows 8.1| Windows Update| |
| Windows 8| Integrationsdienste-Datenträger| |
| Windows 7| Integrationsdienste-Datenträger| |
| Windows Vista (SP 2)| Integrationsdienste-Datenträger| |
| Windows XP (SP 2, SP 3)| Integrationsdienste-Datenträger| |
| –| | |
| Windows Server 2012 R2| Windows Update| |
| Windows Server 2012| Integrationsdienste-Datenträger| |
| Windows Server 2008 R2| Integrationsdienste-Datenträger| |
| Windows Server 2008 (SP 2)| Integrationsdienste-Datenträger| |
| Windows Home Server 2011| Integrationsdienste-Datenträger| |
| Windows Small Business Server 2011| Integrationsdienste-Datenträger| |
| Windows Server 2003 R2 (SP 2)| Integrationsdienste-Datenträger| |
| Windows Server 2003 (SP 2)| Integrationsdienste-Datenträger| |
| -| | |
| Linux-Gastbetriebssysteme| Paket-Manager| Linux-Integrationskomponenten sind in die Distribution integriert. Es können jedoch optionale Updates verfügbar sein.<g id="1" ctype="x-strong">****</g>|


Eine Anleitung zur Aktualisierung mithilfe des Integrationsdienste-Datenträgers für Windows 8 und Windows 8.1 finden Sie <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">hier</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

 <g id="1" ctype="x-strong">\****</g> Weitere Informationen zu Linux-Gastbetriebssystemen finden Sie <g id="3CapsExtId1" ctype="x-link"><g id="3CapsExtId2" ctype="x-linkText">hier</g><g id="3CapsExtId3" ctype="x-title"></g></g>.






<!--HONumber=May16_HO1-->


