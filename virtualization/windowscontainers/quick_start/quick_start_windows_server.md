---
title: Windows-Container unter Windows Server
description: "Containerbereitstellung – Schnellstart"
keywords: Docker, Container
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 0fae34a5a85678a25c47b0312650e67aa6cd7efd
ms.openlocfilehash: 4d02d6423cc910c2bd2fe0691cbb62bddcabb117

---

# Windows-Container unter Windows Server

Die Übung führt durch die einfache Bereitstellung und Verwendung des Windows-Container-Features unter Windows Server. Nach der Ausführung haben Sie die Containerrolle installiert und einen einfachen Windows Server-Container bereitgestellt. Machen Sie sich vor diesem Schnellstart mit grundlegenden Containerkonzepten und der Terminologie vertraut. Diese Informationen finden Sie unter [Windows Containers Quick Start](./quick_start.md) (Windows-Container – Schnellstart).

Dieser Schnellstart ist spezifisch für Windows Server-Container unter Windows Server 2016. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis.

**Voraussetzungen:**

Ein Computersystem (physisch oder virtuell), auf dem Windows Server 2016 ausgeführt wird Wenn Sie Windows Server 2016 TP5 verwenden, updaten Sie auf [Windows Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ). 

> Wichtige Updates sind erforderlich, damit das Feature „Windows-Container“ funktioniert. Installieren Sie alle Updates, bevor Sie dieses Tutorial durcharbeiten.

## 1. Installieren des Containerfeatures

Das Containerfeature muss aktiviert werden, bevor Sie mit Windows-Containern arbeiten können. Führen Sie dazu den folgenden Befehl in einer PowerShell-Sitzung mit erhöhten Rechten aus.

```none
Install-WindowsFeature containers
```

Starten Sie den Computer neu, wenn die Installation des Features abgeschlossen ist.

```none
Restart-Computer -Force
```

## 2. Installieren von Docker

Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Für diese Übung werden beide installiert.

Laden Sie die Release Candidate-Version der Commercially Supported Docker Engine und des Clients als ZIP-Archiv herunter.

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Erweitern Sie das ZIP-Archiv in „Programme“.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Fügen Sie das Docker-Verzeichnis dem Systempfad hinzu.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Führen Sie den folgenden Befehl aus, um Docker als Windows-Dienst zu installieren.

```none
dockerd.exe --register-service
```

Nach Abschluss der Installation kann der Dienst gestartet werden.

```none
Start-Service docker
```

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


<!--HONumber=Sep16_HO5-->


