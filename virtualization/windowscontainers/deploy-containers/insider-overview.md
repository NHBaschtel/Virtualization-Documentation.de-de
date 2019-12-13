---
title: Verwenden von Containern mit dem Windows-Insider-Programm
description: Informationen zu den ersten Schritten bei der Verwendung von Windows-Containern mit dem Windows-Insider-Programm
keywords: docker, Container, Insider, Windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909890"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Verwenden von Containern mit dem Windows-Insider-Programm

Diese Übung führt Sie durch die Bereitstellung und Verwendung der Windows-Containerfunktionen der neuesten Insider-Build von Windows Server aus dem Windows Insider Preview-Programm. In dieser Übung installieren Sie die Containerrolle und stellen eine Preview-Edition der Basisbetriebssystemimage bereit. Wenn Sie sich mit Containern vertraut machen möchten, finden Sie unter [Windows-Container](../about/index.md) entsprechende Informationen.

## <a name="join-the-windows-insider-program"></a>Durch Ihre Teilnahme am Windows-Insider-Programm

Um die Insider-Version von Windows-Containern auszuführen, benötigen Sie einen Host, auf dem der neueste Build von Windows Server aus dem Windows-Insider Programm und/oder dem neuesten Build von Windows 10 aus dem Windows-Insider Programm ausgeführt wird. Treten Sie dem [Windows-Insider Programm](https://insider.windows.com/GettingStarted) bei, und überprüfen Sie die Nutzungsbedingungen.

> [!IMPORTANT]
> Sie müssen einen Build von Windows Server aus dem Windows Server Insider Preview-Programm oder einen Build von Windows 10 aus dem Windows Insider Preview-Programm verwenden, um das unten beschriebene Basis Image zu verwenden. Wenn Sie keine dieser Builds verwenden, kann der Container nicht mit diesen Basisimages gestartet werden.

## <a name="install-docker"></a>Installieren von Docker

Wenn Sie docker nicht bereits installiert haben, befolgen [Sie die Anweisungen](../quick-start/set-up-environment.md) in der Anleitung für die ersten Schritte, um Docker zu installieren.

## <a name="pull-an-insider-container-image"></a>Ein Insider-Container Image abrufen

Wenn Sie Teil des Windows Insider-Programms sind, können Sie unsere neuesten Builds für die Basis Images verwenden.

Zum Aufrufen des Basisimages für Nano Server Insider führen Sie folgenden Befehl aus:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Zum Aufrufen des Basisimages für Windows Server Core Insider führen Sie folgenden Befehl aus:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

Die Basis Images "Windows" und "iotcore" verfügen ebenfalls über eine Insider-Version, die zum Abrufen zur Verfügung steht. Weitere Informationen zu den verfügbaren Insider-Basis Images finden Sie im Dokument zu den [Container Basis Images](../manage-containers/container-base-images.md) .

> [!IMPORTANT]
> Lesen Sie die [Nutzungsbedingungen für](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)Windows [-Container](../images-eula.md ) -Betriebssystem Images und Windows Insider-Programme.
