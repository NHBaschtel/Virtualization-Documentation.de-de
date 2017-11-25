---
title: Windows-Container unter Windows Server
description: "Containerbereitstellung – Schnellstart"
keywords: Docker, Container
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
ms.openlocfilehash: 73112b3d1b86b5c72cb6352faaaac3ae95b0957e
ms.sourcegitcommit: 456485f36ed2d412cd708aed671d5a917b934bbe
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 11/08/2017
---
# <a name="windows-containers-on-windows-server"></a>Windows-Container unter Windows Server

Die Übung führt durch die einfache Bereitstellung und Verwendung des Windows-Container-Features unter Windows Server 2016. In dieser Übung installieren Sie die Containerrolle, und stellen einen einfachen Windows Server-Container bereit. Wenn Sie sich mit Containern vertraut machen möchten, finden Sie unter [Windows-Container](../about/index.md) entsprechende Informationen.

Dieser Schnellstart ist spezifisch für Windows Server-Container unter Windows Server 2016. Weitere Schnellstartdokumentation, darunter Container unter Windows 10, finden Sie links auf dieser Seite im Inhaltsverzeichnis.

**Voraussetzungen:**

Ein Computersystem (physisch oder virtuell), auf dem Windows Server 2016 ausgeführt wird Wenn Sie Windows Server 2016 TP5 verwenden, updaten Sie auf [Windows Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ).

> Wichtige Updates sind erforderlich, damit das Feature „Windows-Container“ funktioniert. Installieren Sie alle Updates, bevor Sie dieses Tutorial durcharbeiten.

Wenn Sie das Feature in Azure bereitstellen möchten, können Sie diese [Vorlage](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template) zu Hilfe nehmen.<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Flive%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>


## <a name="1-install-docker"></a>1. Installieren von Docker

Zur Installation von Docker verwenden wir das [PowerShell-Modul von OneGet](https://github.com/oneget/oneget), das mit „Providern” zur Ausführung der Installation funktioniert, in diesem Fall dem [MicrosoftDockerProvider](https://github.com/OneGet/MicrosoftDockerProvider). Der Anbieter aktiviert das Feature „Container“ auf Ihrem Computer. Außerdem installieren Sie Docker, was einen Neustart erforderlich macht. Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client.

Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und führen Sie die folgenden Befehle aus.

Installieren Sie zuerst den Docker-Microsoft-PackageManagement-Anbieter aus der [PowerShell-Galerie](https://www.powershellgallery.com/packages/DockerMsftProvider).

```
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Als Nächstes verwenden Sie das PackageManagement-PowerShell-Modul, um die neueste Docker-Version zu installieren.
```
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Wenn PowerShell fragt, ob die Paketquelle „DockerDefault“ vertrauenswürdig ist, geben Sie `A` ein, um die Installation fortzusetzen. Wenn die Installation abgeschlossen ist, starten Sie den Computer neu.

```
Restart-Computer -Force
```

> Tipp: Wenn Sie Docker später aktualisieren möchten:
>  - Überprüfen Sie die installierte Version mit `Get-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Suchen Sie die aktuelle Version mit `Find-Package -Name Docker -ProviderName DockerMsftProvider`
>  - Sobald Sie bereit sind, aktualisieren Sie mithilfe von `Install-Package -Name Docker -ProviderName DockerMsftProvider -Update -Force`, gefolgt von `Start-Service Docker`

## <a name="2-install-windows-updates"></a>2. Installieren von Windows Updates

Stellen Sie sicher, dass Ihr Windows Server System auf dem neuesten Stand ist, indem Sie folgendes ausführen:

```
sconfig
```

Dadurch wird ein textbasiertes Konfigurationsmenü angezeigt, über das Sie die Option 6 „Updates herunterladen und installieren“ auswählen können:

```
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

## <a name="3-deploy-your-first-container"></a>3. Bereitstellen Ihres ersten Containers

Für diese Übung laden Sie ein vorab erstelltes .NET-Beispielimage von der Docker Hub-Registrierung herunter und stellen einen einfachen Container bereit, der eine Hello World .NET-Anwendung ausführt.  

Stellen Sie den .Net-Container mit `docker run` bereit. Dabei wird auch das Containerimage heruntergeladen, was mehrere Minuten dauern kann.

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver
```

Der Container startet, gibt die „Hello World“-Nachricht aus und wird beendet.

```console
         Dotnet-bot: Welcome to using .NET Core!
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
Platform: .NET Core 1.0
OS: Microsoft Windows 10.0.14393
```

Weitere Informationen zum Befehl „Docker Run“ finden Sie in der [Referenz zu „Docker Run“ auf Docker.com]( https://docs.docker.com/engine/reference/run/).

## <a name="next-steps"></a>Nächste Schritte

[Automatisieren von Builds und Speichern von Images](./quick-start-images.md)
