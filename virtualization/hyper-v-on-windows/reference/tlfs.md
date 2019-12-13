---
title: Spezifikationen für Hypervisor
description: Spezifikationen für Hypervisor
keywords: Windows 10, Hyper-V
author: allenma
ms.date: 06/26/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
ms.openlocfilehash: afbbcf120961081191aaf9051866427c9ce1478e
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74911180"
---
# <a name="hypervisor-specifications"></a>Spezifikationen für Hypervisor

## <a name="hypervisor-top-level-functional-specification"></a>Funktionsspezifikation der obersten Ebene für Hypervisor

Mit der Funktionsspezifikation der obersten Ebene (Top-Level Functional Specification, TLFS) für Hyper-V-Hypervisor wird das von außen für andere Betriebssystemkomponenten sichtbare Verhalten von Hypervisor beschrieben. Diese Spezifikation soll für Entwickler von Gastbetriebssystemen nützlich sein.
  
> Sie wird im Rahmen von Microsoft Open Specification Promise bereitgestellt.  Hier finden Sie weitere Details zu [Microsoft Open Specification Promise](https://docs.microsoft.com/openspecs/dev_center/ms-devcentlp/51a0d3ff-9f77-464c-b83f-2de08ed28134).  

#### <a name="download"></a>Herunterladen
Version | Dokument
--- | ---
Windows Server 2016 (Revision C) | [Hypervisor-Funktionsspezifikation der obersten Ebene v 5.0 c. PDF](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/live/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0C.pdf)
Windows Server 2012 R2 (Revision B) | [Hypervisor-Funktionsspezifikation der obersten Ebene v 4.0 b. PDF](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 | [Hypervisor-Funktionsspezifikation der obersten Ebene v 3.0. PDF](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor-Funktionsspezifikation der obersten Ebene v 2.0. PDF](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>Anforderungen für die Implementierung der Microsoft-Hypervisorschnittstelle

Die TLFS beschreibt vollständig alle Aspekte der Microsoft-spezifischen Hypervisorarchitektur, die für virtuelle Gastcomputer als „HV#1”-Schnittstelle deklariert wird.  Jedoch müssen nicht alle in den TLFS beschriebenen Schnittstellen vom Hypervisor eines Drittanbieters implementiert werden, der die Konformität mit der HV#1-Hypervisorspezifikation von Microsoft deklarieren möchte. Das Dokument „Anforderungen für die Implementierung der Microsoft Hypervisorschnittstelle” beschreibt die minimale Anzahl von Hypervisorschnittstellen, die von jedem Hypervisor implementiert werden müssen, der Kompatibilität mit der Microsoft HV#1-Schnittstelle beansprucht.

#### <a name="download"></a>Herunterladen

[Anforderungen für die Implementierung der Microsoft Hypervisor Interface. PDF](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)
