---
title: Windows-und Linux-Container unter Windows 10
description: Richten Sie Windows 10 oder Windows Server für Container ein, und fahren Sie dann mit dem Ausführen des ersten Container Bilds fort.
keywords: docker, Container, LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 2c52dd96b3bf2402d41ec5b178af36521d00a649
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288118"
---
# <a name="get-started-prep-windows-for-containers"></a>Erste Schritte: Vorbereiten von Windows für Container

In diesem Lernprogramm wird beschrieben, wie Sie:

- Einrichten von Windows 10 oder Windows Server für Container
- Ausführen des ersten Container Bilds
- Containerisieren einer einfachen .net Core-Anwendung

## <a name="prerequisites"></a>Voraussetzungen

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Zum Ausführen von Containern unter Windows Server benötigen Sie einen physikalischen Server oder virtuellen Computer mit Windows Server (halbjährlicher Kanal), Windows Server 2019 oder Windows Server 2016.

Zum Testen können Sie eine Kopie der [Window Server 2019-Evaluierung](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ) oder einer [Windows Server Insider Preview](https://insider.windows.com/for-business-getting-started-server/)herunterladen.

# [<a name="windows-10"></a>Windows 10](#tab/Windows-10-Client)

Zum Ausführen von Containern unter Windows 10 benötigen Sie Folgendes:

- Ein physisches Computersystem, auf dem Windows 10 Professional oder Enterprise mit Anniversary Update (Version 1607) oder höher ausgeführt wird.
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) sollte aktiviert sein.

> [!NOTE]
>  Ab dem Windows 10 Oktober-Update 2018 verbieten wir Benutzern nicht mehr die Ausführung eines Windows-Containers im Prozess Isolationsmodus unter Windows 10 Enterprise oder Professional für dev/Test-Zwecke. Weitere Informationen finden Sie in den [FAQ](../about/faq.md) . 
> 
> Windows Server-Container verwenden standardmäßig die Hyper-V-Isolierung unter Windows 10, um Entwicklern die gleiche Kernel Version und-Konfiguration zur Verfügung zu stellen, die in der Produktion verwendet wird. Weitere Informationen zur Hyper-V-Isolierung finden Sie im Bereich [Konzepte](../manage-containers/hyperv-container.md) unserer Dokumente.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Installieren von Docker

Der erste Schritt besteht darin, Docker zu installieren, der für die Arbeit mit Windows-Containern erforderlich ist. Andocker stellt eine standardmäßige Laufzeitumgebung für Container mit einer allgemeinen API und einer Befehlszeilenschnittstelle (CLI) bereit.

Weitere Konfigurationsdetails finden Sie unter [docker Modul unter Windows](../manage-docker/configure-docker-daemon.md).

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Wenn Sie docker unter Windows Server installieren möchten, können Sie ein [OneGet-Anbieter-PowerShell-Modul](https://github.com/oneget/oneget) verwenden, das von Microsoft als [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider)veröffentlicht wurde. Dieser Anbieter aktiviert das Container Feature in Windows und installiert das Andock Modul und den Client. Gehen Sie wie folgt vor:

1. Öffnen Sie eine erhöhte PowerShell-Sitzung, und installieren Sie den docker-Microsoft PackageManagement-Anbieter aus dem [PowerShell-Katalog](https://www.powershellgallery.com/packages/DockerMsftProvider).

   ```powershell
   Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
   ```

   Wenn Sie aufgefordert werden, den NuGet-Anbieter zu installieren `Y` , geben Sie ihn ein, um ihn ebenfalls zu installieren.

2. Verwenden Sie das PackageManagement PowerShell-Modul, um die neueste Version von Docker zu installieren.

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
- Suchen Sie die aktuelle Version mit `Find-Package -Name Docker -ProviderName DockerMsftProvider`
- Sobald Sie bereit sind, aktualisieren Sie mithilfe von `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, gefolgt von `Start-Service Docker`

# [<a name="windows-10"></a>Windows 10](#tab/Windows-10-Client)

Mit den folgenden Schritten können Sie docker unter Windows 10 Professional und Enterprise Editions installieren. 

1. Laden Sie den [docker-Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows)herunter, und installieren Sie ein kostenloses docker-Konto, wenn Sie noch keines haben. Weitere Informationen finden Sie in der [docker-Dokumentation](https://docs.docker.com/docker-for-windows/install).

2. Legen Sie während der Installation den Standard Containertyp auf Windows-Container ein. Wenn Sie nach Abschluss der Installation wechseln möchten, können Sie entweder das Andock Element in der Windows-Taskleiste (wie unten dargestellt) oder den folgenden Befehl in einer PowerShell-Eingabeaufforderung verwenden:

   ```console
   & $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
   ```

![Menü "docker-Taskleiste" mit dem Befehl "zu Windows-Container wechseln"](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Nächste Schritte

Nachdem Ihre Umgebung nun ordnungsgemäß konfiguriert wurde, folgen Sie dem Link, um zu erfahren, wie Sie einen Container ausführen.

> [!div class="nextstepaction"]
> [Ausführen des ersten Containers](./run-your-first-container.md)
