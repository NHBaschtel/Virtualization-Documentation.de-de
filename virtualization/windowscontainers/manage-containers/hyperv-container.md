---
title: Isolations Modi
description: Erläuterung der Unterschiede zwischen Hyper-V-Isolation und isolierten Prozess Containern.
keywords: Docker, Container
author: crwilhit
ms.date: 09/26/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 42154683-163b-47a1-add4-c7e7317f1c04
ms.openlocfilehash: fa95ffe1c699a2c837076fcc1b662f6b792b7dfb
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909750"
---
# <a name="isolation-modes"></a>Isolations Modi

Windows-Container bieten zwei unterschiedliche Modi für die Lauf Zeit Isolation: `process`-und `Hyper-V`-Isolation. Container, die in beiden Isolations Modi ausgeführt werden, werden gleich erstellt, verwaltet und funktionieren identisch. Sie erzeugen und nutzen auch die gleichen Containerimages. Der Unterschied zwischen den Isolations Modi besteht darin, welcher Isolations Grad zwischen dem Container, dem Host Betriebssystem und allen anderen Containern, die auf diesem Host ausgeführt werden, erstellt wird.

## <a name="process-isolation"></a>Prozessisolation

Dies ist der "herkömmliche" Isolations Modus für Container, der in der [Übersicht über Windows-Container](../about/index.md)beschrieben wird. Bei der Prozess Isolation werden mehrere Container Instanzen gleichzeitig auf einem angegebenen Host ausgeführt, wobei die Isolation durch Namespace-, Ressourcen Steuerungs-und Prozess Isolations Technologien gewährleistet ist. Bei der Ausführung in diesem Modus verwenden Container denselben Kernel gemeinsam mit dem Host und einander.  Dies entspricht ungefähr der Art und Weise, wie Linux-Container ausgeführt werden.

![](media/container-arch-process.png)

## <a name="hyper-v-isolation"></a>Hyper-V-Isolierung
Dieser Isolations Modus bietet verbesserte Sicherheit und breitere Kompatibilität zwischen Host-und Container Versionen. Bei der Hyper-V-Isolation werden mehrere Container Instanzen gleichzeitig auf einem Host ausgeführt. Jeder Container wird jedoch in einem hochgradig optimierten virtuellen Computer ausgeführt und erhält seinen eigenen Kernel. Das vorhanden sein des virtuellen Computers ermöglicht die Isolierung von Hardware Ebenen zwischen den einzelnen Containern und dem Container Host.

![](media/container-arch-hyperv.png)

## <a name="isolation-examples"></a>Beispiele für Isolation

### <a name="create-container"></a>Erstellen eines Containers

Das Verwalten von Hyper-V-isolierten Containern mit docker ist nahezu identisch mit der Verwaltung von Prozess isolierten Containern. Verwenden Sie zum Erstellen eines Containers mit der Hyper-V-Isolation mit gründlicher docker den `--isolation`-Parameter, um `--isolation=hyperv`festzulegen.

```cmd
docker run -it --isolation=hyperv mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Um einen Container mit gründlicher Prozess Isolation zu erstellen, verwenden Sie den `--isolation`-Parameter, um `--isolation=process`festzulegen.

```cmd
docker run -it --isolation=process mcr.microsoft.com/windows/servercore:ltsc2019 cmd
```

Windows-Container, die unter Windows Server ausgeführt werden, werden standardmäßig mit Prozess Isolation ausgeführt. Windows-Container unter Windows 10 pro und Enterprise werden standardmäßig mit der Hyper-V-Isolation ausgeführt. Ab dem Windows 10-Update vom Oktober 2018 können Benutzer, die einen Windows 10 pro-oder Enterprise-Host ausführen, einen Windows-Container mit Prozess Isolation ausführen. Benutzer müssen die Prozess Isolation direkt mit dem `--isolation=process`-Flag anfordern.

> [!WARNING]
> Die Ausführung mit Prozess Isolation unter Windows 10 pro und Enterprise ist für Entwicklung/Tests gedacht. Auf dem Host muss Windows 10 Build 17763 + ausgeführt werden, und Sie müssen über eine docker-Version mit der Engine 18,09 oder höher verfügen.
> 
> Sie sollten weiterhin Windows Server als Host für Produktions Bereitstellungen verwenden. Wenn Sie dieses Feature unter Windows 10 pro und Enterprise verwenden, müssen Sie auch sicherstellen, dass die Host-und Container Versions Tags Stimmen, da andernfalls der Container nicht gestartet werden kann oder nicht definiertes Verhalten aufweist.

### <a name="isolation-explanation"></a>Erläuterung zur Isolation

In diesem Beispiel werden die Unterschiede in den Isolations Funktionen zwischen Prozess-und Hyper-V-Isolation veranschaulicht.

Hier wird ein Prozess isolierter Container bereitgestellt, der einen Ping-Prozess mit langer Laufzeit hostet.

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

Im Gegensatz dazu wird in diesem Beispiel auch ein Hyper-V-solated-Container mit einem Ping-Prozess gestartet.

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
