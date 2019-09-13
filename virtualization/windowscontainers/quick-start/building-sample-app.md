---
title: Containerisieren einer .net Core-App
description: Erfahren Sie, wie Sie eine .net Core-Beispiel-App mit Containern erstellen.
keywords: Docker, Container
author: cwilhit
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 8165d9c7ee3744fae31711e28be028208140813e
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129270"
---
# <a name="containerize-a-net-core-app"></a>Containerisieren einer .net Core-App

In diesem Segment wird davon ausgegangen, dass Ihre Entwicklungsumgebung bereits für Container konfiguriert ist. Wenn Sie keine Umgebung für Container konfiguriert haben, besuchen Sie "[Einrichten Ihrer Umgebung](./set-up-environment.md)", um zu erfahren, wie Sie beginnen können.

Auf Ihrem Computer muss das git-Quellcodeverwaltungssystem installiert sein. Sie können es hier packen: [git](https://git-scm.com/download)

## <a name="clone-the-sample-code"></a>Klonen des Beispielcodes

Der gesamte Container Beispiel-Quellcode wird unter der [Virtualization-Documentation](https://github.com/MicrosoftDocs/Virtualization-Documentation) git Repo in einem Ordner namens `windows-container-samples`gespeichert. Klonen Sie dieses git Repo in Ihr curent-Arbeitsverzeichnis.

```Powershell
git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
```

Navigieren Sie zu dem Beispielverzeichnis unter `<directory where clone occured>\Virtualization-Documentation\windows-container-samples\asp-net-getting-started` , und erstellen Sie einen Dockerfile. Eine [Dockerfile](https://docs.docker.com/engine/reference/builder/) ist wie ein Makefile – eine Liste mit Anweisungen, die das Containermodul anweisen, wie das Container Bild erstellt werden muss.

```Powershell
#Create the dockerfile for our project
New-Item -name dockerfile -type file
```

## <a name="write-the-dockerfile"></a>Schreiben des dockerfile

Öffnen Sie das soeben erstellte dockerfile (mit dem jeweils gewünschten Text-Editor), und fügen Sie den folgenden Inhalt hinzu.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Wir brechen es Zeile für Zeile ab und erläutern, was die einzelnen Anweisungen tun.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

Die erste Gruppe von Zeilen deklariert, auf Grundlage von welchem Basisimage unser Container erstellt wird. Wenn das lokale System nicht bereits über dieses Image verfügt, versucht Docker automatisch, es abzurufen. Das `mcr.microsoft.com/dotnet/core/sdk:2.1` Paket "kommt" wird mit dem .net Core 2,1 SDK installiert, daher ist es Aufgabe des Erstellens von ASP .net Core-Projekten, die auf Version 2,1 ausgerichtet sind. Mit der nächsten Anweisung wird das Arbeitsverzeichnis in unserem Container so `/app`geändert, dass alle folgenden Befehle in diesem Kontext ausgeführt werden.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

Als Nächstes kopieren diese Anweisungen die csproj-Dateien in das `build-env` Container `/app` Verzeichnis. Nach dem Kopieren dieser Datei liest .net davon und ruft dann alle Abhängigkeiten und Tools ab, die für unser Projekt erforderlich sind.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

Nachdem .net alle Abhängigkeiten in den `build-env` Container gezogen hat, werden in der nächsten Anweisung alle Projekt Quelldateien in den Container kopiert. Wir empfehlen .net dann, unsere Anwendung mit einer Release-Konfiguration zu veröffentlichen und den Ausgabepfad in der.

Die Kompilierung sollte erfolgreich sein. Nun müssen wir das endgültige Bild erstellen. 

> [!TIP]
> Dieser Schnellstart erstellt ein .net Core-Projekt aus der Quelle. Beim Erstellen von Container Bildern empfiehlt es sich, _nur_ die Produktions Nutzlast und deren Abhängigkeiten in das Container Bild einzubeziehen. Wir möchten nicht, dass das .net Core-SDK in unserem endgültigen Bild enthalten ist, da wir nur die .net Core-Laufzeit benötigen, damit der dockerfile für die Verwendung eines temporären Containers geschrieben `build-env` wird, der mit dem SDK, das zum Erstellen der app aufgerufen wurde, verpackt ist.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Da es sich bei unserer Anwendung um ASP.net handelt, geben wir ein Bild an, in dem diese Runtime enthalten ist. Anschließend kopieren wir alle Dateien aus dem Ausgabeverzeichnis unseres temporären Containers in unseren finalen Container. Wir konfigurieren unseren Container so, dass er mit der neuen App als EntryPoint ausgeführt wird, wenn der Container gestartet wird.

Wir haben die dockerfile zum Ausführen eines _mehrstufigen Builds_geschrieben. Wenn die dockerfile ausgeführt wird, verwendet Sie den temporären Container `build-env`, mit dem .net Core 2,1 SDK, um die Beispiel-APP zu erstellen, und kopieren Sie dann die ausgegebenen Binärdateien in einen anderen Container, der nur die .net Core 2,1-Laufzeit enthält, sodass wir die Größe des Letzter Container.

## <a name="run-the-app"></a>Ausführen der App

Mit dem geschriebenen dockerfile können wir andocker auf unser dockerfile verweisen und es anweisen, unser Bild zu erstellen. 

>[!IMPORTANT]
>Der unten ausgeführte Befehl muss in dem Verzeichnis ausgeführt werden, in dem sich die dockerfile befindet.

```Powershell
docker build -t my-asp-app .
```

Führen Sie den folgenden Befehl aus, um den Container auszuführen.

```Powershell
docker run -d -p 5000:80 --name myapp my-asp-app
```

Lassen Sie uns diesen Befehl sezieren:

* `-d` weist andocker tun, dass der Container "losgelöst" ausgeführt wird, was bedeutet, dass keine Konsole an die Konsole im Container angeschlossen ist. Der Container wird im Hintergrund ausgeführt. 
* `-p 5000:80` weist andocker an, Port 5000 auf dem Host Port 80 im Container zuzuordnen. Jeder Container erhält seine eigene IP-Adresse. ASP .net hört standardmäßig auf Port 80. Port-Mapping ermöglicht es uns, zur IP-Adresse des Hosts am zugeordneten Port zu wechseln, und andocker leitet den gesamten Datenverkehr an den Ziel-Port innerhalb des Containers weiter.
* `--name myapp` weist andocker an, diesem Container einen praktischen Namen für die Abfrage zu geben (anstatt die contaienr-ID nachschlagen zu müssen, die von Docker zur Laufzeit zugewiesen wurde).
* `my-asp-app` ist das Bild, das von Docker ausgeführt werden soll. Dies ist das Container Bild, das als Höhepunkt des `docker build` Prozesses erstellt wurde.

Öffnen Sie einen Webbrowser Webbrowser, und navigieren Sie `https://localhost:5000` zu, um von ihrer Containeranwendung begrüßt zu werden.

>![](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>Nächste Schritte

Wir haben erfolgreich eine ASP.net-Web-App Containern. Wenn Sie weitere App-Beispiele und deren zugeordnete dockerfiles anzeigen möchten, klicken Sie auf die Schaltfläche unten.

> [!div class="nextstepaction"]
> [Weitere Container Beispiele finden Sie hier.](../samples.md)
