---
title: Windows-Container unter Windows Server
description: Containerbereitstellung – Schnellstart
keywords: Docker, Container
author: cwilhit
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
ms.openlocfilehash: 665e6186ff5f9530f12ba3d0a400d82bcac755a7
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/07/2019
ms.locfileid: "9999197"
---
# <a name="windows-containers-on-windows-server"></a>Windows-Container unter Windows Server

In dieser Übung geht es um die grundlegende Bereitstellung und Verwendung des Windows-Container Features unter Windows Server 2019 und Windows Server 2016.

In diesem Schnellstart können Sie Folgendes ausführen:

1. Aktivieren des Container Features in Windows Server
2. Andocker installieren
3. Ausführen eines einfachen Windows-Containers

Wenn Sie sich mit Containern vertraut machen möchten, finden Sie unter [Windows-Container](../about/index.md) entsprechende Informationen.

Dieser Schnellstart ist speziell für Windows Server-Container unter Windows Server 2019 und Windows Server 2016. Weitere Schnellstartdokumentation, darunter Container unter Windows 10, finden Sie links auf dieser Seite im Inhaltsverzeichnis.

## <a name="prerequisites"></a>Voraussetzungen

Stellen Sie bitte sicher, dass Sie die folgenden Voraussetzungen erfüllen:
- Ein physisches oder virtuelles Computersystem, auf dem Windows Server 2019 ausgeführt wird. Wenn Sie Windows Server 2019 Insider Preview verwenden, aktualisieren Sie auf [Window Server 2019-Evaluierung](https://www.microsoft.com/evalcenter/evaluate-windows-server-2019 ).

> Wichtige Updates werden benötigt, damit die Windows-Container Funktion funktioniert. Installieren Sie alle Updates, bevor Sie dieses Tutorial durcharbeiten.

Wenn Sie das Feature in Azure bereitstellen möchten, können Sie diese [Vorlage](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template) zu Hilfe nehmen.

<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="install-docker"></a>Installieren von Docker

Zur Installation von Docker verwenden wir das [OneGet-Anbieter-PowerShell-Modul](https://github.com/oneget/oneget) , das mit Anbietern zusammenarbeitet, um die Installation durchzuführen – in diesem Fall die [MicrosoftDockerProvider](https://github.com/OneGet/MicrosoftDockerProvider). Der Anbieter aktiviert das Feature „Container“ auf Ihrem Computer. Außerdem installieren Sie Docker, was einen Neustart erforderlich macht. Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client.

Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und führen Sie die folgenden Befehle aus.

Installieren Sie zuerst den Docker-Microsoft-PackageManagement-Anbieter aus der [PowerShell-Galerie](https://www.powershellgallery.com/packages/DockerMsftProvider).

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Als Nächstes verwenden Sie das PackageManagement-PowerShell-Modul, um die neueste Docker-Version zu installieren.

```powershell
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Wenn PowerShell fragt, ob die Paketquelle „DockerDefault“ vertrauenswürdig ist, geben Sie `A` ein, um die Installation fortzusetzen. Wenn die Installation abgeschlossen ist, starten Sie den Computer neu.

```powershell
Restart-Computer -Force
```

> [!TIP]
> Wenn Sie docker später aktualisieren möchten:
>  - Überprüfen Sie die installierte Version mit `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Suchen Sie die aktuelle Version mit `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Sobald Sie bereit sind, aktualisieren Sie mithilfe von `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, gefolgt von `Start-Service Docker`

## <a name="install-windows-updates"></a>Installieren von Windows-Updates

Stellen Sie sicher, dass Ihr Windows Server System auf dem neuesten Stand ist, indem Sie folgendes ausführen:

```console
sconfig
```

Dadurch wird ein textbasiertes Konfigurationsmenü angezeigt, über das Sie die Option 6 „Updates herunterladen und installieren“ auswählen können:

```console
===============================================================================
                         Server Configuration
===============================================================================

1) Domain/Workgroup:                    Workgroup:  WORKGROUP
2) Computer Name:                       WIN-HEFDK4V68M5
3) Add Local Administrator
4) Configure Remote Management          Enabled

5) Windows Update Settings:             DownloadOnly
6) Download and Install Updates
7) Remote Desktop:                      Disabled
...
```

Wählen Sie nach Aufforderung die Option „A“, um alle Updates herunterzuladen.

## <a name="deploy-your-first-container"></a>Bereitstellen des ersten Containers

Für diese Übung laden Sie ein vorab erstelltes .NET-Beispielimage von der Docker Hub-Registrierung herunter und stellen einen einfachen Container bereit, der eine Hello World .NET-Anwendung ausführt.  

Stellen Sie den .Net-Container mit `docker run` bereit. Dabei wird auch das Containerimage heruntergeladen, was mehrere Minuten dauern kann. Führen Sie je nach Host Version von Windows Server den folgenden Befehl aus.

#### <a name="windows-server-2019"></a>Windows Server 2019

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-1809
```

#### <a name="windows-server-2016"></a>Windows Server 2016

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver-sac2016
```

Der Container startet, gibt die „Hello World“-Nachricht aus und wird beendet.

```console
         Hello from .NET Core!
    __________________
                      \
                       \
                          ....
                          ....'
                           ....
                        ..........
                    .............'..'..
                 ................'..'.....
               .......'..........'..'..'....
              ........'..........'..'..'.....
             .'....'..'..........'..'.......'.
             .'..................'...   ......
             .  ......'.........         .....
             .                           ......
            ..    .            ..        ......
           ....       .                 .......
           ......  .......          ............
            ................  ......................
            ........................'................
           ......................'..'......    .......
        .........................'..'.....       .......
     ........    ..'.............'..'....      ..........
   ..'..'...      ...............'.......      ..........
  ...'......     ...... ..........  ......         .......
 ...........   .......              ........        ......
.......        '...'.'.              '.'.'.'         ....
.......       .....'..               ..'.....
   ..       ..........               ..'........
          ............               ..............
         .............               '..............
        ...........'..              .'.'............
       ...............              .'.'.............
      .............'..               ..'..'...........
      ...............                 .'..............
       .........                        ..............
        .....


**Environment**
Platform: .NET Core
OS: Microsoft Windows 10.0.17763
```

Weitere Informationen zum Befehl „Docker Run“ finden Sie in der [Referenz zu „Docker Run“ auf Docker.com](https://docs.docker.com/engine/reference/run/).

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Erfahren Sie, wie Container-Builds automatisiert und Bilder gespeichert werden.](./quick-start-images.md)
