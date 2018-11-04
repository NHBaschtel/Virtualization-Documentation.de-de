---
title: Hyper-V-Container
description: Erklärung der Unterschiede zwischen Hyper-V-Container und Prozess-Container.
keywords: Docker, Container
author: scooley
ms.date: 09/13/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: e1a5b80773128af0ba0095d5201e4fa123a1741c
ms.sourcegitcommit: 99da24a8c075e0096eabd09a29007a65e3ea35b7
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/04/2018
ms.locfileid: "6022178"
---
# <a name="hyper-v-containers"></a>Hyper-V-Container

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

Die Windows-containertechnologie umfasst zwei verschiedene Arten von Containern: Windows Server-Container (Prozess-Container) und Hyper-V-Container. Beide Arten von Containern werden auf gleiche Weise erstellt und verwaltet und funktionieren identisch. Sie erzeugen und nutzen auch die gleichen Containerimages. Der Unterschied besteht im Isolationsgrad, der zwischen dem Container, dem Hostbetriebssystem und allen anderen Containern besteht, die auf diesem Host ausgeführt werden.

**Windows Server-Container**: Mehrere Containerinstanzen können auf einem Host gleichzeitig isoliert ausgeführt werden, was mithilfe von Technologien zur Isolation von Namespaces, Ressourcensteuerung und Prozessen ermöglicht wird.  Windows Server-Container teilen sich untereinander und mit dem Host den gleichen Kernel.  Dies ist der gleiche etwa, wie unter Linux-Container ausgeführt.

**Hyper-V-Container** – mehrere containerinstanzen können auf einem Host gleichzeitig ausgeführt, jedoch jeder Container auf einem speziellen virtuellen Computer ausgeführt wird. Dadurch wird eine Isolation auf Kernelebene zwischen jedem Hyper-V-Container und dem Containerhost erreicht.

## <a name="hyper-v-container-examples"></a>Hyper-V-Container-Beispiele

### <a name="create-container"></a>Erstellen eines Containers

Verwalten von Hyper-V-Containern mit Docker ist nahezu identisch mit der Verwaltung von Windows Server-Container. Verwenden Sie zum Erstellen eines Hyper-V-Containers mit Docker der `--isolation` Parameter festlegen `--isolation=hyperv`.

``` cmd
docker run -it --isolation=hyperv microsoft/nanoserver cmd
```

### <a name="isolation-explanation"></a>Erläuterung zur Isolation

Dieses Beispiel veranschaulicht die Unterschiede im netzwerkisolationsfunktionen zwischen Windows Server und Hyper-V-Container. 

Hier wird ein Windows Server-Container bereitgestellt und hostet einen Pingprozess mit langer Ausführungsdauer.

``` cmd
docker run -d microsoft/windowsservercore ping localhost -t
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

Dieses Beispiel startet ebenfalls einen Hyper-V-Container mit einem Pingprozess. 

```
docker run -d --isolation=hyperv microsoft/nanoserver ping -t localhost
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
