---
title: Windows-und Linux-Container unter Windows 10
description: Richten Sie Windows 10 oder Windows Server für Container ein, und fahren Sie dann mit dem Ausführen Ihres ersten Container Images fort.
keywords: docker, Container, lkuh
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909560"
---
# <a name="get-started-prep-windows-for-containers"></a>Einstieg: prep Windows for Containers

In diesem Tutorial wird Folgendes beschrieben:

- Einrichten von Windows 10 oder Windows Server für Container
- Ausführen Ihres ersten Container Images
- Containerisieren einer einfachen .net Core-Anwendung

## <a name="prerequisites"></a>Voraussetzungen

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

Zum Ausführen von Containern unter Windows Server benötigen Sie einen physischen Server oder virtuellen Computer, auf dem Windows Server (halbjährlicher Kanal), Windows Server 2019 oder Windows Server 2016 ausgeführt wird.

Zum Testen können Sie eine Kopie von Windows [Server 2019 Evaluation](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ) oder [Windows Server Insider Preview](https://insider.windows.com/for-business-getting-started-server/)herunterladen.

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

Zum Ausführen von Containern unter Windows 10 benötigen Sie Folgendes:

- Ein physisches Computersystem, auf dem Windows 10 Professional oder Enterprise mit Anniversary Update (Version 1607) oder höher ausgeführt wird.
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) sollte aktiviert sein.

> [!NOTE]
>  Ab dem Windows 10 Oktober-Update 2018 ist es nicht mehr zulässig, dass Benutzer einen Windows-Container im Prozess Isolations Modus in Windows 10 Enterprise oder Professional für dev/Test-Zwecke ausführen. Weitere Informationen finden Sie in den [FAQ](../about/faq.md) . 
> 
> Windows Server-Container verwenden die Hyper-V-Isolation standardmäßig unter Windows 10, um Entwicklern die gleiche Kernel Version und Konfiguration bereitzustellen, die in der Produktionsumgebung verwendet werden. Weitere Informationen zur Hyper-V-Isolation finden Sie im Bereich " [Konzepte](../manage-containers/hyperv-container.md) " unserer Dokumentation.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Installieren von Docker

Der erste Schritt ist die Installation von Docker, die für die Arbeit mit Windows-Containern erforderlich ist. Docker bietet eine standardmäßige Laufzeitumgebung für Container mit einer gemeinsamen API und Befehlszeilenschnittstelle (CLI).

Weitere Informationen zur Konfiguration finden Sie unter [docker-Engine unter Windows](../manage-docker/configure-docker-daemon.md).

<!-- start tab view -->
# <a name="windows-servertabwindows-server"></a>[Windows Server](#tab/Windows-Server)

Zum Installieren von Docker unter Windows Server können Sie das von Microsoft veröffentlichte [oneget-Anbieter-PowerShell-Modul](https://github.com/oneget/oneget) als [dockermicrosoftprovider](https://github.com/OneGet/MicrosoftDockerProvider)verwenden. Dieser Anbieter aktiviert das Container Feature in Windows und installiert die Docker-Engine und den Client. Gehen Sie dazu wie folgt vor:

1. Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und installieren Sie den docker-Microsoft packagemanagement-Anbieter aus der [PowerShell-Katalog](https://www.powershellgallery.com/packages/DockerMsftProvider).

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   Wenn Sie zum Installieren des nuget-Anbieters aufgefordert werden, geben Sie `Y` ein, um es auch zu installieren.

2. Verwenden Sie das PowerShell-Modul packagemanagement, um die neueste Version von Docker zu installieren.

   ```powershell
   Install-Package -Name docker -ProviderName DockerMsftProvider
   ```

   Wenn PowerShell fragt, ob die Paketquelle „DockerDefault“ vertrauenswürdig ist, geben Sie `A` ein, um die Installation fortzusetzen.
3. Nachdem die Installation abgeschlossen ist, starten Sie den Computer neu.

   ```powershell
   Restart-Computer -Force
   ```

Wenn Sie docker später aktualisieren möchten:

- Überprüfen Sie die installierte Version mit `Get-Package -Name Docker -ProviderName DockerMsftProvider`
- Suchen der aktuellen Version mit `Find-Package -Name Docker -ProviderName DockerMsftProvider`
- Wenn Sie bereit sind, führen Sie ein Upgrade mit `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`aus, gefolgt von `Start-Service Docker`

# <a name="windows-10tabwindows-10-client"></a>[Windows 10](#tab/Windows-10-Client)

Mithilfe der folgenden Schritte können Sie docker unter Windows 10 Professional und Enterprise Edition installieren. 

1. Laden Sie [docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)herunter, und installieren Sie es, indem Sie ein kostenloses docker-Konto erstellen. Weitere Informationen finden Sie in der [docker-Dokumentation](https://docs.docker.com/docker-for-windows/install).

2. Legen Sie während der Installation den Standard Containertyp auf Windows-Container fest. Wenn Sie nach Abschluss der Installation wechseln möchten, können Sie entweder das docker-Element in der Windows-Taskleiste (wie unten gezeigt) oder den folgenden Befehl in einer PowerShell-Eingabeaufforderung verwenden:

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![Das docker-System leisten Menü, das den Befehl "zum Wechseln zu Windows-Containern" anzeigt.](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Nächste Schritte

Nachdem Sie Ihre Umgebung ordnungsgemäß konfiguriert haben, befolgen Sie den Link, um zu erfahren, wie ein Container ausgeführt wird.

> [!div class="nextstepaction"]
> [Ausführen des ersten Containers](./run-your-first-container.md)
