---
title: Basis-Lebenszyklus von Bildservice
description: Informationen zum Lebenszyklus der Windows-Containerbasis Bilder.
keywords: Windows-Container, Container, Lebenszyklus, Veröffentlichungsinformationen, Basis Bild, Containerbasis Bild
author: Heidilohr
ms.author: helohr
ms.date: 06/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: d3a8240dbba8af3c74ce5d185620e129d103ef81
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883183"
---
# <a name="base-image-servicing-lifecycles"></a>Basis-Lebenszyklus von Bildservice

Windows-Containerbasis Bilder basieren entweder auf halbjährlichen Kanal Versionen oder auf langfristigen Wartungs Kanal Versionen von Windows Server. In diesem Artikel erfahren Sie, wie lange Unterstützung für verschiedene Versionen von Basisbildern von beiden Kanälen abhängt.

Der halbjährliche Kanal ist eine Version des Feature-Updates für zwei Mal pro Jahr mit achtzehnmonatigen Wartungs Zeitachsen für jede Version. Auf diese Weise können Kunden die neuen Betriebssystemfunktionen schneller nutzen, sowohl in Anwendungen (insbesondere auf Containern und microservices) als auch im softwaredefinierten Hybrid-Rechenzentrum. Weitere Informationen finden Sie in der [Übersicht über den halbjährlichen Windows Server-Kanal](https://docs.microsoft.com/windows-server/get-started/semi-annual-channel-overview).

Für Server Core-Bilder können Kunden auch den lang Zeitdienst Kanal verwenden, der alle zwei bis drei Jahre eine neue Hauptversion von Windows Server freigibt. Langfristige Wartungs Kanal Freigaben erhalten fünf Jahre Mainstream-Support und fünf Jahre erweiterter Support. Dieser Kanal funktioniert mit Systemen, die eine längere Service Option und Funktionsstabilität erfordern.

In der folgenden Tabelle sind die einzelnen Typen von Basisbildern, deren Wartungs Kanal und die Dauer der Unterstützung aufgeführt.

|Basis Bild                       |Servicing Channel|Version|BS-Build|Verfügbarkeit|Enddatum für grundlegenden Support|Verlängerter Support Termin|
|---------------------------------|-----------------|-------|--------|------------|---------------------------|---------------------|
|Server Core, Nano Server, Windows|Halbjährlich      |1903   |18362   |05/21/2019  |12/08/2020                 |n.v.                  |
|Server Core                      |Langfristig        |1809   |17763   |13.11.2018  |09.01.2024                 |09.01.2029           |
|Server Core, Nano Server, Windows|Halbjährlich      |1809   |17763   |13.11.2018  |05/12/2020                 |n.v.                  |
|Server Core, Nano Server         |Halbjährlich      |1803   |17134   |30.04.2018  |12.11.2019                 |n.v.                  |
|Server Core, Nano Server         |Halbjährlich      |1709   |16299   |17.10.2017  |09.04.2019                 |n.a.                  |
|Server Core                      |Langfristig        |1607   |14393   |15.10.2016  |11.01.2022                 |11.01.2027           |
|Nano Server                      |Halbjährlich      |1607   |14393   |15.10.2016  |10/09/2018                 |n.v.                  |

Informationen zu Wartungsanforderungen und anderen zusätzlichen Informationen finden Sie in den [Windows-Lebenszyklus-häufig](https://support.microsoft.com/help/18581/lifecycle-faq-windows-products)gestellten Fragen, [Windows Server-Veröffentlichungsinformationen](https://docs.microsoft.com/windows-server/get-started/windows-server-release-info)und dem [WindowsBase OS Images andocker Hub-Repo](https://hub.docker.com/_/microsoft-windows-base-os-images).
