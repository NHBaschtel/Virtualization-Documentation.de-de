---
title: Verwenden von Containern mit Windows-Insider-Programm
description: Informationen zum Einstieg in die Verwendung von Windows-Containern mit dem Windows-Insider-Programm
keywords: docker, Container, Insider, Windows
author: cwilhit
ms.openlocfilehash: 92fb359df1c207b848fb985caf7f46852f6b4f90
ms.sourcegitcommit: 6080b2c5053720490d374f6fb0daa870d5ddd4e8
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/25/2019
ms.locfileid: "10257794"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Verwenden von Containern mit dem Windows-Insider-Programm

Diese Übung führt Sie durch die Bereitstellung und Verwendung der Windows-Containerfunktionen der neuesten Insider-Build von Windows Server aus dem Windows Insider Preview-Programm. In dieser Übung installieren Sie die Containerrolle und stellen eine Preview-Edition der Basisbetriebssystemimage bereit. Wenn Sie sich mit Containern vertraut machen möchten, finden Sie unter [Windows-Container](../about/index.md) entsprechende Informationen.

## <a name="join-the-windows-insider-program"></a>Am Windows-Insider-Programm teilnehmen

Um die Insider-Version von Windows-Containern ausführen zu können, müssen Sie über einen Host verfügen, der den neuesten Build von Windows Server aus dem Windows-Insider-Programm und/oder dem neuesten Build von Windows 10 aus dem Windows-Insider-Programm ausführt. Nehmen Sie am [Windows-Insider-Programm](https://insider.windows.com/GettingStarted) Teil und überprüfen Sie die Nutzungsbestimmungen.

> [!IMPORTANT]
> Sie müssen einen Build von Windows Server aus dem Windows Server Insider Preview-Programm oder einen Build von Windows 10 aus dem Windows Insider Preview-Programm verwenden, um das unten beschriebene Basis Bild zu verwenden. Wenn Sie keine dieser Builds verwenden, kann der Container nicht mit diesen Basisimages gestartet werden.

## <a name="install-docker"></a>Installieren von Docker

Wenn Sie docker nicht bereits installiert haben, folgen Sie dem Leitfaden " [Erste Schritte](../quick-start/set-up-environment.md) ", um andocker zu installieren.

## <a name="pull-an-insider-container-image"></a>Ziehen eines Insider-Container Bilds

Indem Sie Teil des Windows-Insider-Programms sind, können Sie unsere neuesten Builds für die Basisbilder verwenden.

Zum Aufrufen des Basisimages für Nano Server Insider führen Sie folgenden Befehl aus:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Zum Aufrufen des Basisimages für Windows Server Core Insider führen Sie folgenden Befehl aus:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

Die Basisbilder "Windows" und "IoTCore" verfügen ebenfalls über eine Insider-Version, die Sie abrufen können. Weitere Informationen zu den verfügbaren Insider Base-Bildern finden Sie im Dokument " [Container-Basisbilder](../manage-containers/container-base-images.md) ".

> [!IMPORTANT]
> Bitte lesen Sie die [Nutzungsbedingungen für](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)Windows-Container-Betriebs [System Bilder und](../images-eula.md ) das Windows-Insider-Programm.
