---
title: Containerisieren einer .net Core-App
description: Erfahren Sie, wie Sie eine .net Core-Beispiel-App mit Containern erstellen.
keywords: Docker, Container
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288068"
---
# <a name="containerize-a-net-core-app"></a>Containerisieren einer .net Core-App

In diesem Thema wird beschrieben, wie Sie eine vorhandene Beispiel-.net-App für die Bereitstellung als Windows-Container Verpacken, nachdem Sie Ihre Umgebung wie unter erste [Schritte: Vorbereiten von Windows für Container](set-up-environment.md)beschrieben eingerichtet haben, und führen Sie den ersten Container wie unter [Ausführen des ersten Windows-Containers](run-your-first-container.md)beschrieben aus.

Außerdem müssen Sie das git-Quellcodeverwaltungssystem auf Ihrem Computer installiert haben. Besuchen Sie [git](https://git-scm.com/download), um es zu installieren.

## <a name="clone-the-sample-code-from-github"></a>Klonen des Beispielcodes aus GitHub

Der gesamte Container Beispiel-Quellcode wird unter dem git-Repository für [Virtualisierungs-Dokumentation](https://github.com/MicrosoftDocs/Virtualization-Documentation) (bekannt als Repo-Datei) in einem Ordner `windows-container-samples`namens gespeichert.

1. Öffnen Sie eine PowerShell-Sitzung, und ändern Sie die Verzeichnisse in den Ordner, in dem Sie dieses Repository speichern möchten. (Andere Eingabeaufforderungsfenster Typen funktionieren ebenfalls, aber unsere Beispielbefehle verwenden PowerShell.)
2. Klonen Sie das Repo in Ihr Aktuelles Arbeitsverzeichnis:

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. Navigieren Sie mit den folgenden Befehlen zu `Virtualization-Documentation\windows-container-samples\asp-net-getting-started` dem Beispielverzeichnis unter und erstellen Sie ein Dockerfile.

   Eine [Dockerfile](https://docs.docker.com/engine/reference/builder/) ist wie ein Makefile – eine Liste mit Anweisungen, die dem Containermodul mitteilen, wie das Container Bild erstellt wird.

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Schreiben des Dockerfile

Öffnen Sie die Dockerfile, die Sie soeben erstellt haben, mit dem Text-Editor, den Sie mögen, und fügen Sie dann den folgenden Inhalt hinzu:

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

Wir haben die dockerfile zum Ausführen eines _mehrstufigen Builds_geschrieben. Wenn die dockerfile ausgeführt wird, verwendet Sie den temporären Container `build-env`, mit dem .net Core 2,1 SDK, um die Beispiel-APP zu erstellen, und kopieren Sie dann die ausgegebenen Binärdateien in einen anderen Container, der nur die .net Core 2,1-Laufzeit enthält, sodass die Größe des endgültigen Containers minimiert wird.

## <a name="build-and-run-the-app"></a>Erstellen und Ausführen der App

Mit dem geschriebenen Dockerfile können wir andocker auf unser Dockerfile verweisen und es zum Erstellen und dann zum Ausführen des Bilds sagen:

1. Navigieren Sie in einem Eingabeaufforderungsfenster zu dem Verzeichnis, in dem sich die dockerfile befindet, und führen Sie dann den Befehl [andocker erstellen](https://docs.docker.com/engine/reference/commandline/build/) aus, um den Container aus dem dockerfile zu erstellen.

   ```Powershell
   docker build -t my-asp-app .
   ```

2. Führen Sie zum Ausführen des neu erstellten Containers den Befehl [Andocken ausführen](https://docs.docker.com/engine/reference/commandline/run/) aus.

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   Lassen Sie uns diesen Befehl sezieren:

   * `-d` weist andocker tun, dass der Container "losgelöst" ausgeführt wird, was bedeutet, dass keine Konsole an die Konsole im Container angeschlossen ist. Der Container wird im Hintergrund ausgeführt. 
   * `-p 5000:80` weist andocker an, Port 5000 auf dem Host Port 80 im Container zuzuordnen. Jeder Container erhält seine eigene IP-Adresse. ASP .net hört standardmäßig auf Port 80. Port-Mapping ermöglicht es uns, zur IP-Adresse des Hosts am zugeordneten Port zu wechseln, und andocker leitet den gesamten Datenverkehr an den Ziel-Port innerhalb des Containers weiter.
   * `--name myapp` weist andocker an, diesem Container einen praktischen Namen für die Abfrage zu geben (anstatt die contaienr-ID nachschlagen zu müssen, die von Docker zur Laufzeit zugewiesen wurde).
   * `my-asp-app` ist das Bild, das von Docker ausgeführt werden soll. Dies ist das Container Bild, das als Höhepunkt des `docker build` Prozesses erstellt wurde.

3. Öffnen Sie einen Webbrowser Webbrowser, und navigieren Sie `http://localhost:5000` zu ihrer Containeranwendung, wie in diesem Screenshot gezeigt:

   >![ASP.net Core-Webseite, die vom localhost in einem Container ausgeführt wird](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>Nächste Schritte

1. Der nächste Schritt besteht darin, die Container-ASP.net-Web-App in einer privaten Registrierung unter Verwendung der Azure-Container Registrierung zu veröffentlichen. Auf diese Weise können Sie Sie in Ihrer Organisation bereitstellen.

   > [!div class="nextstepaction"]
   > [Erstellen einer privaten Container Registrierung](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   Wenn Sie zu dem Abschnitt gelangen, in dem Sie [Ihr Container Bild in die Registrierung schieben](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry), geben Sie den Namen der ASP.net-APP,`my-asp-app`die Sie soeben verpackt haben () zusammen mit Ihrer `contoso-container-registry`Container Registrierung an (beispielsweise:):

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   Weitere App-Beispiele und deren zugeordnete dockerfiles finden Sie unter [zusätzliche Container Beispiele](../samples.md).

2. Nachdem Sie Ihre APP in der Container Registrierung veröffentlicht haben, besteht der nächste Schritt darin, die app in einem Kubernetes-Cluster bereitzustellen, den Sie mit Azure Kubernetes-Dienst erstellen.

   > [!div class="nextstepaction"]
   > [Erstellen eines Kubernetes-Clusters](https://docs.microsoft.com/azure/aks/windows-container-cli)
