---
title: "Spezifikationen für Hypervisor"
description: "Spezifikationen für Hypervisor"
keywords: Windows 10, Hyper-V
author: theodthompson
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: aee64ad0-752f-4075-a115-2d6b983b4f49
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: a92ba96f26ae2881d2c6013e8b8dcf7bafe69a47

---

# Spezifikationen für Hypervisor

## Funktionsspezifikation der obersten Ebene für Hypervisor

Mit der Funktionsspezifikation der obersten Ebene (Top-Level Functional Specification, TLFS) für Hyper-V-Hypervisor wird das von außen für andere Betriebssystemkomponenten sichtbare Verhalten von Hypervisor beschrieben. Diese Spezifikation soll für Entwickler von Gastbetriebssystemen nützlich sein.
  
> Sie wird im Rahmen von Microsoft Open Specification Promise bereitgestellt.  Hier finden Sie weitere Details zu [Microsoft Open Specification Promise](https://msdn.microsoft.com/en-us/openspecifications).  

#### Herunterladen
Version | Dokument
--- | ---
Windows Server 2012 R2 (Revision B) | [Hypervisor Top Level Functional Specification v4.0b.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0b.pdf)
Windows Server 2012 R2 | [Hypervisor Top Level Functional Specification v4.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v4.0.pdf)
Windows Server 2012 | [Hypervisor Top Level Functional Specification v3.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v3.0.pdf)
Windows Server 2008 R2 | [Hypervisor Top Level Functional Specification v2.0.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Hypervisor%20Top%20Level%20Functional%20Specification%20v2.0.pdf)

## Anforderungen für die Implementierung der Microsoft-Hypervisorschnittstelle

Für Windows-Betriebssysteme muss eine bestimmte Auswahl von Hypervisorschnittstellen auf einem virtuellen Gastcomputer ausgeführt werden (auch bekannt als die „HV #1“-Schnittstelle). Darüber hinaus können verschiedene optionale Features von einem Microsoft-kompatiblen Hypervisor implementiert werden. Diese Optionen ändern das Verhalten von Windows auf einem virtuellen Computer. Unter „Anforderungen für die Implementierung der Microsoft-Hypervisorschnittstelle“ werden die erforderlichen und optionalen Features beschrieben, die von einem Microsoft-kompatiblen Hypervisor implementiert werden.

#### Herunterladen

[Requirements for Implementing the Microsoft Hypervisor Interface.pdf](https://github.com/Microsoft/Virtualization-Documentation/raw/master/tlfs/Requirements%20for%20Implementing%20the%20Microsoft%20Hypervisor%20Interface.pdf)


<!--HONumber=Oct16_HO4-->


