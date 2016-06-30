---
title: Verwaltung von Containerressourcen
description: Verwalten von Containerressourcen mit Windows-Containern.
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: b2192e64-9d74-474e-8af0-2d8b3ad1deee
translationtype: Human Translation
ms.sourcegitcommit: cfa3c14e932f8b86edf6667200ac028ea0a16b67
ms.openlocfilehash: 82cc37e4bcf001e938dcff7308be16978fa955e2

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


<!--HONumber=Jun16_HO4-->


