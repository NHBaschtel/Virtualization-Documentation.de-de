---
title: Windows-und Linux-Container unter Windows 10
description: Containerbereitstellung – Schnellstart
keywords: docker, Container, LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 5f0922a1ee2588b6e5a06091fe34e07ceadf89cb
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129377"
---
# <a name="get-started-configure-your-environment-for-containers"></a>Erste Schritte: Konfigurieren der Umgebung für Container

Dieser Schnellstart zeigt, wie Sie:

> [!div class="checklist"]
> * Einrichten Ihrer Umgebung für Container
> * Ausführen des ersten Container Bilds
> * Containerisieren einer einfachen .net Core-Anwendung

## <a name="prerequisites"></a>Voraussetzungen

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Stellen Sie bitte sicher, dass Sie die folgenden Voraussetzungen erfüllen:

- Ein physisches oder virtuelles Computersystem, auf dem Windows Server 2016 oder höher ausgeführt wird.

> [!NOTE]
> Wenn Sie Windows Server 2019 Insider Preview verwenden, aktualisieren Sie auf [Window Server 2019-Evaluierung](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ).

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 Professional und Enterprise](#tab/Windows-10-Client)

Stellen Sie bitte sicher, dass Sie die folgenden Voraussetzungen erfüllen:

- Ein physisches Computersystem, auf dem Windows 10 Professional oder Enterprise mit Anniversary Update (Version 1607) oder höher ausgeführt wird.
- [Hyper-V](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements) sollte aktiviert sein.

> [!NOTE]
>  Ab dem Windows 10 Oktober-Update 2018 verbieten wir Benutzern nicht mehr die Ausführung eines Windows-Containers im Prozess Isolationsmodus unter Windows 10 Enterprise oder Professional für dev/Test-Zwecke. Weitere Informationen finden Sie in den [FAQ](../about/faq.md) . 
> 
> Windows Server-Container verwenden standardmäßig die Hyper-V-Isolierung unter Windows 10, um Entwicklern die gleiche Kernel Version und-Konfiguration zur Verfügung zu stellen, die in der Produktion verwendet wird. Weitere Informationen zur Hyper-V-Isolierung finden Sie im Bereich [Konzepte](../manage-containers/hyperv-container.md) unserer Dokumente.

---
<!-- stop tab view -->

## <a name="install-docker"></a>Installieren von Docker

Docker ist die definitive Toolchain für die Arbeit mit Windows-Containern. Andocker bietet eine CLI für Benutzer zum Verwalten von Containern auf einem bestimmten Host, zum Erstellen von Containern, zum Entfernen von Containern und mehr. Weitere Informationen zu docker finden Sie im Bereich " [Konzepte](../manage-containers/configure-docker-daemon.md) " unserer Dokumente.

<!-- start tab view -->
# [<a name="windows-server"></a>Windows Server](#tab/Windows-Server)

Unter Windows Server wird docker über ein OneGet- [Anbieter-PowerShell-Modul](https://github.com/oneget/oneget) installiert, das von Microsoft als [DockerMicrosoftProvider](https://github.com/OneGet/MicrosoftDockerProvider)veröffentlicht wurde. Dieser Anbieter:

- aktiviert die Container Funktion auf Ihrem Computer
- installiert das Andock Modul und den Client auf Ihrem Computer.

Wenn Sie docker installieren möchten, öffnen Sie eine erhöhte PowerShell-Sitzung, und installieren Sie den docker-Microsoft PackageManagement-Anbieter aus dem [PowerShell-Katalog](https://www.powershellgallery.com/packages/DockerMsftProvider).

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Verwenden Sie als nächstes das PackageManagement PowerShell-Modul, um die neueste Version von Docker zu installieren.

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Wenn PowerShell fragt, ob die Paketquelle „DockerDefault“ vertrauenswürdig ist, geben Sie `A` ein, um die Installation fortzusetzen. Nachdem die Installation abgeschlossen ist, müssen Sie den Computer neu starten.

```powershell
Restart-Computer -Force
```

> [!TIP]
> Wenn Sie docker später aktualisieren möchten:
>  - Überprüfen Sie die installierte Version mit `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Suchen Sie die aktuelle Version mit `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Sobald Sie bereit sind, aktualisieren Sie mithilfe von `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, gefolgt von `Start-Service Docker`

# [<a name="windows-10-professional-and-enterprise"></a>Windows 10 Professional und Enterprise](#tab/Windows-10-Client)

Unter Windows 10 Professional und Enterprise wird docker über ein klassisches Installationsprogramm installiert. Laden Sie [docker Desktop](https://store.docker.com/editions/community/docker-ce-desktop-windows) herunter, und führen Sie das Installationsprogramm aus. Sie werden aufgefordert, sich anzumelden. Erstellen Sie ein Konto, wenn Sie noch keines haben. Ausführlichere Installationsanweisungen finden Sie in der [docker-Dokumentation](https://docs.docker.com/docker-for-windows/install).

Nach der Installation wird der Docker-Desktop standardmäßig mit Linux-Containern ausgeführt. Wechseln Sie zu Windows-Containern entweder über das Docking-Tray-Menü oder durch Ausführen des folgenden Befehls in einer PowerShell-Eingabeaufforderung:

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

---
<!-- stop tab view -->

## <a name="next-steps"></a>Nächste Schritte

Nachdem Ihre Umgebung nun ordnungsgemäß konfiguriert wurde, folgen Sie dem Link, um zu erfahren, wie Sie einen Container ziehen und ausführen.

> [!div class="nextstepaction"]
> [Ausführen des ersten Containers](./run-your-first-container.md)
