---
title: PowerShell-Codeausschnitte
description: PowerShell-Codeausschnitte
keywords: windows 10, hyper-v
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: dc33c703-c5bc-434e-893b-0c0976b7cb88
translationtype: Human Translation
ms.sourcegitcommit: e14ede0a2b13de08cea0a955b37a21a150fb88cf
ms.openlocfilehash: 4b8a6905e3497b5fbecf938ea35b6cc57ae37be2

---

# PowerShell-Codeausschnitte

PowerShell ist ein sehr leistungsfähiges Skripterstellungs-, Automatisierungs- und Verwaltungstool für Hyper-V.  In diesem Artikel stellen wir verschiedene Aufgaben vor, die Sie mit PowerShell erledigen können!

Sämtliche Hyper-V-Verwaltungsaufgaben müssen mit Administratorrechten ausgeführt werden, sodass vorausgesetzt wird, dass alle Skripts und Codeausschnitte mit einem Hyper-V-Administratorkonto ausgeführt werden.

Wenn Sie nicht sicher sind, ob Sie über die richtigen Berechtigungen verfügen, geben Sie `Get-VM` ein. Falls der Befehl ohne Fehler ausgeführt wird, können Sie loslegen.


## PowerShell Direct-Tools
Für alle Skripts und Codeausschnitte in diesem Abschnitt gelten die folgenden Voraussetzungen.

**Anforderungen**:  
*  PowerShell Direct.  Windows 10 als Gast- und Hostbetriebssystem.

**Gängige Variablen**:  
`$VMName` : Zeichenfolge mit dem VM-Namen.  Rufen Sie eine Liste der verfügbaren VMs ab mit `Get-VM`  
`$cred` : Anmeldeinformationen für das Gastbetriebssystem.  Können aufgefüllt werden mit `$cred = Get-Credential`  

### Prüfen, ob der Gast gestartet wurde

Hyper-V-Manager bietet Ihnen keinen Einblick in das Gastbetriebssystem, weshalb es häufig schwierig ist festzustellen, ob das Gastbetriebssystem gestartet wurde.

Hier sind zwei Ansichten derselben Funktionalität, zunächst als Codeausschnitt dann als PowerShell-Funktion.

Codeausschnitt:  
``` PowerShell
if((Invoke-Command -VMName $VMName -Credential $cred {"Test"}) -ne "Test"){Write-Host "Not Booted"} else {Write-Host "Booted"}
```  

Funktion:  
``` PowerShell
function waitForPSDirect([string]$VMName, $cred){
   Write-Output "[$($VMName)]:: Waiting for PowerShell Direct (using $($cred.username))"
   while ((Invoke-Command -VMName $VMName -Credential $cred {"Test"} -ea SilentlyContinue) -ne "Test") {Sleep -Seconds 1}}
```

**Ergebnis**  
Gibt eine Meldung aus und aktiviert eine Sperre, bis die Verbindung mit der VM hergestellt ist.  
Erfolgt unbeaufsichtigt.

### Skriptsperre, bis das Gastbetriebssystem eine Netzwerkverbindung hat
Mit PowerShell Direct ist es möglich, eine Verbindung mit einer PowerShell-Sitzung auf einem virtuellen Computer herzustellen, bevor der virtuelle Computer eine IP-Adresse empfangen hat.

``` PowerShell
# Wait for DHCP
while ((Get-NetIPAddress | ? AddressFamily -eq IPv4 | ? IPAddress -ne 127.0.0.1).SuffixOrigin -ne "Dhcp") {sleep -Milliseconds 10}
```

** Ergebnis ** Sperre, bis eine DHCP-Lease empfangen wird.  Da dieses Skript kein bestimmtes Subnetz bzw. keine bestimmte IP-Adresse sucht, funktioniert es unabhängig von der verwendeten Netzwerkkonfiguration.  
Erfolgt unbeaufsichtigt.

## Verwalten von Anmeldeinformationen mit PowerShell
Hyper-V-Skripts erfordern häufig die Verarbeitung von Anmeldeinformationen für einen oder mehrere virtuelle Computer, Hyper-V-Hosts oder beides.

Es gibt beim Arbeiten mit PowerShell Direct oder standardmäßigem PowerShell-Remoting mehrere Methoden, wie Sie dies erreichen können:

1. Die erste (und einfachste) Möglichkeit ist das Verwenden der gleichen gültigen Anmeldeinformationen im Host und Gast bzw. lokalen und Remotehost.  
  Dies ist recht einfach, wenn Sie sich mit Ihrem Microsoft-Konto anmelden oder sich in einer Domänenumgebung befinden.  
  In diesem Szenario können Sie nur `Invoke-Command -VMName "test" {get-process}` ausführen.

2. Von PowerShell zur Eingabe von Anmeldeinformationen auffordern lassen  
  Wenn Ihre Anmeldeinformationen nicht übereinstimmen, erhalten Sie automatisch eine Anmeldeaufforderung, sodass Sie die entsprechenden Anmeldeinformationen für den virtuellen Computer angeben können.

3. Speichern Sie Anmeldeinformationen zur Wiederverwendung in einer Variablen.
  Führen Sie einen einfachen Befehl wie diesen aus:  
  ``` PowerShell
  $localCred = Get-Credential
   ```
  Und dann einen wie diesen:
  ``` PowerShell
  Invoke-Command -VMName "test" -Credential $localCred  {get-process} 
  ```
  Das bedeutet, dass Sie pro Skript-/PowerShell-Sitzung nur einmal zur Eingabe Ihrer Anmeldeinformationen aufgefordert werden.

4. Programmieren Sie Ihre Anmeldeinformationen in Ihre Skripts.  **Machen Sie das auf keinen Fall für reale Workloads oder Systeme.**
 > Warnung: _Gehen Sie bei einem Produktionssystem auf keinen Fall so vor.  Verwenden Sie keine echten Kennwörter._
  
  Sie können ein „PSCredential“-Objekt mit Code wie dem folgenden manuell erstellen:  
  ``` PowerShell
  $localCred = New-Object -typename System.Management.Automation.PSCredential -argumentlist "Administrator", (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) 
  ```
  Überaus unsicher, aber zum Testen hilfreich.  Jetzt erhalten Sie in der ganzen Sitzung keine Aufforderungen mehr. 




<!--HONumber=Jun16_HO4-->


