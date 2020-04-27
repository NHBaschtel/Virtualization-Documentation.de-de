---
title: Verwalten von virtuellen Windows-Computern mit PowerShell Direct
description: Verwalten von virtuellen Windows-Computern mit PowerShell Direct
keywords: Windows 10, Hyper-V, PowerShell, Integrationsdienste, Integrationskomponenten, Automatisierung, PowerShell Direct
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: fb228e06-e284-45c0-b6e6-e7b0217c3a49
ms.openlocfilehash: ed96c7ba30c83906cd3245a279ab078229400d8d
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/24/2020
ms.locfileid: "77439547"
---
# <a name="virtual-machine-automation-and-management-using-powershell"></a>Automatisierung und Verwaltung virtueller Computer mithilfe von PowerShell

Sie können PowerShell Direct verwenden, um beliebige PowerShell-Befehle in einem virtuellen Computer unter Windows 10 oder Windows Server 2016 von Ihrem Hyper-V-Host aus auszuführen – unabhängig von der Netzwerkkonfiguration oder den Einstellungen für die Remoteverwaltung.

Es folgen einige Möglichkeiten, wie Sie PowerShell Direct ausführen können:

* [Als interaktive Sitzung mit dem Cmdlet Enter-PSSession](#create-and-exit-an-interactive-powershell-session)
* [Als Einzelverwendungsabschnitt zum Ausführen eines einzelnen Befehls oder Skripts mit dem Cmdlet Invoke-Command](#run-a-script-or-command-with-invoke-command)
* [Als dauerhafte Sitzung (Build 14280 und höher) mit den Cmdlets New-PSSession, Copy-Item und Remove-PSSession](#copy-files-with-new-pssession-and-copy-item)

## <a name="requirements"></a>Anforderungen
**Betriebssystemanforderungen:**
* Host: Windows 10, Windows Server 2016 oder höher mit Hyper-V.
* Gast- bzw. virtueller Computer: Windows 10, Windows Server 2016 oder höher.

Wenn Sie ältere virtuelle Computer verwalten, verwenden Sie die Verbindung mit virtuellen Computern (VMConnect) oder [Konfigurieren Sie ein virtuelles Netzwerk für den virtuellen Computer](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc816585(v=ws.10)). 

**Konfigurationsanforderungen:**    
* Der virtuelle Computer muss lokal auf dem Host ausgeführt werden.
* Der virtuelle Computer muss gestartet sein und mit mindestens einem konfigurierten Benutzerprofil ausgeführt werden.
* Sie müssen am Hostcomputer als Hyper-V-Administrator angemeldet sein.
* Sie müssen gültige Anmeldeinformationen für den virtuellen Computer angeben.

-------------

## <a name="create-and-exit-an-interactive-powershell-session"></a>Erstellen und Beenden einer interaktiven PowerShell-Sitzung

Die einfachste Möglichkeit, PowerShell auf einem virtuellen Computer auszuführen, ist der Start einer interaktiven Sitzung.

Wenn die Sitzung gestartet wird, werden die eingegebenen Befehle auf dem virtuellen Computer ausgeführt – genau so, als würden Sie die Befehle direkt in eine PowerShell-Sitzung auf dem virtuellen Computer selbst eingeben.

**So starten Sie eine interaktive Sitzung:**

1. Öffnen Sie PowerShell auf dem Hyper-V-Host als Administrator.

2. Führen Sie einen der folgenden Befehle aus, um eine interaktive Sitzung mit dem Namen oder der GUID des virtuellen Computers zu erstellen:  
  
  ``` PowerShell
  Enter-PSSession -VMName <VMName>
  Enter-PSSession -VMId <VMId>
  ```
  
  Geben Sie Anmeldeinformationen für den virtuellen Computer ein, wenn Sie dazu aufgefordert werden.

3. Führen Sie Befehle auf dem virtuellen Computer aus.
  
  Der Name des virtuellen Computers wird als Präfix (VMName) für Ihre PowerShell-Eingabeaufforderung angezeigt:
  
  ``` 
  [VMName]: PS C:\>
  ```
  
  Jeder Befehl wird auf dem virtuellen Computer ausgeführt. Um dies zu testen, können Sie `ipconfig` oder `hostname` ausführen, um sicherzustellen, dass diese Befehle tatsächlich auf dem virtuellen Computer ausgeführt werden.
  
4. Wenn Sie fertig sind, führen Sie den folgenden Befehl zum Schließen der Sitzung aus:  
  
   ``` PowerShell
   Exit-PSSession 
   ``` 

> Hinweis: Wenn keine Verbindung für die Sitzung hergestellt werden kann, lesen Sie den Abschnitt [Problembehandlung](#troubleshooting), um mögliche Ursachen herauszufinden. 

Weitere Informationen zu diesen Cmdlets finden Sie unter [Enter-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Enter-PSSession?view=powershell-5.1) und [Exit-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Exit-PSSession?view=powershell-5.1). 

-------------

## <a name="run-a-script-or-command-with-invoke-command"></a>Ausführen eines Skripts oder Befehls mit „Invoke-Command“

PowerShell Direct mit Invoke-Command eignet sich ideal für Situationen, in denen Sie nur einen Befehl oder ein Skript auf einem virtuellen Computer ausführen, danach aber nicht weiter mit dem virtuellen Computer interagieren möchten.

**So führen Sie einen einzelnen Befehl aus:**

1. Öffnen Sie PowerShell auf dem Hyper-V-Host als Administrator.

2. Führen Sie einen der folgenden Befehle aus, um eine Sitzung mit dem Namen oder der GUID des virtuellen Computers zu erstellen:  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -ScriptBlock { command } 
   Invoke-Command -VMId <VMId> -ScriptBlock { command }
   ```
   
   Geben Sie Anmeldeinformationen für den virtuellen Computer ein, wenn Sie dazu aufgefordert werden.
   
   Der Befehl wird auf dem virtuellen Computer ausgeführt. Wenn der Befehl eine Konsolenausgabe umfasst, erfolgt diese auf Ihrer Konsole.  Die Verbindung wird automatisch geschlossen, nachdem der Befehl ausgeführt wurde.
   
   
**So führen Sie ein Skript aus:**

1. Öffnen Sie PowerShell auf dem Hyper-V-Host als Administrator.

2. Führen Sie einen der folgenden Befehle aus, um eine Sitzung mit dem Namen oder der GUID des virtuellen Computers zu erstellen:  
   
   ``` PowerShell
   Invoke-Command -VMName <VMName> -FilePath C:\host\script_path\script.ps1 
   Invoke-Command -VMId <VMId> -FilePath C:\host\script_path\script.ps1 
   ```
   
   Geben Sie Anmeldeinformationen für den virtuellen Computer ein, wenn Sie dazu aufgefordert werden.
   
   Das Skript wird auf dem virtuellen Computer ausgeführt.  Die Verbindung wird automatisch geschlossen, nachdem der Befehl ausgeführt wurde.

Weitere Informationen zu diesem Cmdlet finden Sie unter [Invoke-Command](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/Invoke-Command?view=powershell-5.1). 

-------------

## <a name="copy-files-with-new-pssession-and-copy-item"></a>Kopieren von Dateien mit New-PSSession und Copy-Item

> **Hinweis:** PowerShell Direct unterstützt nur permanente Sitzungen in Windows-Build 14280 und höher.

Permanente PowerShell-Sitzungen sind unglaublich nützlich, wenn Sie Skripts schreiben, die Aktionen über einen oder mehrere Remotecomputer hinweg koordinieren.  Nach dem Erstellen sind permanente Sitzungen im Hintergrund vorhanden, bis Sie sich dafür entscheiden, sie zu löschen.  Das bedeutet, dass Sie mit `Invoke-Command` und `Enter-PSSession` immer wieder auf die gleiche Sitzung verweisen können, ohne Anmeldeinformationen übergeben zu müssen.

Ebenso behalten Sitzungen ihren Zustand bei.  Da permanente Sitzungen – wie der Name schon sagt – dauerhaft sind, werden jegliche Variablen, die in einer Sitzung erstellt oder an eine Sitzung übergeben wurden, über mehrere Aufrufe hinweg beibehalten. Für die Arbeit mit permanenten Sitzungen sind verschiedene Tools verfügbar.  In diesem Beispiel werden [New-PSSession](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Core/New-PSSession?view=powershell-5.1) und [Copy-Item](https://docs.microsoft.com/powershell/module/Microsoft.PowerShell.Management/Copy-Item?view=powershell-5.1) verwendet, um Daten vom Host auf einen virtuellen Computer und von einem virtuellen Computer auf den Host zu verschieben.

**So erstellen Sie eine Sitzung und kopieren anschließend Dateien:**  

1. Öffnen Sie PowerShell auf dem Hyper-V-Host als Administrator.

2. Führen Sie einen der folgenden Befehle aus, um mit `New-PSSession` eine permanente PowerShell-Sitzung auf dem virtuellen Computer zu erstellen.
  
  ``` PowerShell
  $s = New-PSSession -VMName <VMName> -Credential (Get-Credential)
  $s = New-PSSession -VMId <VMId> -Credential (Get-Credential)
  ```
  
  Geben Sie Anmeldeinformationen für den virtuellen Computer ein, wenn Sie dazu aufgefordert werden.
  
  > **Warnung:**  
   Builds vor Buildnummer 14500 weisen einen Fehler auf.  Wenn Anmeldeinformationen nicht explizit mit dem Flag `-Credential` angegeben werden, stürzt der Dienst im Gastsystem ab und muss neu gestartet werden.  Wenn dieses Problem auftritt, finden Sie [hier](#error-a-remote-session-might-have-ended) Anweisungen zur Umgehung.
  
3. Kopieren Sie eine Datei auf den virtuellen Computer.
  
  Um `C:\host_path\data.txt` vom Host auf den virtuellen Computer zu kopieren, führen Sie folgenden Befehl aus:
  
  ``` PowerShell
  Copy-Item -ToSession $s -Path C:\host_path\data.txt -Destination C:\guest_path\
  ```
  
4.  Kopieren Sie eine Datei vom virtuellen Computer auf den Host. 
   
   Um `C:\guest_path\data.txt` vom virtuellen Computer auf den Host zu kopieren, führen Sie folgenden Befehl aus:
  
  ``` PowerShell
  Copy-Item -FromSession $s -Path C:\guest_path\data.txt -Destination C:\host_path\
  ```

5. Beenden Sie die permanente Sitzung mit `Remove-PSSession`.
  
  ``` PowerShell 
  Remove-PSSession $s
  ```
  
-------------

## <a name="troubleshooting"></a>Problembehandlung

PowerShell Direct zeigt eine kleine Menge von Fehlermeldungen an.  Es folgen die häufigsten davon, verschiedene Ursachen und Tools für die Untersuchung von Problemen.

### <a name="-vmname-or--vmid-parameters-dont-exist"></a>Parameter „-VMName“ oder „-VMID“ nicht vorhanden
**Problem:**  
`Enter-PSSession`, `Invoke-Command` oder `New-PSSession` weisen keinen Parameter `-VMName` oder `-VMId` auf.

**Mögliche Ursachen:**  
Der wahrscheinlichste Grund ist, dass PowerShell Direct von Ihrem Hostbetriebssystem nicht unterstützt wird.

Sie können Ihren Windows-Build mithilfe des folgenden Befehls überprüfen:

``` PowerShell
[System.Environment]::OSVersion.Version
```

Wenn Sie einen unterstützten Build ausführen, ist es auch möglich, dass Ihre PowerShell-Version PowerShell Direct nicht unterstützt.  Um PowerShell Direct und JEA verwenden zu können, müssen Sie über Hauptversion 5 oder höher verfügen.

Sie können Ihren PowerShell-Versionsbuild mithilfe des folgenden Befehls überprüfen:

``` PowerShell
$PSVersionTable.PSVersion
```


### <a name="error-a-remote-session-might-have-ended"></a>Fehler: Eine Remotesitzung wurde möglicherweise getrennt.
> **Hinweis:**  
Bei Eingabe von „Enter-PSSession“ auf Hosts mit Builds zwischen 10240 und 12400 werden alle unten stehenden Fehler als „Eine Remotesitzung wurde möglicherweise beendet.“ gemeldet.

**Fehlermeldung:**
```
Enter-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**Mögliche Ursachen:**
* Der virtuelle Computer ist vorhanden, wird aber nicht ausgeführt.
* Das Gastbetriebssystem unterstützt PowerShell Direct nicht (Informationen finden Sie in den [Anforderungen](#requirements)).
* PowerShell ist noch nicht auf dem Gast verfügbar.
  * Der Startvorgang des Betriebssystems ist noch nicht abgeschlossen.
  * Das Betriebssystem kann nicht ordnungsgemäß starten.
  * Bei einigen Ereignissen zur Startzeit sind Benutzereingaben erforderlich.

Sie können das Cmdlet [Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps) verwenden, um zu überprüfen, welche virtuellen Computer auf dem Host ausgeführt werden.

**Fehlermeldung:**  
```
New-PSSession : An error has occurred which Windows PowerShell cannot handle. A remote session might have ended.
```

**Mögliche Ursachen:**
* Einer der oben genannten Gründe – alle sind gleichermaßen anwendbar für `New-PSSession`  
* Ein Fehler in aktuellen Builds. Anmeldeinformationen müssen explizit mithilfe von `-Credential` übergeben werden.  Wenn dies passiert, reagiert der gesamte Dienst im Gastbetriebssystem nicht mehr und muss neu gestartet werden.  Sie können mit Enter-PSSession überprüfen, ob die Sitzung noch verfügbar ist.

Um das Problem mit den Anmeldeinformationen zu umgehen, melden Sie sich mithilfe von VMConnect am virtuellen Computer an, öffnen Sie PowerShell, und starten Sie den vmicvmsession-Dienst mithilfe des folgenden PowerShell-Befehls neu:

``` PowerShell
Restart-Service -Name vmicvmsession
```

### <a name="error-parameter-set-cannot-be-resolved"></a>Fehler: Parametersatz kann nicht aufgelöst werden.
**Fehlermeldung:**  
``` 
Enter-PSSession : Parameter set cannot be resolved using the specified named parameters.
```

**Mögliche Ursachen:**  
* `-RunAsAdministrator` wird beim Herstellen von Verbindungen mit virtuellen Computern nicht unterstützt.
     
  Beim Verbinden mit einem Windows-Container erlaubt das Flag `-RunAsAdministrator` Administratorverbindungen ohne explizite Anmeldeinformationen.  Da virtuelle Computer dem Host keinen impliziten Administratorzugriff gewähren, müssen Sie die Anmeldeinformationen explizit eingeben.

Administratoranmeldeinformationen können dem virtuellen Computer über den Parameter `-Credential` übergeben oder auf Aufforderung manuell eingegeben werden.


### <a name="error-the-credential-is-invalid"></a>Fehler: Die Anmeldeinformationen sind ungültig.

**Fehlermeldung:**  
```
Enter-PSSession : The credential is invalid.
```

**Mögliche Ursachen:** 
* Die Gastanmeldeinformationen konnten nicht überprüft werden.
  * Die angegebenen Anmeldeinformationen sind falsch.
  * Es gibt keine Benutzerkonten auf dem Gast (das Betriebssystem wurde noch nicht gestartet)
  * Wenn Sie die Verbindung als Administrator herstellen: Der Administrator wurde nicht als aktiver Benutzer festgelegt.  Weitere Informationen finden Sie [hier](<https://docs.microsoft.com/previous-versions/windows/it-pro/windows-8.1-and-8/hh825104(v=win.10)>).
  
### <a name="error-the-input-vmname-parameter-does-not-resolve-to-any-virtual-machine"></a>Fehler: Der Parameter „VMName“ kann nicht in einen virtuellen Computer aufgelöst werden.

**Fehlermeldung:**  
```
Enter-PSSession : The input VMName parameter does not resolve to any virtual machine.
```

**Mögliche Ursachen:**  
* Sie sind kein Hyper-V-Administrator.  
* Der virtuelle Computer ist nicht vorhanden.

Mit dem Cmdlet [Get-VM](https://docs.microsoft.com/powershell/module/hyper-v/get-vm?view=win10-ps) können Sie überprüfen, ob die von Ihnen verwendeten Anmeldeinformationen die Rolle „Hyper-V-Administrator“ enthalten und welche VMs lokal auf dem Host ausgeführt werden und gestartet wurden.


-------------

## <a name="samples-and-user-guides"></a>Beispiele und Benutzerleitfäden

PowerShell Direct unterstützt JEA (Just Enough Administration).  Informationen zum Ausprobieren finden Sie in diesem Benutzerleitfaden.

Sehen Sie sich die Beispiele auf [GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93) an.
