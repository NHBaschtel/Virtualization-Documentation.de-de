---
title: Verlauf für Windows-Containerbasisimages
description: Eine Liste der mit SHA256-Layer-Hashes veröffentlichten Windows-Containerimages
keywords: Docker, Container, Hashes
author: patricklang
ms.date: 01/12/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: b2f2d6418fdda2ad0aa0b81c05efad6b99f74375
ms.sourcegitcommit: 73134bf279f3ed18235d24ae63cdc2e34a20e7b7
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 09/12/2019
ms.locfileid: "10107904"
---
# <a name="container-base-images"></a>Container-Basisbilder

## <a name="supported-base-images"></a>Unterstützte Basisbilder

Windows-Container werden mit vier Container-Basisbildern angeboten: Windows Server Core, Nano Server, Windows und vieles mehr. Nicht alle Konfigurationen unterstützen beide Betriebssystemimages. Diese Tabelle enthält Details zu den unterstützten Konfigurationen.

|Host-Betriebssystem|Windows-Container|Hyper-V-Isolierung|
|---------------------|-----------------|-----------------|
|Windows Server 2016 oder Windows Server 2019 (Standard oder Datacenter)|Server Core, Nano Server, Windows|Server Core, Nano Server, Windows|
|Nano Server|Nano Server|Server Core, Nano Server, Windows|
|Windows 10 pro oder Windows 10 Enterprise|Nicht verfügbar|Server Core, Nano Server, Windows|
|IoT Core|IoT Core|Nicht verfügbar|

> [!WARNING]  
> Ab Windows Server, Version 1709, steht Nano Server nicht mehr als Container Host zur Verfügung.

## <a name="base-image-differences"></a>Grundlegende Bild Unterschiede

Wie wählt man das richtige Basis Bild aus, auf dem aufgebaut werden soll? Zwar können Sie mit dem, was Sie möchten, zusammenbauen, doch dies sind die allgemeinen Richtlinien für jedes Bild:

- [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): Wenn Ihre Anwendung das vollständige .NET Framework benötigt, ist dies das beste zu verwendende Bild.
- [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): für Anwendungen, die nur .net Core erfordern, bietet Nano Server ein wesentlich schlankeres Bild.
- [Windows](https://hub.docker.com/_/microsoft-windowsfamily-windows): Sie können feststellen, dass Ihre Anwendung von einer Komponente oder DLL abhängt, die in Server Core-oder Nano-Server Abbildern fehlt, beispielsweise in GDI-Bibliotheken. Dieses Bild enthält den vollständigen abhängigkeitssatz von Windows.
- Viel [Core](https://hub.docker.com/_/microsoft-windows-iotcore): dieses Bild ist speziell für viele [Anwendungen](https://developer.microsoft.com/windows/iot)konzipiert. Sie sollten dieses Container Bild verwenden, wenn Sie auf einen viel Core-Host abzielen.

Für die meisten Benutzer ist Windows Server Core oder Nano Server das am besten geeignete Bild, das Sie verwenden können. Im folgenden finden Sie einige Punkte, die Sie berücksichtigen sollten, wenn Sie über den NanoServer auf dem neuesten Stand sind:

- Der Bereitstellungsstapel wurde entfernt.
- .NET Core ist nicht enthalten (Sie können jedoch das [.NET Core Nano Server-Image](https://hub.docker.com/r/microsoft/dotnet/) verwenden).
- PowerShell wurde entfernt.
- WMI wurde entfernt.
- Ab Windows Server der Version 1709 werden Anwendungen unter einem Benutzerkontext ausgeführt, sodass Befehle, die Administratorrechte erfordern, fehlschlagen. Sie können das Container-Administratorkonto über das--User-Flag (wie docker-Lauf-Benutzer-ContainerAdministrator) angeben, doch in Zukunft beabsichtigen wir, Administratorkonten vollständig aus dem Server zu entfernen.

Dies ist lediglich eine Übersicht über die wichtigsten Unterschiede und keine vollständige Liste. Es gibt weitere Komponenten, die hier nicht genannt sind, die ebenfalls nicht vorhanden sind. Aufbauend auf Nano Server können Sie nach Bedarf jederzeit Ebenen hinzufügen. Ein Beispiel hierfür finden Sie unter [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).
