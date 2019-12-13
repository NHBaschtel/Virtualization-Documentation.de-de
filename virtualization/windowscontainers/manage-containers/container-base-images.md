---
title: Windows-Container-Basis Images
description: Eine Übersicht über die Windows-Containerbasis Images und deren Verwendung.
keywords: Docker, Container, Hashes
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 2a69fbace51589cce08476bd68fdb5c34a7907e6
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909780"
---
# <a name="container-base-images"></a>Container Basis Images

Windows bietet vier Containerbasis Images, aus denen Benutzer erstellen können. Jedes Basis Image unterscheidet sich vom Windows-Betriebssystem, unterscheidet sich von Datenträgern und unterscheidet sich von der Größe des Windows-API-Satzes.

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-servercore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows Server Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Unterstützt herkömmliche .NET Framework-Anwendungen.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-nanoserver" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Nano Server</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Basiert auf .net Core-Anwendungen.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Stellt den vollständigen Windows-API-Satz bereit.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-quarter has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="https://hub.docker.com/_/microsoft-windows-iotcore" data-linktype="external">
            <article class="card has-outline-hover is-relative is-full-height has-padding-none">
                    <div class="cardImageOuter bgdAccent1 has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                        <div class="cardImage centered has-padding-top-large has-padding-bottom-large has-padding-left-large has-padding-right-large">
                            <img src="media/Microsoft_logo.svg" alt="" data-linktype="relative-path">
                        </div>
                    </div>
                <div class="card-content has-text-overflow-ellipsis has-padding-top-small">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Windows IOT Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Speziell für IOT-Anwendungen.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>Image Ermittlung

Alle Windows-Container-Basis Images können über [docker Hub](https://hub.docker.com/_/microsoft-windows-base-os-images)erkannt werden. Die Windows-Containerbasis Images selbst werden von [MCR.Microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/), dem Microsoft Container Registry (MCR), bedient. Aus diesem Grund sehen die Pull-Befehle für die Windows-Containerbasis Images wie folgt aus:

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

Die MCR verfügt nicht über eine eigene Katalog Umgebung und soll vorhandene Kataloge wie docker Hub unterstützen. Dank des globalen Speicherplatzes von Azure und der Verbindung mit Azure CDN bietet MCR eine konsistente und schnelle Image Pull-Funktion. Azure-Kunden, die ihre Workloads in Azure ausführen, profitieren von den Leistungsverbesserungen im Netzwerk sowie von der engen Integration mit dem MCR (der Quelle für Microsoft-Container Images), Azure Marketplace und der wachsenden Anzahl von Diensten in Azure, die Container als Bereitstellungs Paketformat.

## <a name="choosing-a-base-image"></a>Auswählen eines Basis Images

Wie wählen Sie das richtige Basis Image aus, auf dem erstellt werden soll? Für die meisten Benutzer sind `Windows Server Core` und `Nanoserver` das geeignetste zu verwendende Image.

### <a name="guidelines"></a>Richtlinien

 Obwohl Sie das gewünschte Image als Ziel verwenden können, finden Sie hier einige Richtlinien, die Ihnen helfen, Ihre Wahl zu steuern:

- **Ist für Ihre Anwendung das vollständige .NET Framework erforderlich?** Wenn die Antwort auf diese Frage "yes" lautet, sollten Sie `Windows Server Core`als Ziel festlegen.
- **Entwickeln Sie eine Windows-APP, die auf .net Core basiert?** Wenn die Antwort auf diese Frage "yes" lautet, sollten Sie `Nanoserver`als Ziel festlegen.
- **Wird eine IOT-Anwendung aufgebaut?** Wenn die Antwort auf diese Frage "yes" lautet, sollten Sie `IoT Core`als Ziel festlegen.
- **Fehlt dem Windows Server Core-Container Image eine Abhängigkeit, die Ihre APP benötigt?** Wenn die Antwort auf diese Frage "yes" lautet, sollten Sie versuchen, auf `Windows`zu Zielen. Dieses Bild ist wesentlich größer als die anderen Basis Images, aber es enthält viele der Windows-Kernbibliotheken (z. b. die GDI-Bibliothek).
- **Sind Sie Windows-Insider?** Wenn ja, sollten Sie die Verwendung der Insider-Version der Images in Erwägung gezogen. Weitere Informationen finden Sie unten unter "Basis Images für Windows-Insider".

> [!TIP]
> Viele Windows-Benutzer möchten Anwendungen, die eine Abhängigkeit von .net aufweisen, containerisieren. Zusätzlich zu den vier hier beschriebenen Basis Images veröffentlicht Microsoft mehrere Windows-Container Images, die mit beliebten Microsoft-Frameworks, wie z. b. dem [.NET Framework](https://hub.docker.com/_/microsoft-dotnet-framework) -Image und dem [ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/) -Image, vorkonfiguriert sind.

### <a name="base-images-for-windows-insiders"></a>Basis Images für Windows-Insider

Microsoft stellt "Insider"-Versionen für jedes Containerbasis Image bereit. Diese Insider-Container Images enthalten die neueste und größte Featureentwicklung in unseren Container Images. Beim Ausführen eines Hosts, bei dem es sich um eine Insider Version von Windows handelt (entweder Windows Insider oder Windows Server Insider), empfiehlt es sich, diese Images zu verwenden. Die Insider-Images sind auf dem docker-Hub verfügbar:

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

Weitere Informationen finden Sie unter [Verwenden von Containern mit dem Windows-Insider-Programm](../deploy-containers/insider-overview.md) .

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core im Vergleich zu NanoServer

`Windows Server Core` und `Nanoserver` sind die gängigsten Basis Images, die als Ziel dienen. Der Hauptunterschied zwischen diesen Images besteht darin, dass NanoServer eine deutlich kleinere API-Oberfläche aufweist. PowerShell, WMI und der Windows-Wartungs Stapel fehlen im NanoServer-Image.

NanoServer wurde erstellt, um genau genug API-Oberfläche zum Ausführen von apps bereitzustellen, die eine Abhängigkeit von .net Core oder anderen modernen Open Source-Frameworks aufweisen. Als Kompromiss zur kleineren API-Oberfläche hat das NanoServer-Image einen erheblich geringeren Speicherbedarf als die restlichen Windows-Basis Images. Aufbauend auf Nano Server können Sie nach Bedarf jederzeit Ebenen hinzufügen. Ein Beispiel hierfür finden Sie unter [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).
