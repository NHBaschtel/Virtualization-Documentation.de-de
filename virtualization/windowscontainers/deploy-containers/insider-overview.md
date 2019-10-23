---
title: Verwenden von Containern mit Windows-Insider-Programm
description: Informationen zum Einstieg in die Verwendung von Windows-Containern mit dem Windows-Insider-Programm
keywords: docker, Container, Insider, Windows
author: cwilhit
ms.openlocfilehash: 137209a66c3d0b907003498fe78a04a57a140130
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254396"
---
# <a name="use-containers-with-the-windows-insider-program"></a>Verwenden von Containern mit dem Windows-Insider-Programm

Diese Übung führt Sie durch die Bereitstellung und Verwendung der Windows-Containerfunktionen der neuesten Insider-Build von Windows Server aus dem Windows Insider Preview-Programm. In dieser Übung installieren Sie die Containerrolle und stellen eine Preview-Edition der Basisbetriebssystemimage bereit. Wenn Sie sich mit Containern vertraut machen möchten, finden Sie unter [Windows-Container](../about/index.md) entsprechende Informationen.

> [!NOTE]
> Dieser Inhalt ist speziell für Windows Server-Container im Windows Server Insider Preview-Programm. Wenn Sie nach nicht Insider-Anleitungen für die Verwendung von Windows-Containern suchen, lesen Sie den Leitfaden " [Erste Schritte](../quick-start/set-up-environment.md) ".

## <a name="join-the-windows-insider-program"></a>Am Windows-Insider-Programm teilnehmen

Um die Insider-Version von Windows-Containern ausführen zu können, müssen Sie über einen Host verfügen, der den neuesten Build von Windows Server aus dem Windows-Insider-Programm und/oder dem neuesten Build von Windows 10 aus dem Windows-Insider-Programm ausführt. Nehmen Sie am [Windows-Insider-Programm](https://insider.windows.com/GettingStarted) Teil und überprüfen Sie die Nutzungsbestimmungen.

> [!IMPORTANT]
> Sie müssen einen Build von Windows Server aus dem Windows Server Insider Preview-Programm oder einen Build von Windows 10 aus dem Windows Insider Preview-Programm verwenden, um das unten beschriebene Basis Bild zu verwenden. Wenn Sie keine dieser Builds verwenden, kann der Container nicht mit diesen Basisimages gestartet werden.

## <a name="install-docker"></a>Installieren von Docker

<!-- start tab view -->
# [<a name="windows-server-insider"></a>Windows Server Insider](#tab/Windows-Server-Insider)

Verwenden Sie das PowerShell-Modul von OneGet, um Docker EE zu installieren. Der Anbieter aktiviert die Containerfunktion auf Ihrem Computer und installiert Docker EE. Dazu ist ein Neustart erforderlich. Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und führen Sie die folgenden Befehle aus.

> [!NOTE]
> Die Installation von Docker EE mit Windows Server Insider-Builds erfordert einen anderen OneGet-Anbieter als den, der für nicht-Insider-Builds verwendet wird. Wenn Docker EE und der OneGet-Anbieter DockerMsftProvider bereits installiert sind, entfernen Sie diese Komponenten, bevor Sie fortfahren.

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

Installieren Sie das PowerShell-Modul OneGet für die Verwendung mit Windows-Insider-Builds.

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

Verwenden Sie OneGet, um die neueste Vorschauversion von Docker EE zu installieren.

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

Wenn die Installation abgeschlossen ist, starten Sie den Computer neu.

```powershell
Restart-Computer -Force
```

# [<a name="windows-10-insider"></a>Windows 10 Insider](#tab/Windows-10-Insider)

Unter Windows 10 Insider wird docker Edge über das gleiche Installationsprogramm wie das docker-Desktop stabil installiert. Laden Sie [docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) herunter, und führen Sie das Installationsprogramm aus. Sie werden aufgefordert, sich anzumelden. Erstellen Sie ein Konto, wenn Sie noch keines haben. Ausführlichere Installationsanweisungen finden Sie in der [docker-Dokumentation](https://docs.docker.com/docker-for-windows/install).

Öffnen Sie nach der Installation die Einstellungen des Dockers, und wechseln Sie zum Kanal "Edge".

![](./media/docker-edge-instruction.png)

---
<!-- stop tab view -->

## <a name="pull-an-insider-container-image"></a>Ziehen eines Insider-Container Bilds

Vor der Arbeit mit Windows-Containern muss ein Basisimage installiert werden. Indem Sie Teil des Windows-Insider-Programms sind, können Sie unsere neuesten Builds für die Basisbilder verwenden. Weitere Informationen zu den verfügbaren Basisbildern finden Sie im Dokument " [Container Basisbilder](../manage-containers/container-base-images.md) ".

Zum Aufrufen des Basisimages für Nano Server Insider führen Sie folgenden Befehl aus:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Zum Aufrufen des Basisimages für Windows Server Core Insider führen Sie folgenden Befehl aus:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> Bitte lesen Sie die [Nutzungsbedingungen für](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)Windows-Container-Betriebs [System Bilder und](../images-eula.md ) das Windows-Insider-Programm.
