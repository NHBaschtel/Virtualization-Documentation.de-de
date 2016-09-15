---
title: Verwaltung von Containerressourcen
description: Verwalten von Containerressourcen mit Windows-Containern.
keywords: Docker, Container
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b2192e64-9d74-474e-8af0-2d8b3ad1deee
redirect_url: https://docs.docker.com/engine/reference/run/
translationtype: Human Translation
ms.sourcegitcommit: 59626096d428072dec098c7817e2d6b39c10e9cf
ms.openlocfilehash: 7544e91856d969b09241df7a466d776d818a2968

---

# Verwaltung von Containerressourcen

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

Für Windows-Container kann konfiguriert werden, wie viele CPU-, Datenträger-E/A-, Netzwerk- und Arbeitsspeicherressourcen ihnen zur Verfügung stehen. Das Einschränken von Containerressourcen ermöglicht eine effiziente Nutzung von Hostressourcen und verhindert eine übermäßige Nutzung. Dieses Dokument bietet ausführliche Informationen zum Verwalten von Containerressourcen mit Docker.

## Verwalten von Ressourcen mit Docker 

Wir bieten die Möglichkeit, eine Teilmenge der Containerressourcen mit Docker zu verwalten. Insbesondere ermöglichen wir Benutzern die Angabe, wie viel der CPU-Kapazität für Container freigegeben wird. 

### CPU

CPU-Freigaben unter Containern können zur Laufzeit mithilfe des Kennzeichens „--cpu-shares“ verwaltet werden. Standardmäßig steht allen Container ein gleich großer Anteil der CPU-Zeit zur Verfügung. Um die relative CPU-Freigabe für Container zu ändern, geben Sie für das Kennzeichen „--cpu-shares“ einen Wert von 1 bis 10.000 an. Standardmäßig erhalten alle Container eine Gewichtung von 5000. Weitere Informationen zur CPU-Freigabeeinschränkung finden Sie in der [Referenz zu „Docker Run“]( https://docs.docker.com/engine/reference/run/#cpu-share-constraint). 

```none 
docker run -it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## Bekannte Probleme

- Für Hyper-V-Container wird die Steuerung von CPU- und E/A-Ressourcen derzeit nicht unterstützt.
- Die Steuerung von E/A-Ressourcen wird derzeit für freigegebene Containerdatenvolumes nicht unterstützt.


<!--HONumber=Sep16_HO2-->


