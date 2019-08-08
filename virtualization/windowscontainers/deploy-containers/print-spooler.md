---
title: Druckspooler in Windows-Containern
description: Erläutert das aktuelle Arbeitsverhalten für den Druckspooler-Dienst in Windows-Containern
keywords: docker, Container, Drucker, Spooler
author: cwilhit
ms.openlocfilehash: e104a87046545b90d244783aafb62ad9d151e14b
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999097"
---
# <a name="print-spooler-in-windows-containers"></a>Druckspooler in Windows-Containern

Anwendungen mit einer Abhängigkeit von Druckdiensten können mit Windows-Containern erfolgreich Containern werden. Es gibt spezielle Anforderungen, die erfüllt werden müssen, um die Drucker Dienstfunktionalität erfolgreich zu aktivieren. In diesem Leitfaden wird erläutert, wie Sie Ihre Bereitstellung richtig konfigurieren.

> [!IMPORTANT]
> Wenn der Zugriff auf die Druckdienste erfolgreich in Containern funktioniert, ist die Funktionalität in der Form limitiert; Einige druckbezogene Aktionen funktionieren möglicherweise nicht. Apps, die eine Abhängigkeit von der Installation von Druckertreibern in den Host haben, können beispielsweise nicht containeriert werden, da die **Treiberinstallation innerhalb eines Containers nicht unterstützt wird**. Bitte öffnen Sie unten ein Feedback, wenn Sie eine nicht unterstützte Druckfunktion finden, die in Containern unterstützt werden soll.

## <a name="setup"></a>Setup

* Der Host sollte Windows Server 2019 oder Windows 10 pro/Enterprise Oktober 2018 Update oder höher sein.
* Das [MCR.Microsoft.com/Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) -Bild sollte das Ziel-Basis Bild sein. Andere Windows-Container-Basisbilder (wie Nano-Server und Windows Server Core) tragen nicht die Druck Server Rolle.

### <a name="hyper-v-isolation"></a>Hyper-V-Isolierung

Wir empfehlen, den Container mit Hyper-V-Isolierung auszuführen. Wenn Sie in diesem Modus ausgeführt werden, können Sie mit dem Zugriff auf die Druckdienste so viele Container wie gewünscht ausführen. Sie müssen den Spooler-Dienst auf dem Host nicht ändern.

Sie können die Funktionalität mit der folgenden PowerShell-Abfrage überprüfen:

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

### <a name="process-isolation"></a>Prozessisolierung

Aufgrund des Shared-Kernel-Charakters von Prozess isolierten Containern schränkt das aktuelle Verhalten den Benutzer ein, dass nur **eine Instanz** des Druckerspooler-Diensts über den Host und alle zugehörigen Container-untergeordneten Elemente ausgeführt wird. Wenn der Druckerspooler auf dem Host ausgeführt wird, müssen Sie den Dienst auf dem Host beenden, bevor Sie Attemping, um den Drucker Dienst im Gast zu starten.

> [!TIP]
> Wenn Sie einen Container starten und den Spooler-Dienst sowohl im Container als auch im Host gleichzeitig Abfragen, werden beide ihren Status als "Running" melden. Lassen Sie sich aber nicht täuschen – der Container kann keine Liste der verfügbaren Drucker Abfragen. Der Spooler-Dienst des Hosts darf nicht ausgeführt werden. 

Um zu überprüfen, ob der Host den Drucker Dienst ausführt, verwenden Sie die Abfrage in PowerShell unten:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Wenn Sie den Spooler-Dienst auf dem Host beenden möchten, verwenden Sie die folgenden Befehle in PowerShell unten:

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