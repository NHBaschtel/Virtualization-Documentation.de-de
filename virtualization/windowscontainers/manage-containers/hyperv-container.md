---
title: Hyper-V-Isolierung
description: Erklärung der Unterschiede zwischen Hyper-V-Isolierung von Prozess isolierte Container.
keywords: Docker, Container
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: 4ab473c1752c377955bb23bdf6c9ef83a3336aa8
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380124"
---
# <a name="hyper-v-isolation"></a>Hyper-V-Isolierung

Die Windows-containertechnologie umfasst zwei unterschiedliche Ebenen der Isolation für Container, Prozess und Hyper-V-Isolierung. Beide Arten werden erstellt und verwaltet und funktionieren identisch. Sie erzeugen und nutzen auch die gleichen Containerimages. Der Unterschied besteht im Isolationsgrad, der zwischen dem Container, dem Hostbetriebssystem und allen anderen Containern besteht, die auf diesem Host ausgeführt werden.

**Prozessisolation** – mehrere Instanzen gleichzeitig auf einem Host mit Isolierung ausgeführt werden können Containerelemente Technologien, Namespaces, ressourcensteuerung und Prozessisolation bereitgestellt.  Container teilen sich den gleichen Kernel mit dem Host als auch miteinander.  Dies entspricht etwa wie unter Linux-Container ausgeführt.

**Hyper-V-Isolierung** – mehrere containerinstanzen können auf einem Host gleichzeitig ausgeführt, jedoch jeder Container in einem speziellen virtuellen Computer ausgeführt wird. Dies bietet Isolation auf Kernelebene zwischen jedem Container als auch die Container-Host.

## <a name="hyper-v-isolation-examples"></a>Hyper-V-Isolierung-Beispiele

### <a name="create-container"></a>Erstellen eines Containers

Verwalten von isolierte Hyper-V-Containern mit Docker ist nahezu identisch mit der Verwaltung von Windows Server-Container. Um einen Container mit Hyper-V-Isolierung erstellen gründliche Docker, verwenden die `--isolation` Parameter fest `--isolation=hyperv`.

``` cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1809 cmd
```

### <a name="isolation-explanation"></a>Erläuterung zur Isolation

Dieses Beispiel veranschaulicht die Unterschiede im netzwerkisolationsfunktionen zwischen Windows Server und Hyper-V-Isolierung.

Hier ein Prozess isolierten Container bereitgestellt und hostet einen pingprozess langer Ausführungsdauer.

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

Im Gegensatz dazu wird in diesem Beispiel wird einen isolierten Hyper-V-Container mit einem pingprozess sowie gestartet.

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
