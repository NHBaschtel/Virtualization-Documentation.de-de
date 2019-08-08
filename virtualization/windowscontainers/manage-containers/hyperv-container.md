---
title: Hyper-V-Isolierung
description: Erläuterung der Unterschiede zwischen Hyper-V-Isolierung und isolierten Prozess Containern
keywords: Docker, Container
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 092312848173102bec5a791f2c48fe8166e70d5f
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998327"
---
# <a name="hyper-v-isolation"></a>Hyper-V-Isolierung

Die Windows-Container Technologie umfasst zwei unterschiedliche Isolationsstufen für Container, Prozess und Hyper-V-Isolierung. Beide Typen werden erstellt, verwaltet und funktionieren identisch. Sie erzeugen und nutzen auch die gleichen Containerimages. Der Unterschied besteht im Isolationsgrad, der zwischen dem Container, dem Hostbetriebssystem und allen anderen Containern besteht, die auf diesem Host ausgeführt werden.

**Prozessisolierung** : mehrere Container Instanzen können gleichzeitig auf einem Host ausgeführt werden, wobei die Isolierung über Namespace, Ressourcensteuerung und Prozess Isolierungs Technologien bereitgestellt wird.  Container verwenden denselben Kernel sowohl für den Host als auch für die anderen.  Dies entspricht ungefähr dem, wie Container unter Linux ausgeführt werden.

**Hyper-V-Isolierung** : mehrere Container Instanzen können gleichzeitig auf einem Host ausgeführt werden, jedoch wird jeder Container innerhalb einer speziellen virtuellen Maschine ausgeführt. Dadurch wird die Isolierung auf Kernelebene zwischen den einzelnen Containern und dem Container Host bereitgestellt.

## <a name="hyper-v-isolation-examples"></a>Beispiele für die Hyper-V-Isolierung

### <a name="create-container"></a>Erstellen eines Containers

Das Verwalten von isolierten Hyper-V-Containern mit docker ist mit der Verwaltung von Windows Server-Containern nahezu identisch. Verwenden Sie zum Erstellen eines Containers mit vollständiger Hyper-V- `--isolation` Isolierung den Parameter `--isolation=hyperv`für die Einstellung.

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>Erläuterung zur Isolation

In diesem Beispiel werden die Unterschiede bei den Isolierungsfunktionen zwischen Windows Server und Hyper-V-Isolierung veranschaulicht.

Hier wird ein Prozess isolierter Container bereitgestellt, der einen lang andauernden Ping-Prozess hostet.

``` cmd
docker run -d mcr.microsoft.com/windows/servercore:1809 ping localhost -t
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

In diesem Beispiel wird jedoch auch ein isolierter Hyper-V-Container mit einem Ping-Vorgang gestartet.

```
docker run -d --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 ping -t localhost
```

Ebenso kann `docker top` verwendet werden, um die ausgeführten Prozesse vom Container zurückzugeben.

```
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

Beim Suchen nach dem Prozess auf dem Containerhost wird jedoch kein Pingprozess gefunden, und es wird ein Fehler ausgegeben.

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
