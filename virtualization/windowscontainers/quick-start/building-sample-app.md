---
title: Erstellen einer Beispiel-App
description: Hier erfahren Sie, wie Sie mithilfe von Containern eine Beispiel-App erstellen.
keywords: Docker, Container
author: cwilhit
ms.date: 07/25/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 7ffc16e9d5b7c4b4a935a06c012b1d28b5e70f1a
ms.sourcegitcommit: 27e9cd37beaf11e444767699886e5fdea5e1a2d0
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/30/2019
ms.locfileid: "10058485"
---
# <a name="build-a-sample-app"></a>Erstellen einer Beispiel-App

In dieser Übung wird gezeigt, wie Sie eine ASP.NET-Beispiel-App für die Ausführung in einem Container konvertieren. Informationen zum Einstieg mit Containern in Windows 10 erhalten Sie in der [Schnellstartanleitung für Windows 10](./quick-start-windows-10.md).

Dieser Schnellstart bezieht sich speziell auf Windows10. Weitere Schnellstartdokumentation finden Sie links auf dieser Seite im Inhaltsverzeichnis. Da sich dieses Lernprogramm auf Container konzentriert, werden wir das Schreiben von Code hier auslassen und uns nur mit Containern befassen. Das gesamte Lernprogramm finden Sie in der [ASP.Net Core-Dokumentation](https://docs.microsoft.com/aspnet/core/tutorials/first-mvc-app-xplat/).

Wenn die Git-Quellcodeverwaltung auf Ihrem Computer nicht installiert ist, finden Sie sie hier: [Git](https://git-scm.com/download).

## <a name="getting-started"></a>Erste Schritte

Dieses Beispielprojekt wurde mit [VSCode](https://code.visualstudio.com/) eingerichtet. Wir verwenden auch PowerShell. Den Democode finden wir auf GitHub. Sie können das Repository mit Git klonen oder das Projekt direkt herunterladen: [SampleASPContainerApp](https://github.com/cwilhit/SampleASPContainerApp).

```Powershell
git clone https://github.com/cwilhit/SampleASPContainerApp.git
```

Nun navigieren wir zum Projektverzeichnis und erstellen die Dockerfile-Datei. Ein [Dockerfile](https://docs.docker.com/engine/reference/builder/) ist wie ein Makefile – eine Liste von Anweisungen, die beschreiben, wie ein Containerimage erstellt werden soll.

```Powershell
#Create the dockerfile for our proj
New-Item C:/Your/Proj/Location/Dockerfile -type file
```

## <a name="writing-our-dockerfile"></a>Erstellen der Dockerfile-Datei

Nun öffnen wir (mit einem beliebigen Text-Editor) die Dockerfile-Datei, die wir im Projektstammordner erstellt haben, und fügen Logik hinzu. Dann gehen wir sie Zeile für Zeile durch, damit Sie sehen, was passiert.

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

Die erste Gruppe von Zeilen deklariert, auf Grundlage von welchem Basisimage unser Container erstellt wird. Wenn das lokale System nicht bereits über dieses Image verfügt, versucht Docker automatisch, es abzurufen. Aspnetcore-Build enthält bereits die Abhängigkeiten zum Kompilieren unseres Projekts. Anschließend ändern wir das Arbeitsverzeichnis in unserem Container in „/App“, sodass alle nachfolgenden Befehle in unserer Dockerfile-Datei dort ausgeführt werden.

>[!NOTE]
>Da wir unser Projekt erstellen müssen, handelt es sich bei diesem ersten Container, den wir erstellen, um einen temporären Container, den wir genau dazu verwenden werden, und ihn dann am Ende zu verwerfen.

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app
```

Als Nächstes kopieren wir die CSPROJ-Dateien in das Verzeichnis "/app" unseres temporären Containers. Wir tun dies, weil csproj-Dateien eine Liste der Paket Bezüge enthalten, die unser Projekt benötigt.

Nach dem Kopieren der Datei wird diese von Dotnet gelesen. Anschließend werden alle Abhängigkeiten und Tools abgerufen, die unser Projekt benötigt.

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

Nach dem Abrufen dieser Abhängigkeiten kopieren wir sie in den temporären Container. Wir weisen Dotnet anschließend an, unsere Anwendung mit einer Releasekonfiguration zu veröffentlichen, und geben den Ausgabepfad an.

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

Unser Projekt sollte nun erfolgreich kompiliert worden sein. Nun müssen wir unseren fertigen Container erstellen. Da die Anwendung ASP.NET ist, geben wir ein Image mit diesen Bibliotheken als Quelle an. Anschließend kopieren wir alle Dateien aus dem Ausgabeverzeichnis unseres temporären Containers in unseren finalen Container. Wir konfigurieren unseren Container für die Ausführung mit unserer neuen DLL, die wir beim Starten kompiliert haben.

>[!NOTE]
>Unser Basisbild für diesen letzten Container ist ähnlich, aber anders als ```FROM``` der obige Befehl--er verfügt nicht über die Bibliotheken, die eine ASP.net-APP _Erstellen_ können, sondern nur ausgeführt.

```Dockerfile
FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

Wir haben jetzt erfolgreich einen sogenannten _mehrstufigen Build_ ausgeführt. Zum Erstellen des Images haben wir den temporären Container verwendet und anschließend die veröffentlichte DLL in einen anderen Container verschoben. Dadurch wurde der letztendliche Speicherbedarf verringert. Wir möchten für die Ausführung dieses Containers nur die absolut erforderlichen Mindestabhängigkeiten festlegen. Wenn wir das erste Image verwenden würden, wären im Paket noch andere Ebenen (zum Erstellen von ASP.NET-Apps) enthalten, die nicht erforderlich sind und somit das Image nur unnötig vergrößern.

## <a name="running-the-app"></a>Ausführen der App

Nachdem die Dockerfile-Datei erstellt wurde, müssen wir Docker nur noch anweisen, die App zu erstellen und den Container auszuführen. Wir geben nun den Port zum Veröffentlichen an und vergeben das Tag „myapp“ für unseren Container. Führen Sie in PowerShell die folgenden Befehle aus.

>[!NOTE]
>Das aktuelle Arbeitsverzeichnis ihrer PowerShell-Konsole muss das Verzeichnis sein, in dem sich der oben erstellte dockerfile befindet.

```Powershell
docker build -t myasp .
docker run -d -p 5000:80 --name myapp myasp
```

Damit wir sehen, wie unsere App ausgeführt wird, müssen wir die entsprechende Adresse aufrufen. Mithilfe dieses Befehls können wir die IP-Adresse abrufen.

```Powershell
 docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" myapp
```

Wenn Sie diesen Befehl ausführen, erhalten Sie die IP-Adresse Ihres ausgeführten Containers. Hier ist ein Beispiel dafür, wie die Ausgabe aussehen könnte:

```Powershell
 172.19.172.12
```

Geben Sie diese IP-Adresse in einen beliebigen Webbrowser ein, und Sie sehen, wie die Anwendung erfolgreich in einem Container ausgeführt wird.

>![](media/SampleAppScreenshot.png)

Klicken Sie in der Navigationsleiste auf „MvcMovie“. Dadurch gelangen Sie zu einer Webseite, auf der Sie Filmeinträge eingeben, bearbeiten und löschen können.

## <a name="next-steps"></a>Nächste Schritte

Wir haben mithilfe von Docker erfolgreich eine ASP.NET-Web-App konfiguriert und erstellt und diese erfolgreich in einem ausgeführten Container bereitgestellt. Aber Sie können noch weitere Schritte ausführen. Sie können die Web-App beispielsweise in weitere Komponenten aufteilen: einen Container, der die Web-API ausführt, einen Container, der das Front-End ausführt, und einen Container, der den SQL Server ausführt.

Nachdem Sie nun die Container hängen, gehen Sie dorthin und bauen Sie tolle Container-Software!

> [!div class="nextstepaction"]
> [Weitere Container Beispiele finden Sie hier.](../samples.md)
