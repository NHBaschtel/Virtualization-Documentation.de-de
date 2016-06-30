---
title: Hyper-V-Container
description: Bereitstellen von Hyper-V-Containern
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
translationtype: Human Translation
ms.sourcegitcommit: 18571fc02ecbb37c5db596a610124727f6d5bb90
ms.openlocfilehash: 2677b9228371b4a4ba72c249509cd9ccacc89473

---

# Hyper-V-Container

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

Die Windows-Containertechnologie umfasst zwei Arten von Containern: Windows Server-Container und Hyper-V-Container. Beide Arten von Containern werden auf gleiche Weise erstellt und verwaltet und funktionieren identisch. Sie erzeugen und nutzen auch die gleichen Containerimages. Der Unterschied besteht im Isolationsgrad, der zwischen dem Container, dem Hostbetriebssystem und allen anderen Containern besteht, die auf diesem Host ausgeführt werden.

**Windows Server-Container**: Mehrere Containerinstanzen können auf einem Host gleichzeitig isoliert ausgeführt werden, was mithilfe von Technologien zur Isolation von Namespaces, Ressourcensteuerung und Prozessen ermöglicht wird.  Windows Server-Container teilen sich untereinander und mit dem Host den gleichen Kernel.

**Hyper-V-Container**: Mehrere Containerinstanzen können auf einem Host gleichzeitig ausgeführt werden, jeder Container wird jedoch auf einem speziellen virtuellen Computer ausgeführt. Dadurch wird eine Isolation auf Kernelebene zwischen jedem Hyper-V-Container und dem Containerhost erreicht.

## Hyper-V-Container

### Erstellen eines Containers

Die Verwaltung von Hyper-V-Containern mit Docker ist nahezu identisch mit der Verwaltung von Windows Server-Containern. Beim Erstellen eines Hyper-V-Containers mit Docker wird der Parameter`--isolation=hyperv` verwendet.

```none
docker run -it --isolation=hyperv nanoserver cmd
```

### Erläuterung zur Isolation

Dieses Beispiel stellt die Isolationsfähigkeiten von Windows- und Hyper-V-Containern gegenüber. 

Hier wird ein Windows Server-Container bereitgestellt und hostet einen Pingprozess mit langer Ausführungsdauer.

```none
docker run -d windowsservercore ping localhost -t
```

Mithilfe des `docker top`-Befehls wird der Pingprozess zurückgegeben, wie im Container zu sehen. Der Prozess in diesem Beispiel weist die ID 3964 auf.

```none
docker top 1f8bf89026c8f66921a55e773bac1c60174bb6bab52ef427c6c8dbc8698f9d7a

3964 ping
```

Auf dem Containerhost kann der `get-process`-Befehl verwendet werden, um einen beliebigen ausgeführten Pingprozess vom Host zurückzugeben. In diesem Beispiel ist ein solcher Prozess vorhanden, und die Prozess-ID entspricht der ID im Container. Es handelt sich um den gleichen Prozess, der sowohl im Container als auch auf dem Host sichtbar ist.

```none
get-process -Name ping

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
     67       5      820       3836 ...71     0.03   3964   3 PING
```

Dieses Beispiel startet ebenfalls einen Hyper-V-Container mit einem Pingprozess. 

```none
docker run -d --isolation=hyperv nanoserver ping -t localhost
```

Ebenso kann `docker top` verwendet werden, um die ausgeführten Prozesse vom Container zurückzugeben.

```none
docker top 5d5611e38b31a41879d37a94468a1e11dc1086dcd009e2640d36023aa1663e62

1732 ping
```

Beim Suchen nach dem Prozess auf dem Containerhost wird jedoch kein Pingprozess gefunden, und es wird ein Fehler ausgegeben.

```none
get-process -Name ping

get-process : Cannot find a process with the name "ping". Verify the process name and call the cmdlet again.
At line:1 char:1
+ get-process -Name ping
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (ping:String) [Get-Process], ProcessCommandException
    + FullyQualifiedErrorId : NoProcessFoundForGivenName,Microsoft.PowerShell.Commands.GetProcessCommand
```

Auf dem Host ist der Prozess `vmwp` sichtbar, bei dem es sich um den ausgeführten virtuellen Computer handelt, der den ausgeführten Container kapselt und die ausgeführten Prozesse vor dem Hostbetriebssystem schützt.

```none
get-process -Name vmwp

Handles  NPM(K)    PM(K)      WS(K) VM(M)   CPU(s)     Id  SI ProcessName
-------  ------    -----      ----- -----   ------     --  -- -----------
   1737      15    39452      19620 ...61     5.55   2376   0 vmwp
```



<!--HONumber=Jun16_HO4-->


