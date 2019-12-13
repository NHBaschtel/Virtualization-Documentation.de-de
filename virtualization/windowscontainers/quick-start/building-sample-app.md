---
title: Containerisieren einer .net Core-App
description: Erfahren Sie, wie Sie eine .net Core-Beispiel-App mit Containern erstellen
keywords: Docker, Container
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910140"
---
# <a name="containerize-a-net-core-app"></a>Containerisieren einer .net Core-App

In diesem Thema wird beschrieben, wie Sie eine vorhandene .NET-Beispiel-App für die Bereitstellung als Windows-Container Verpacken, nachdem Sie Ihre Umgebung wie unter Erste Schritte [: Vorbereiten von Windows für Container](set-up-environment.md)und ausführen Ihres ersten Containers (wie unter [Ausführen Ihres ersten Windows-Containers](run-your-first-container.md)beschrieben) eingerichtet haben.

Außerdem muss das git-Quell Code Verwaltungssystem auf dem Computer installiert sein. Informationen zur Installation finden Sie unter [git](https://git-scm.com/download).

## <a name="clone-the-sample-code-from-github"></a>Klonen des Beispielcodes von GitHub

Der gesamte Container Beispiel-Quellcode wird in einem Ordner namens "`windows-container-samples`" unter dem git-Repository für die [virtualisierungsdokumentation](https://github.com/MicrosoftDocs/Virtualization-Documentation) (als informell bezeichnet) gespeichert.

1. Öffnen Sie eine PowerShell-Sitzung, und ändern Sie die Verzeichnisse in den Ordner, in dem Sie dieses Repository speichern möchten. (Andere Eingabe Aufforderungs Fenstertypen funktionieren ebenfalls, aber in den Beispiel Befehlen wird PowerShell verwendet.)
2. Klonen Sie das Repository in Ihr Aktuelles Arbeitsverzeichnis:

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. Navigieren Sie zu dem unter `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` gefundenen Beispiel Verzeichnis, und erstellen Sie mithilfe der folgenden Befehle eine dockerfile-Datei.

   Eine [dockerfile-Datei](https://docs.docker.com/engine/reference/builder/) ist wie ein Makefile-– eine Liste mit Anweisungen, die der Container-Engine mitteilt, wie das Container Image erstellt werden soll.

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Dockerfile-Datei schreiben

Öffnen Sie die dockerfile-Datei, die Sie soeben erstellt haben, mit dem Text-Editor, und fügen Sie den folgenden Inhalt hinzu:

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

Lassen Sie uns den Vorgang zeilenweise durchgehen und erläutern, was jede Anleitung bewirkt.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

Die erste Gruppe von Zeilen deklariert, auf Grundlage von welchem Basisimage unser Container erstellt wird. Wenn das lokale System nicht bereits über dieses Image verfügt, versucht Docker automatisch, es abzurufen. Die `mcr.microsoft.com/dotnet/core/sdk:2.1` wird mit installiertem .net Core 2,1 SDK verpackt und ist daher die Aufgabe, ASP .net Core-Projekte mit der Zielversion 2,1 zu entwickeln. Mit der nächsten Anweisung wird das Arbeitsverzeichnis in unserem Container so geändert, dass es `/app`wird, sodass alle Befehle, die dieser Anweisung folgen, in diesem Kontext ausgeführt werden.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

Anschließend kopieren diese Anweisungen die csproj-Dateien in das `/app` Verzeichnis des `build-env` Containers. Nachdem Sie diese Datei kopiert haben, liest .net Sie aus und ruft dann alle Abhängigkeiten und Tools ab, die von dem Projekt benötigt werden.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

Nachdem .net alle Abhängigkeiten in den `build-env`-Container gezogen hat, werden bei der nächsten Anweisung alle Projekt Quelldateien in den Container kopiert. Wir informieren .net dann, dass Sie die Anwendung mit einer Releasekonfiguration veröffentlichen und den Ausgabepfad in der angeben.

Die Kompilierung sollte erfolgreich sein. Nun müssen wir das endgültige Image erstellen. 

> [!TIP]
> In dieser Schnellstartanleitung wird ein .net Core-Projekt aus der Quelle erstellt. Beim Aufbau von Container Images empfiehlt es sich, _nur_ die Produktions Nutzlast und ihre Abhängigkeiten in das Container Image einzubeziehen. Wir möchten nicht, dass das .net Core SDK in unser endgültiges Image integriert ist, da wir nur die .net Core-Laufzeit benötigen. Daher wird die dockerfile-Datei so geschrieben, dass Sie einen temporären Container verwendet, der mit dem SDK namens `build-env` verpackt ist, um die APP zu erstellen.

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

Da es sich bei der Anwendung um ASP.net handelt, geben wir ein Bild an, das diese Laufzeit enthält. Anschließend kopieren wir alle Dateien aus dem Ausgabeverzeichnis unseres temporären Containers in unseren finalen Container. Wir konfigurieren den Container so, dass er mit unserer neuen App als EntryPoint ausgeführt wird, wenn der Container gestartet wird.

Wir haben die dockerfile-Datei geschrieben, um einen _mehrstufigen Build_auszuführen. Wenn die dockerfile-Datei ausgeführt wird, wird der temporäre Container (`build-env`) mit dem .net Core 2,1 SDK verwendet, um die Beispiel-APP zu erstellen und dann die ausgegebenen Binärdateien in einen anderen Container zu kopieren, der nur die .net Core 2,1-Laufzeit enthält, sodass wir die Größe des letzten Containers minimiert haben.

## <a name="build-and-run-the-app"></a>Erstellen und Ausführen der App

Nachdem Sie die dockerfile-Datei geschrieben haben, können wir docker auf unsere dockerfile-Datei verweisen und Sie darüber informieren, das Image zu erstellen und dann auszuführen:

1. Navigieren Sie in einem Eingabe Aufforderungs Fenster zu dem Verzeichnis, in dem sich die dockerfile befindet, und führen Sie dann den Befehl [docker Build](https://docs.docker.com/engine/reference/commandline/build/) aus, um den Container aus der dockerfile-Datei zu erstellen.

   ```Powershell
   docker build -t my-asp-app .
   ```

2. Führen Sie den Befehl [docker Run](https://docs.docker.com/engine/reference/commandline/run/) aus, um den neu erstellten Container auszuführen.

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   Wir übernehmen diesen Befehl:

   * `-d` mitteilt, dass docker-tut den Container "getrennt" ausführen, was bedeutet, dass keine Konsole mit der Konsole im Container verknüpft ist. Der Container wird im Hintergrund ausgeführt. 
   * `-p 5000:80` weist docker an, Port 5000 auf dem Host dem Port 80 im Container zuzuordnen. Jeder Container erhält seine eigene IP-Adresse. ASP .net lauscht standardmäßig an Port 80. Mithilfe der Port Zuordnung können wir die IP-Adresse des Hosts am zugeordneten Port aufrufen, und docker führt den gesamten Datenverkehr an den Zielport innerhalb des Containers weiter.
   * `--name myapp` weist docker an, diesem Container einen geeigneten Namen für die Abfrage zu übergeben (anstatt die von Docker zur Laufzeit zugewiesene kontaienr-ID zu überprüfen).
   * `my-asp-app` ist das Image, das von Docker ausgeführt werden soll. Dies ist das Container Image, das als Höhepunkt des `docker build` Prozesses erzeugt wird.

3. Öffnen Sie einen Webbrowser Webbrowser, und navigieren Sie zu `http://localhost:5000`, um Ihre containerisierte Anwendung anzuzeigen, wie in diesem Screenshot gezeigt:

   >![ASP.net Core Webseite, die auf dem localhost in einem Container ausgeführt wird](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>Nächste Schritte

1. Der nächste Schritt besteht darin, ihre containerisierte ASP.net-Web-App mit Azure Container Registry in einer privaten Registrierung zu veröffentlichen. Auf diese Weise können Sie Sie in Ihrer Organisation bereitstellen.

   > [!div class="nextstepaction"]
   > [Erstellen einer privaten Container Registrierung](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   Wenn Sie zu dem Abschnitt gelangen, in dem Sie [Ihr Container Image per Push in die Registrierung über](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry)setzen, geben Sie den Namen der ASP.net-APP an, die Sie soeben gepackt haben (`my-asp-app`), zusammen mit ihrer Container Registrierung (z. b. `contoso-container-registry`):

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   Weitere App-Beispiele und die zugehörigen dockerfiles-Dateien finden Sie unter [zusätzliche Container Beispiele](../samples.md).

2. Nachdem Sie Ihre APP in der Container Registrierung veröffentlicht haben, besteht der nächste Schritt in der Bereitstellung der app in einem Kubernetes-Cluster, den Sie mit Azure Kubernetes Service erstellen.

   > [!div class="nextstepaction"]
   > [Schnellstart: Bereitstellen eines AKS-Clusters (Azure Kubernetes Service) über die Azure-Befehlszeilenschnittstelle](https://docs.microsoft.com/azure/aks/windows-container-cli)
