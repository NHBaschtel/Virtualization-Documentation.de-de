---
title: Isolationsmodi
description: Erläuterung, wie sich die Hyper-V-Isolierung von isolierten Prozess Containern unterscheidet.
keywords: Docker, Container
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: fa95ffe1c699a2c837076fcc1b662f6b792b7dfb
ms.sourcegitcommit: e9dda81f1f68359ece9ef132a184a30880bcdb1b
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 09/27/2019
ms.locfileid: "10161727"
---
# <a name="isolation-modes"></a>Isolationsmodi

Windows-Container bieten zwei unterschiedliche Modi zur Laufzeit `process` - `Hyper-V` Isolierung: und Isolierung. Container, die unter beiden Isolationsmodi ausgeführt werden, werden erstellt, verwaltet und funktionieren identisch. Sie erzeugen und nutzen auch die gleichen Containerimages. Der Unterschied zwischen den Isolationsmodi besteht in dem Grad der Isolierung zwischen dem Container, dem Hostbetriebssystem und allen anderen Containern, die auf diesem Host ausgeführt werden.

## <a name="process-isolation"></a>Prozessisolierung

Dies ist der "herkömmliche" Isolationsmodus für Container und wird in der [Windows Containers-Übersicht](../about/index.md)beschrieben. Bei der Prozessisolierung werden mehrere Container Instanzen gleichzeitig auf einem bestimmten Host ausgeführt, wobei die Isolierung über Namespace-, Ressourcen Steuerungs-und Prozess Isolierungs Technologien bereitgestellt wird. Bei der Ausführung in diesem Modus verwenden Container denselben Kernel mit dem Host und einander.  Dies ist ungefähr so, wie Linux-Container ausgeführt werden.

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Hyper-V-Isolierung
Dieser Isolationsmodus bietet erhöhte Sicherheit und umfassendere Kompatibilität zwischen Host-und Container Versionen. Bei der Hyper-V-Isolierung werden mehrere Container Instanzen gleichzeitig auf einem Host ausgeführt. Jeder Container läuft jedoch innerhalb einer hoch optimierten virtuellen Maschine und erhält effektiv seinen eigenen Kernel. Das vorhanden sein des virtuellen Computers bietet Isolierung auf Hardwareebene zwischen den einzelnen Containern und dem Container Host.

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>Beispiele für die Isolierung

### <a name="create-container"></a>Erstellen eines Containers

Das Verwalten von Hyper-V-isolierten Containern mit docker ist nahezu identisch mit der Verwaltung Prozess isolierter Container. Verwenden Sie zum Erstellen eines Containers mit vollständiger Hyper-V- `--isolation` Isolierung den Parameter `--isolation=hyperv`für die Einstellung.

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Verwenden Sie zum Erstellen eines Containers mit der `--isolation` Prozessisolierung eine vollständige Andock `--isolation=process`Einrichtung.

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Windows-Container, die unter Windows Server ausgeführt werden, werden standardmäßig mit der Prozessisolierung ausgeführt. Windows-Container, die unter Windows 10 pro und Enterprise ausgeführt werden, werden standardmäßig mit Hyper-V-Isolierung ausgeführt. Beginnend mit dem 2018-Update für Windows 10 Oktober können Benutzer, die einen Windows 10 pro-oder Enterprise-Host ausführen, einen Windows-Container mit Prozessisolierung ausführen. Benutzer müssen die Prozessisolierung direkt mithilfe der `--isolation=process` Kennzeichnung anfordern.

> [!WARNING]
> Das Ausführen mit der Prozessisolierung unter Windows 10 pro und Enterprise ist für die Entwicklung/Testung vorgesehen. Auf Ihrem Host muss Windows 10 Build 17763 + ausgeführt werden, und Sie müssen über eine Andock Version mit Engine 18,09 oder höher verfügen.
> 
> Sie sollten Windows Server weiterhin als Host für Produktionsbereitstellungen verwenden. Wenn Sie dieses Feature unter Windows 10 pro und Enterprise verwenden, müssen Sie auch sicherstellen, dass Ihre Host-und Container Versions Tags übereinstimmen, da der Container andernfalls möglicherweise nicht gestartet wird oder nicht definiertes Verhalten aufweist.

### <a name="isolation-explanation"></a>Erläuterung zur Isolation

In diesem Beispiel werden die Unterschiede bei den Isolierungsfunktionen zwischen Prozess und Hyper-V-Isolierung veranschaulicht.

Hier wird ein Prozess isolierter Container bereitgestellt, der einen lang andauernden Ping-Prozess hostet.

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

Mithilfe des `docker top`-Befehls wird der Pingprozess zurückgegeben, wie im Container zu sehen. Der Prozess in diesem Beispiel weist die ID 3964 auf.

``` cmd
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

Auf dem Containerhost kann der `get-process`-Befehl verwendet werden, um einen beliebigen ausgeführten Pingprozess vom Host zurückzugeben. In diesem Beispiel ist ein solcher Prozess vorhanden, und die Prozess-ID entspricht der ID im Container. Es handelt sich um den gleichen Prozess, der sowohl im Container als auch auf dem Host sichtbar ist.

```
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

In diesem Beispiel wird ein Hyper-V-solated-Container mit einem pingprozess ebenfalls gestartet.

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 ping localhost -t
```

Ebenso kann `docker top` verwendet werden, um die ausgeführten Prozesse vom Container zurückzugeben.

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

Bei der Suche nach dem Prozess auf dem Container Host wird jedoch kein Ping-Prozess gefunden, und es wird ein Fehler ausgelöst.

```
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

Auf dem Host ist der Prozess `vmwp` sichtbar, bei dem es sich um den ausgeführten virtuellen Computer handelt, der den ausgeführten Container kapselt und die ausgeführten Prozesse vor dem Hostbetriebssystem schützt.

```
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```
