---
title: PowerShell für Docker
description: Verwalten von Docker-Containern mit PowerShell
keywords: Docker, Container, PowerShell
author: PatrickLang
ms.date: 12/19/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 4a0e907d-0d07-42f8-8203-2593391071da
ms.openlocfilehash: 71d91e12ae843bf96e1b4001915ffb55bb352ffd
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576851"
---
### <a name="powershell-for-docker"></a>PowerShell für Docker

Im Gespräch mit unseren Benutzern – in Foren, über Twitter, in GitHub oder auch persönlich – wurde uns eine Frage häufiger gestellt als jede andere: Warum werden Docker-Container in PowerShell nicht angezeigt? 

Nach einer ausführlichen Diskussion über die Vor- und Nachteile sowie die verschiedenen Optionen sind wir zu dem Schluss gekommen, dass das PowerShell-Containermodul ein Update vertragen kann ... Wir werden also das PowerShell-Containermodul, das mit den Vorschaubuilds von Windows Server2016 bereitgestellt wurde, außer Betrieb nehmen und haben damit begonnen, es durch ein neues PowerShell-Modul für Docker zu ersetzen.  Die Entwicklung dieses neuen Moduls ist bereits in vollem Gang, dabei verfolgen wir jedoch einen anderen Ansatz als bisher: Wir machen unsere Arbeit öffentlich.  Wir werden bei diesem Modul verstärkt auf die Zusammenarbeit mit der Community setzen, damit das Docker-Modul zu einer benutzerfreundlicheren PowerShell-Oberfläche für Container beiträgt.  Dieses neue Modul setzt direkt auf der REST-Schnittstelle des Docker-Moduls auf und ermöglicht es Benutzern, die Docker-Befehlszeilenschnittstelle, PowerShell oder beide Tools einzusetzen.

Die Entwicklung eines leistungsstarken PowerShell-Moduls ist kein leichtes Unterfangen: Nicht nur der Code muss stimmen, sondern auch die richtige Balance zwischen Objekten, Parametersätzen und Cmdlet-Namen.  Deshalb wenden wir uns an Sie – unsere Endbenutzer und die riesige PowerShell- und Docker-Community: Helfen Sie uns, dieses Modul perfekt zu machen!  Welche Parametersätze sind wichtig für Sie?  Sollen wir ein Äquivalent zu „docker run“ einbauen, oder sollte eine Pipe von new-container zu start-container vorhanden sein – was ziehen Sie vor?  Erfahren mehr über dieses Modul und Bestandteil der Entwicklung Bitte Head über unserer GitHub-Seite (https://github.com/Microsoft/Docker-PowerShell/) und teilnehmen.

Sobald wir ein solides Alphamodul entwickelt haben, werden wir dieses im PowerShell-Katalog bereitstellen und diese Seite mit Informationen zur Verwendung aktualisieren.
