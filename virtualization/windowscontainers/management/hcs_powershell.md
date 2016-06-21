---
title: HCS PowerShell
description: Arbeiten mit HCS PowerShell und Windows-Containern.
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 45144ec5-f76a-4460-abd1-9b60e47506d6
---

# Verwaltungsinteroperabilität

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

## Anzeigen aller Container

Verwenden Sie den Befehl `Get-ComputeProcess`, um eine Liste der Container zurückzugeben.

```none
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## Beenden eines Containers

Mit dem Befehl `Stop-ComputeProcess` können Sie einer Container beenden, unabhängig davon, ob er mit PowerShell oder Docker erstellt wurde.

> Zum Zeitpunkt der Verfassung dieses Artikels muss der VMMS-Dienst neu gestartet werden, damit bei Verwendung des Befehls `Get-Container` die Container als beendet angezeigt werden.

```none
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```


<!--HONumber=May16_HO3-->


