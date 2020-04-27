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
ms.openlocfilehash: 6af2002d06258e2a4d89d1f5d2698d22d2f46049
ms.sourcegitcommit: 16ebc4f00773d809fae84845208bd1dcf08a889c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/24/2020
ms.locfileid: "80929966"
---
# <a name="hypervisor-specifications"></a>Spezifikationen für Hypervisor

## <a name="hypervisor-top-level-functional-specification"></a>Funktionsspezifikation der obersten Ebene für Hypervisor

Mit der Funktionsspezifikation der obersten Ebene (Top-Level Functional Specification, TLFS) für Hyper-V-Hypervisor wird das von außen für andere Betriebssystemkomponenten sichtbare Verhalten von Hypervisor beschrieben. Diese Spezifikation soll für Entwickler von Gastbetriebssystemen nützlich sein.
  
> Sie wird im Rahmen von Microsoft Open Specification Promise bereitgestellt.  Hier finden Sie weitere Details zu [Microsoft Open Specification Promise](https://docs.microsoft.com/openspecs/dev_center/ms-devcentlp/51a0d3ff-9f77-464c-b83f-2de08ed28134).  

#### <a name="download"></a>Herunterladen
Version | Dokument
--- | ---
Windows Server 2019 (Revision B) | [Hypervisor Top Level Functional Specification v6.0b.pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v6.0b.pdf)
Windows Server 2016 (Revision C) | [Hypervisor Top Level Functional Specification v5.0c.pdf](https://github.com/MicrosoftDocs/Virtualization-Documentation/raw/live/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v5.0C.pdf)
Windows Server 2012 R2 (Revision B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## <a name="requirements-for-implementing-the-microsoft-hypervisor-interface"></a>Anforderungen für die Implementierung der Microsoft-Hypervisorschnittstelle

Die TLFS beschreibt vollständig alle Aspekte der Microsoft-spezifischen Hypervisorarchitektur, die für virtuelle Gastcomputer als „HV#1”-Schnittstelle deklariert wird.  Jedoch müssen nicht alle in der TLFS beschriebenen Schnittstellen vom Hypervisor eines Drittanbieters implementiert werden, der Konformität mit der HV#1-Hypervisorspezifikation von Microsoft deklarieren möchte. Das Dokument „Anforderungen für die Implementierung der Microsoft Hypervisorschnittstelle” beschreibt die minimale Anzahl von Hypervisorschnittstellen, die von jedem Hypervisor implementiert werden müssen, der Kompatibilität mit der Microsoft HV#1-Schnittstelle beansprucht.

#### <a name="download"></a>Herunterladen

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)
