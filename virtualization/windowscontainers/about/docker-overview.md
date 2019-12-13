---
title: Informationen zu docker
description: Erfahren Sie mehr über Docker.
keywords: Docker, Container
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: cdb7771a1293e7c3fd505103f0010bdfba47cc31
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910790"
---
# <a name="about-docker"></a>Informationen zu docker

Informationen zu Containern erwähnen zwangsläufig auch Docker. Die Docker-Engine ist ein Container-Management-Toolset, das Container Images verpackt und bereitstellt. Die resultierenden Images können überall als Container, unabhängig davon, ob Sie lokal, in der Cloud oder auf einem persönlichen Computer ausgeführt werden, ausgeführt werden.

![](media/docker.png)

Sie können einen Windows Server-Container mit [docker](https://www.docker.com) wie jeden anderen Container verwalten.

Das Konzept der Namespace Isolation und der Ressourcenkontrolle über einen Container ist für einen längeren Zeitraum aufgetreten und wurde zu den BSD-Jails, Solaris-Zonen und dem grundlegenden Unix-change root-Mechanismus (chroot) zurückgekehrt. Docker stellt eine solide Grundlage für die Entwicklung durch ein gemeinsames Toolset, Paket Modell und einen Bereitstellungs Mechanismus dar, die die Containerisierung und die Verteilung von Anwendungen vereinfachen. Diese Anwendungen können dann an einer beliebigen Stelle auf einem Linux-Host und in Windows ausgeführt werden.

Ein allgegenwärtiges Verpackungs Modell und eine Bereitstellungs Technologie vereinfachen die Verwaltung, indem die gleichen Verwaltungs Befehle für jeden Host angeboten werden, wodurch eine einmalige Gelegenheit für nahtlose devops entsteht. Sie können auch ein docker-Image erstellen, das in wenigen Sekunden in jeder Umgebung bereitgestellt wird, egal, ob es sich um einen Desktop eines Entwicklers, einen Testcomputer oder eine Gruppe von Produktions Computern handelt. Dies hat ein riesiges und wachsendes Ökosystem von Anwendungen entwickelt, die in docker-Containern mit dockerhub verpackt sind, und zwar mit der öffentlichen Anwendungs Registrierung in Containern, die Docker verwaltet.

Im folgenden wird das Ökosystem der Anwendungen erläutert und erläutert, wie Sie auf docker-Konzepten aufbauen können, um einen Entwicklungs-und Bereitstellungs Workflow zu erstellen, der für Ihre Anforderungen geeignet ist.

## <a name="get-started-with-docker"></a>Erste Schritte mit Docker

Informationen zum Erstellen von Containern mit docker finden Sie unter [docker-Engine unter Windows](../manage-docker/configure-docker-daemon.md). Weitere ausführliche Informationen zur Verwendung von Docker finden Sie auf [der Docker-Website](https://www.docker.com) .