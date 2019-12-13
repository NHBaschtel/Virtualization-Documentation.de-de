---
title: Lebenszyklus der Basis Image Wartung
description: Informationen zum Lebenszyklus des Windows-Containerbasis Images.
keywords: Windows-Container, Container, Lebenszyklus, Releaseinformationen, Basis Image, Containerbasis Image
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: bb5e5fabadde421de9d420edd2fc921457432930
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909990"
---
# <a name="base-image-servicing-lifecycles"></a>Lebenszyklus der Basis Image Wartung

Windows-Container-Basis Images basieren entweder auf halbjährlichen Kanal Releases oder auf langfristigen Wartungs Kanälen von Windows Server. In diesem Artikel erfahren Sie, wie lange der Support für verschiedene Versionen von Basis Images von beiden Kanälen dauern wird.

Der halbjährliche Kanal ist eine Feature-Update-Version von zwei Jahren mit 18-monatigen Wartungszeit Achsen für jede Version. Dadurch können Kunden die neuen Betriebssystemfunktionen schneller nutzen, und zwar sowohl in Anwendungen (insbesondere in Containern und in den gleichen Diensten) als auch im Software-Defined Hybrid Datacenter. Weitere Informationen finden Sie unter [Übersicht über den halbjährlichen Kanal in Windows Server](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview).

Bei Server Core-Images können Kunden auch den langfristigen Wartungs Kanal verwenden, der alle zwei bis drei Jahre eine neue Hauptversion von Windows Server freigibt. Langfristige Wartungs Kanal Releases erhalten fünf Jahre von grundlegender Unterstützung und fünf Jahre erweiterten Support. Dieser Channel funktioniert mit Systemen, die eine längere Wartungs Option und funktionale Stabilität erfordern.

In der folgenden Tabelle werden die einzelnen Basis Image Typen, der zugehörige Wartungs Kanal und die Dauer der Unterstützung aufgeführt.

|Base image                       |Servicing Channel|Version|OS-Build|Verfügbarkeit|Enddatum für grundlegenden Support|Datum des erweiterten Supports|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core, Nano Server, Windows|Halbjährlich      |1909   |18363   |12.11.2019  |11.05.2021                 |N/V                  |
|Server Core, Nano Server, Windows|Halbjährlich      |1903   |18362   |05/21/2019  |08.12.2020                 |N/V                  |
|Server Core                      |Langfristig        |1809   |17763   |13.11.2018  |09.01.2024                 |09.01.2029           |
|Server Core, Nano Server, Windows|Halbjährlich      |1809   |17763   |13.11.2018  |05/12/2020                 |N/V                  |
|Server Core, Nano Server         |Halbjährlich      |1803   |17134   |30.04.2018  |12.11.2019                 |N/V                  |
|Server Core, Nano Server         |Halbjährlich      |1709   |16299   |17.10.2017  |09.04.2019                 |N/V                  |
|Server Core                      |Langfristig        |1607   |14393   |15.10.2016  |11.01.2022                 |11.01.2027           |
|Nano Server                      |Halbjährlich      |1607   |14393   |15.10.2016  |10/09/2018                 |N/V                  |

Wartungsanforderungen und weitere Weitere Informationen finden Sie in den häufig gestellten Fragen zum [Windows-Lebenszyklus](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products), den [Windows Server](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)-Versionsinformationen und dem [docker Hub-Repository für Windows Base-Betriebssystem Images](https://hub.docker.com/_/microsoft-windows-base-os-images).
