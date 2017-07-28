---
title: 'Schnellstartanleitung: Windows-Container'
description: 'Schnellstartanleitung: Windows-Container.'
keywords: Docker, Container
author: enderb-ms
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-contianers
ms.service: windows-containers
ms.assetid: 4878f5d2-014f-4f3c-9933-97f03348a147
ms.openlocfilehash: fb97f1d0f533b28acfb711e52bd021b29212f66e
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 07/21/2017
---
# Schnellstartanleitung: Windows-Container

In der Schnellstartanleitung zu Windows-Containern werden die Produkt- und Containerterminologie und Schritte in einfachen Beispielen zur Containerbereitstellung vorgestellt. Außerdem enthält sie Verweise auf fortgeschrittene Themen. Wenn Sie noch keine Erfahrung mit Containern bzw. Windows-Containern haben, erhalten Sie in den einzelnen Schritten dieser Schnellstartanleitung praktische Tipps zum Umgang mit dieser Technologie.

## 1. Was sind Container?

Bei Containern handelt es sich um eine isolierte, von Ressourcen gesteuerte und portierbare Betriebsumgebung.

Im Grunde ist ein Container ein isolierter Ort, wo eine Anwendung ausgeführt werden kann, ohne den Rest des Systems zu beeinflussen, und ohne dass das System die Anwendung beeinflusst. Container sind der nächste Entwicklungsschritt bei der Virtualisierung.

Wenn Sie sich in einem Container befänden, sähe es dort so aus, als befänden Sie sich in einem frisch installierten physischen Container bzw. virtuellen Computer. Und mit [Docker](https://www.docker.com/) kann ein Windows-Container auf die gleiche Weise wie jeder andere Container verwaltet werden.

## 2. Windows-Containertypen

Windows-Container enthalten zwei verschiedene Containertypen bzw. Laufzeiten.

**Windows Server-Container**: – Bieten Anwendungsisolation mithilfe einer Technologie zum Isolieren von Prozessen und Namespaces. Ein Windows Servercontainer teilt sich einen Kernel mit dem Containerhost und allen Containern, die auf dem Host ausgeführt werden.  Diese Container bieten keine schädlichen Sicherheitsgrenzen und sollten nicht zum Isolieren von nicht vertrauenswürdigen Codes verwendet werden.  Diese Container teilen sich den Kernelspeicher mit dem Host und mit den anderen Containern auf demselben Host. Daher müssen die Kernel übereinstimmen, d.h. sie müssen die gleiche Version und Konfiguration haben.

**Hyper-V-Isolierung**: – Erweitert die von Windows Server-Containern bereitgestellte Isolation, indem jeder Container in einem hochgradig optimierten virtuellen Computer ausgeführt wird. Bei dieser Konfiguration wird der Kernel des Containerhosts nicht gemeinsam mit anderen Containern desselben Hosts verwendet.  Diese Container wurden für schädliches mandantenfähiges Hosting mit der gleichen Sicherheitsgarantie wie der eines virtuellen Computers entwickelt. Da diese Container den Kernel nicht mit dem Host oder mit anderen Containern auf dem Host teilen, können sie Kernel mit verschiedenen Versionen und Konfigurationen (in unterstützten Versionen) ausführen – z.B. wenn alle Windows-Container unter Windows10 Hyper-V-Isolation verwenden, um die Version und Konfiguration des Windows Server-Kernels zu nutzen.

## 3. Grundlegendes zu Containern

Beim Arbeiten mit Containern bemerken Sie viele ähnlichkeiten zwischen einem Container und einer virtuellen Maschine. Ein Container, in dem ein Betriebssystem ausgeführt wird, verfügt über ein Dateisystem, und auf ihn kann über ein Netzwerk so zugegriffen werden, als handele es sich um ein physisches oder virtuelles Computersystem. Dies bedeutet, dass die Technologie und die Konzepte hinter Container sehr von dem der virtuelle Computer unterscheiden. Die folgenden grundlegenden Konzepte können nützlich sein, wenn Sie beginnen, Windows-Container zu erstellen und damit zu arbeiten. 

**Containerhost:** – Physisches oder virtuelles Computersystem, das mit dem Feature „Windows-Container“ konfiguriert wurde.

**Containerbetriebssystem-Image:** – Container werden mithilfe von Images bereitgestellt. Das Betriebssystemabbild Container ist möglicherweise viele Bildebenen, aus denen ein Container der ersten Ebene. Grafik für das Betriebssystem.

**Containerimage:** – Ein Containerimage enthält das zugrunde liegende Betriebssystem, die Anwendung und alle Abhängigkeiten der Anwendung, die zur schnellen Bereitstellung eines Containers benötigt werden. 

**Containerregistrierung:** – Containerimages werden in einer Containerregistrierung gespeichert und können bei Bedarf heruntergeladen werden. 

**Dockerfile-Datei:** – Mit Dockerfile-Dateien wird die Erstellung von Containerimages automatisiert.

## Nächster Schritt:

[Windows Server-Container – Schnellstart](quick-start-windows-server.md)  

[Windows 10-Container – Schnellstart](quick-start-windows-10.md)

