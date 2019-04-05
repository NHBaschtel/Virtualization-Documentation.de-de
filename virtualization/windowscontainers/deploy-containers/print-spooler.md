---
title: Druckspooler in Windows-Containern
description: Erläutert die aktuelle arbeiten Verhalten für den Druckspooler-Dienst in Windows-Container
keywords: Docker, Container, Drucker, spoolerprozess
author: cwilhit
ms.openlocfilehash: 48130bc6a826a45dfa49d0a3b4600d227f34704e
ms.sourcegitcommit: 3c81b0efd1ac2c4c93d58f16edae1044c9a5ad55
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/05/2019
ms.locfileid: "9284574"
---
# <a name="print-spooler-in-windows-containers"></a>Druckspooler in Windows-Containern

Anwendungen mit einer Abhängigkeit Druck-Dienste können erfolgreich mit Windows-Containern in Containern werden. Gibt es spezielle Anforderungen, die erfüllt sein müssen, um erfolgreich aktivieren Drucker-Service-Funktion. Dieses Handbuch erklärt, wie Sie Ihre Bereitstellung ordnungsgemäß konfigurieren.

> [!IMPORTANT]
> Während der ersten Zugriff auf das Drucken erfolgreich in Containern funktioniert services, ist die Funktionalität in Form eingeschränkt. Einige Aktionen druckbezogener funktioniert möglicherweise nicht. Z. B. apps, die eine Abhängigkeit von Installieren von Druckertreibern in der Host haben können nicht werden in Containern da **Treiberinstallation aus innerhalb eines Containers wird nicht unterstützt**. Öffnen Sie unten Feedback, wenn Sie eine nicht unterstützte Druckfunktion finden, die in Containern unterstützt werden sollen.

## <a name="setup"></a>Setup

* Der Host sollte sein, Windows Server 2019 oder Windows 10 Pro/Enterprise October 2018 update oder höher.
* Das Bild [mcr.microsoft.com/windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) sollte die gezielte Basis-Image. Andere Windows-Container-Basisimages (z. B. Nano Server und Windows Server Core) nicht die Serverrolle drucken vorgenommen werden.

### <a name="hyper-v-isolation"></a>Hyper-V-Isolierung

Es wird empfohlen, den Container mit Hyper-V-Isolierung ausgeführt. Wenn in diesem Modus ausgeführt wird, haben Sie möglichst viele Container wie ausgeführt mit Zugriff auf die Dienste gewünscht. Sie müssen nicht den spoolerprozess-Dienst auf dem Host ändern.

Sie können die Funktion mit dem folgenden PowerShell-Abfrage überprüfen:

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation hyperv mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```

### <a name="process-isolation"></a>Prozessisolation

Aufgrund der Natur gemeinsamer Kernel der Prozess-isolierten Container beschränkt aktuelles Verhalten den Benutzer nur **eine Instanz** des Dienstes spoolerprozess Drucker auf dem Host und alle Container untergeordneten ausgeführt. Wenn der Host die Drucker spoolerprozess ausgeführt hat, müssen Sie den Dienst auf dem Host vor Attemping zum Starten des Drucker-Diensts auf dem Gast beenden.

> [!TIP]
> Wenn Sie eines Containers starten und der spoolerprozess-Dienst im Container und dem Host gleichzeitig Abfragen, meldet beide ihren Zustand als "Running". Jedoch nicht deceived – des Containers wird nicht in der Lage, um eine Liste der verfügbaren Drucker abzufragen. Der Host spoolerprozess Dienst muss nicht ausgeführt werden. 

Um zu überprüfen, ob der Host den Drucker-Dienst ausgeführt wird, verwenden Sie die Abfrage in PowerShell folgenden:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Um die auf dem Host beenden, verwenden Sie die folgenden Befehle in PowerShell folgenden:

```PowerShell
Stop-Service spooler
Set-Service spooler -StartupType Disabled
```

Starten Sie den Container, und überprüfen Sie den Zugriff auf die Drucker.

```PowerShell
PS C:\Users\Administrator> docker run -it --isolation process mcr.microsoft.com/windows:1809 powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.


PS C:\> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler


PS C:\> Get-Printer

Name                           ComputerName    Type         DriverName                PortName        Shared   Published
----                           ------------    ----         ----------                --------        ------   --------
Microsoft XPS Document Writer                  Local        Microsoft XPS Document... PORTPROMPT:     False    False
Microsoft Print to PDF                         Local        Microsoft Print To PDF    PORTPROMPT:     False    False
Fax                                            Local        Microsoft Shared Fax D... SHRFAX:         False    False


PS C:\>
```