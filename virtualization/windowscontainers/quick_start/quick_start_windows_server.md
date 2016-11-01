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
translationtype: Human Translation
ms.sourcegitcommit: 6c85bb2ac3922dac4b09939d3ea71d7fbb5e16ad
ms.openlocfilehash: d06f38ddc9abf40a2842089203462c26765be672

---

# Windows-Container unter Windows Server

Die Übung führt durch die einfache Bereitstellung und Verwendung des Windows-Container-Features unter Windows Server. Nach der Ausführung haben Sie die Containerrolle installiert und einen einfachen Windows Server-Container bereitgestellt. Machen Sie sich vor diesem Schnellstart mit grundlegenden Containerkonzepten und der Terminologie vertraut. Diese Informationen finden Sie unter [Windows Containers Quick Start](./quick_start.md) (Windows-Container – Schnellstart).

Dieser Schnellstart ist spezifisch für Windows Server-Container unter Windows Server 2016. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis.

**Voraussetzungen:**

Ein Computersystem (physisch oder virtuell), auf dem Windows Server 2016 ausgeführt wird Wenn Sie Windows Server 2016 TP5 verwenden, updaten Sie auf [Windows Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ). 

> Wichtige Updates sind erforderlich, damit das Feature „Windows-Container“ funktioniert. Installieren Sie alle Updates, bevor Sie dieses Tutorial durcharbeiten.

## 1. Installieren von Docker

Verwenden Sie das [PowerShell-Modul von OneGet](https://github.com/oneget/oneget), um Docker zu installieren. Der Anbieter aktiviert die Containerfunktion auf Ihrem Computer und installiert Docker. Dies macht einen Neustart erforderlich. Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client.

Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und führen Sie die folgenden Befehle aus.

Zunächst installieren Sie das PowerShell-Modul von OneGet.

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

Als Nächstes verwenden Sie OneGet, um die neueste Version von Docker zu installieren.
```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Wenn PowerShell fragt, ob die Paketquelle „DockerDefault“ vertrauenswürdig ist, geben Sie „A“ ein, um die Installation fortzusetzen. Wenn die Installation abgeschlossen ist, starten Sie den Computer neu.

```none
Restart-Computer -Force
```

## 2. Installieren von Windows-Updates

Sie sollten Windows-Updates installieren, um sicherzustellen, dass Ihr Windows Server-System auf dem neuesten Stand ist. Führen Sie hierzu Folgendes aus:

```none
sconfig
```

Ihnen wird ein textbasiertes Konfigurationsmenü angezeigt, über das Sie die Option 6 „Updates herunterladen und installieren“ auswählen können:

```none
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

## 3. Bereitstellen Ihres ersten Containers

Für diese Übung laden Sie ein vorab erstelltes .NET-Beispielimage von der Docker Hub-Registrierung herunter und stellen einen einfachen Container bereit, der eine Hello World .NET-Anwendung ausführt.  

Verwenden Sie `docker run`, um den .NET-Container bereitzustellen. Dabei wird auch das Containerimage heruntergeladen, was mehrere Minuten dauern kann.

```none
docker run microsoft/sample-dotnet
```

Der Container startet, gibt die Hello World-Nachricht aus und wird beendet.

```none
       Welcome to .NET Core!
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
```

Weitere Informationen zum Befehl „Docker Run“ finden Sie in der [Referenz zu „Docker Run“ auf Docker.com]( https://docs.docker.com/engine/reference/run/).

## Nächste Schritte

[Containerimages unter Windows Server](./quick_start_images.md)

[Windows-Container unter Windows 10](./quick_start_windows_10.md)



<!--HONumber=Oct16_HO4-->


