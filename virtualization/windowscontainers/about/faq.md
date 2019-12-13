---
title: Windows-Container – häufig gestellte Fragen
description: FAQ zu Windows Server-Containern
keywords: Docker, Container
author: PatrickLang
ms.date: 10/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 25de368c-5a10-40a4-b4aa-ac8c9a9ca022
ms.openlocfilehash: 405b2abc43a4ae2c546de351679deb755e4a9317
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910800"
---
# <a name="frequently-asked-questions-about-containers"></a>Häufig gestellte Fragen zu Containern

## <a name="whats-the-difference-between-linux-and-windows-server-containers"></a>Worin besteht der Unterschied zwischen Linux-und Windows Server-Containern?

Linux und Windows Server implementieren beide ähnliche Technologien in Ihren Kernel-und Kern Betriebssystemen. Der Unterschied ergibt sich aus der Plattform und Arbeitslasten, die innerhalb des Containers ausgeführt.  

Wenn ein Kunde Windows Server-Container verwendet, kann er in vorhandene Windows-Technologien wie .net, ASP.net und PowerShell integriert werden.

## <a name="what-are-the-prerequisites-for-running-containers-on-windows"></a>Was sind die Voraussetzungen für die Ausführung von Containern unter Windows?

Container wurden mit Windows Server 2016 in die Plattform eingeführt. Zum Verwenden von Containern benötigen Sie entweder Windows Server 2016 oder das Windows 10 Anniversary Update (Version 1607) oder höher. Weitere Informationen finden Sie in den [System Anforderungen](../deploy-containers/system-requirements.md) .

## <a name="what-are-wcow-and-lcow"></a>Was sind wkuh und lkuh?

Wkuh ist für Windows-Container unter Windows kurz. Lkuh ist für "Linux-Container unter Windows" kurz.

## <a name="how-are-containers-licensed-is-there-a-limit-to-the-number-of-containers-i-can-run"></a>Wie werden Container lizenziert? Gibt es eine Beschränkung für die Anzahl der Container, die ich ausführen kann?

Das Windows-Container Image [EULA](../images-eula.md) beschreibt eine Verwendung, die davon abhängt, dass ein Benutzer über ein ordnungsgemäß lizenziertes Host Betriebssystem verfügt. Die Anzahl der Container, die ein Benutzer ausführen darf, hängt von der Host-Betriebssystem Edition und dem [Isolations Modus](../manage-containers/hyperv-container.md) ab, mit dem ein Container ausgeführt wird, sowie davon, ob diese Container für Entwicklungs-/Testzwecke oder in der Produktionsumgebung ausgeführt werden.

|Hostbetriebssystem                                                         |Limit für Prozess isolierte Container                   |Limit für durch Hyper-V-isolierte Container                   |
|----------------------------------------------------------------|---------------------------------------------------|---------------------------------------------------|
|Windows Server Standard                                         |Unbegrenzt                                          |2                                                  |
|Windows Server Datacenter                                       |Unbegrenzt                                          |Unbegrenzt                                          |
|Windows 10 pro und Enterprise                                   |Unbegrenzt *(nur für Test-oder Entwicklungszwecke)*|Unbegrenzt *(nur für Test-oder Entwicklungszwecke)*|
|Windows 10 IOT Core und Enterprise                             |Grenzenlos                                         |Grenzenlos                                          |

Die Verwendung des Windows Server-Container Images wird durchlesen der Anzahl der für diese [Edition](/windows-server/get-started-19/editions-comparison-19.md)unterstützten Virtualisierungshosts ermittelt. <br/>

>[!NOTE]
>\*Produktions Nutzung von Containern in einer IOT-Edition von Windows ist abhängig davon, ob Sie die kommerziellen Nutzungsbedingungen von Microsoft für Windows 10 Core-Lauf Zeit Images oder die Windows 10 IOT Enterprise-Geräte Lizenz ("Windows IOT-Handelsvereinbarung") zugestimmt haben. Zusätzliche Nutzungsbedingungen in den Windows IOT-Handelsvereinbarungen gelten für die Verwendung des Container Images in einer Produktionsumgebung. Lesen Sie das [Container Image EULA](../images-eula.md) , um genau zu verstehen, was zulässig ist und was nicht.

## <a name="as-a-developer-do-i-have-to-rewrite-my-app-for-each-type-of-container"></a>Muss ich als Entwickler meine APP für jeden Containertyp umschreiben?

Nein. Windows-Container Images sind für Windows Server-Container und Hyper-V-Isolation üblich. Die Auswahl des Containertyps erfolgt beim Starten des Containers. Aus Sicht des Entwicklers sind Windows Server-Container und Hyper-V-Isolation zwei Arten desselben. Sie bieten die gleiche Entwicklungs-, Programmierungs-und Verwaltungsfunktionen und sind offen und erweiterbar und enthalten die gleiche Ebene der Integration und Unterstützung mit Docker.

Ein Entwickler kann ein Container Image mithilfe eines Windows Server-Containers erstellen und in Hyper-V-Isolation oder umgekehrt ohne andere Änderungen als die Angabe des entsprechenden Laufzeitflags bereitstellen.

Windows Server-Container bieten höhere Dichte und Leistung, wenn die Geschwindigkeit entscheidend ist, wie z. b. eine niedrigere aufzurufende Zeit und eine schnellere Laufzeitleistung im Vergleich zu den in der Umgebung Die Hyper-V-Isolation, die auf den Namen zutreffen, bietet eine größere Isolation und stellt sicher, dass der Code, der in einem Container ausgeführt wird, nicht beeinträchtigt wird oder Auswirkungen auf das Host Betriebssystem oder andere auf demselben Host ausgeführt wird. Dies ist hilfreich für mehr Instanzen fähige Szenarien mit Anforderungen für das Hosting von nicht vertrauenswürdigem Code, einschließlich SaaS-Anwendungen und computehosting.

## <a name="can-i-run-windows-containers-in-process-isolated-mode-on-windows-10"></a>Kann ich Windows-Container unter Windows 10 im Prozess isolierten Modus ausführen?

Ab dem Windows 10-Update vom Oktober 2018 können Sie einen Windows-Container mit Prozess Isolation ausführen. Sie müssen jedoch zunächst die Prozess Isolation direkt anfordern, indem Sie das `--isolation=process`-Flag verwenden, wenn Sie Ihre Container mit `docker run`ausführen. Die Prozess Isolation ist kompatibel mit Windows 10 pro, Windows 10 Enterprise, Windows 10 IOT Core und Windows 10 IOT Enterprise.

Wenn Sie Ihre Windows-Container auf diese Weise ausführen möchten, müssen Sie sicherstellen, dass auf dem Host Windows 10 Build 17763 und höher ausgeführt wird und Sie über eine docker-Version mit der Engine 18,09 oder höher verfügen.

> [!WARNING]
> Abgesehen von auf IOT Core-und IOT Enterprise-Hosts (nachdem Sie weitere Bedingungen und Einschränkungen akzeptiert haben) ist dieses Feature nur für Entwicklung und Tests gedacht. Sie sollten weiterhin Windows Server als Host für Produktions Bereitstellungen verwenden. Wenn Sie diese Funktion verwenden, müssen Sie auch sicherstellen, dass die Host-und Container Versions Tags Stimmen, da andernfalls der Container nicht gestartet werden kann oder nicht definiertes Verhalten aufweist.

## <a name="how-do-i-make-my-container-images-available-on-air-gapped-machines"></a>Gewusst wie die Verfügbarkeit meiner Container Images auf abgelösten Computern?

Windows-Containerbasis Images enthalten Artefakte, deren Verteilung durch eine Lizenz eingeschränkt ist. Wenn Sie auf diesen Images aufbauen und Sie per Push an eine private oder öffentliche Registrierung übermittelt haben, werden Sie feststellen, dass die Basisebene niemals per Push übermittelt wird Stattdessen wird das Konzept einer fremden Ebene verwendet, die auf die tatsächliche Basisschicht im Azure-cloudspeicher verweist.

Dies kann die Dinge erschweren, wenn Sie einen Computer mit einem Computer haben, der nur Bilder von der Adresse Ihrer privaten Container Registrierung abrufen kann. In diesem Fall ist es nicht möglich, die fremd Schicht zu befolgen, um das Basis Image zu erhalten. Um das Verhalten der fremd Schicht zu überschreiben, können Sie das `--allow-nondistributable-artifacts`-Flag im docker-Daemon verwenden.

> [!IMPORTANT]
> Die Verwendung dieses Flags schließt Ihre Verpflichtung nicht aus, die Bedingungen der Windows-Container-Basis Image Lizenz einzuhalten. Sie dürfen keinen Windows-Inhalt für die öffentliche oder Drittanbieter Verteilung bereitstellen. Die Verwendung in ihrer eigenen Umgebung ist zulässig.

## <a name="additional-feedback"></a>Zusätzliches Feedback

Möchten Sie den FAQ etwas hinzufügen? Öffnen Sie im Abschnitt "Kommentare" ein neues Feedback Problem, oder richten Sie eine Pull Request für diese Seite mit GitHub ein.
