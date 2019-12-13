---
title: Druck Spooler in Windows-Containern
description: Erläutert das aktuelle Arbeitsverhalten für den Druckspoolerdienst in Windows-Containern.
keywords: docker, Container, Drucker, Spooler
author: cwilhit
ms.openlocfilehash: 48130bc6a826a45dfa49d0a3b4600d227f34704e
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910530"
---
# <a name="print-spooler-in-windows-containers"></a>Druck Spooler in Windows-Containern

Anwendungen mit einer Abhängigkeit von Druckdiensten können mit Windows-Containern erfolgreich in den Container integriert werden. Es gibt spezielle Anforderungen, die erfüllt sein müssen, damit die Drucker Dienst Funktionalität erfolgreich aktiviert werden kann. In diesem Handbuch wird erläutert, wie Sie die Bereitstellung ordnungsgemäß konfigurieren.

> [!IMPORTANT]
> Obwohl der Zugriff auf Druckdienste in Containern erfolgreich funktioniert, ist die Funktionalität in Form von eingeschränkt. Einige druckbezogene Aktionen funktionieren möglicherweise nicht. Beispielsweise kann es sein, dass apps, die von der Installation von Druckertreibern in den Host abhängig sind, nicht in den Container integriert werden, da die **Treiberinstallation aus einem Container nicht unterstützt wird**. Öffnen Sie unten ein Feedback, wenn Sie ein nicht unterstütztes Druck Feature finden, das in Containern unterstützt werden soll.

## <a name="setup"></a>Setup

* Der Host sollte Windows Server 2019 oder Windows 10 pro/Enterprise-Update vom Oktober 2018 oder neuer sein.
* Das [MCR.Microsoft.com/Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows) -Bild sollte das Zielbild sein. Andere Windows-Container-Basis Images (z. b. Nano Server und Windows Server Core) enthalten nicht die Druck Server Rolle.

### <a name="hyper-v-isolation"></a>Hyper-V-Isolierung

Es wird empfohlen, den Container mit der Hyper-V-Isolation zu betreiben. Wenn Sie in diesem Modus ausführen, können Sie über so viele Container verfügen, wie Sie mit Zugriff auf die Druckdienste ausführen möchten. Sie müssen den Spoolerdienst auf dem Host nicht ändern.

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

### <a name="process-isolation"></a>Prozessisolation

Aufgrund der gemeinsamen Kernel Natur von Prozess isolierten Containern beschränkt das aktuelle Verhalten den Benutzer auf die Ausführung von nur **einer Instanz** des Druckerspoolerdiensts auf dem Host und allen zugehörigen untergeordneten Containern. Wenn der Druckerspooler auf dem Host ausgeführt wird, müssen Sie den Dienst auf dem Host anhalten, bevor Sie den Drucker Dienst im Gast starten.

> [!TIP]
> Wenn Sie einen Container starten und den Spoolerdienst sowohl im Container als auch im Host gleichzeitig Abfragen, melden beide den Zustand "wird ausgeführt". Aber nicht täuschen: der Container kann keine Liste der verfügbaren Drucker Abfragen. Der Spoolerdienst des Hosts darf nicht ausgeführt werden. 

Um zu überprüfen, ob auf dem Host der Drucker Dienst ausgeführt wird, verwenden Sie die folgende Abfrage in PowerShell:

```PowerShell
PS C:\Users\Administrator> Get-Service spooler

Status   Name               DisplayName
------   ----               -----------
Running  spooler            Print Spooler

PS C:\Users\Administrator>
```

Verwenden Sie die folgenden Befehle in PowerShell, um den Spoolerdienst auf dem Host zu unterbinden:

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