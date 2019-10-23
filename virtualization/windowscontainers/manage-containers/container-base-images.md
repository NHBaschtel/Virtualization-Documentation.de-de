---
title: Windows-Container-Basisbilder
description: Eine Übersicht über die Windows-Containerbasis Bilder und deren Verwendung.
keywords: Docker, Container, Hashes
author: patricklang
ms.date: 09/25/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 88e6e080-cf8f-41d8-a301-035959dc5ce0
ms.openlocfilehash: 2a69fbace51589cce08476bd68fdb5c34a7907e6
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254270"
---
# <a name="container-base-images"></a>Container-Basisbilder

Windows bietet vier Containerbasis Bilder, aus denen Benutzer erstellen können. Jedes Basisbild weist eine andere Variante des Windows-Betriebssystems auf, hat einen anderen Speicherplatz auf dem Datenträger und hat eine unterschiedliche Menge des Windows-API-Satzes.

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
                        <p>Für .net Core-Anwendungen konzipiert.</p>
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
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">WindowsIoT Core</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Speziell für viele Anwendungen konzipiert.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="image-discovery"></a>Bild Ermittlung

Alle Windows-Container-Basisbilder sind über den [docker-Hub](https://hub.docker.com/_/microsoft-windows-base-os-images)auffindbar. Die Windows-Containerbasis Bilder selbst werden von [MCR.Microsoft.com](https://azure.microsoft.com/en-us/services/container-registry/), der Microsoft-Container Registrierung ("Microsoft Container Registry"), bereitgestellt. Aus diesem Grund sehen die Pull-Befehle für die Windows-Containerbasis Bilder wie folgt aus:

```code
docker pull mcr.microsoft.com/windows/servercore:ltsc2019
```

Das "Sie"-Element verfügt nicht über eine eigene Katalog Erfahrung und unterstützt vorhandene Kataloge wie docker-Hub. Dank des globalen Footprints von Azure und einer Kombination aus Azure CDN bietet das "" "-Symbol eine Bild Zieh Erfahrung, die konsistent und schnell ist. Azure-Kunden, die ihre Arbeitslasten in Azure ausführen, profitieren von Verbesserungen in der Netzwerkleistung sowie einer engen Integration mit der Quelle für Microsoft-Container Bilder, Azure Marketplace und der wachsenden Anzahl von Diensten in Azure, die das Angebot Container als Bereitstellungspaket Format.

## <a name="choosing-a-base-image"></a>Auswählen eines Basis Bilds

Wie wählen Sie das richtige Basis Bild aus, auf dem Sie aufbauen möchten? Für die meisten Benutzer `Windows Server Core` und `Nanoserver` ist das am besten geeignete Bild, das verwendet werden soll.

### <a name="guidelines"></a>Anleitungen

 Wenn Sie das gewünschte Bild abgleichen möchten, finden Sie hier einige Richtlinien, die Ihnen helfen, Ihre Wahl zu steuern:

- **Erfordert Ihre Anwendung das vollständige .NET Framework?** Wenn die Antwort auf diese Frage "Ja" lautet, sollten `Windows Server Core`Sie eine Zielvorgabe erreichen.
- **Erstellen Sie auf der Grundlage von .net Core eine Windows-App?** Wenn die Antwort auf diese Frage "Ja" lautet, sollten `Nanoserver`Sie eine Zielvorgabe erreichen.
- **Erstellen Sie eine viel-Anwendung?** Wenn die Antwort auf diese Frage "Ja" lautet, sollten `IoT Core`Sie eine Zielvorgabe erreichen.
- **Fehlt für das Windows Server Core-Container Abbild eine Abhängigkeit, die Ihre APP benötigt?** Wenn die Antwort auf diese Frage ja lautet, sollten Sie versuchen, ein `Windows`Ziel zu erreichen. Dieses Bild ist viel größer als die anderen Basisbilder, aber es enthält viele der wichtigsten Windows-Bibliotheken (wie die GDI-Bibliothek).
- **Sind Sie ein Windows-Insider?** Wenn ja, sollten Sie die Verwendung der Insider-Version der Bilder in Frage stellen. Weitere Informationen finden Sie unten unter "Basisbilder für Windows-Insider".

> [!TIP]
> Viele Windows-Benutzer möchten Anwendungen containerisieren, die eine Abhängigkeit von .net aufweisen. Zusätzlich zu den hier beschriebenen vier Basisbildern veröffentlicht Microsoft mehrere Windows-Container Bilder, die mit beliebten Microsoft-Frameworks vorkonfiguriert sind, beispielsweise ein [.NET Framework](https://hub.docker.com/_/microsoft-dotnet-framework) -Abbild und das [ASP .net](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/) -Abbild.

### <a name="base-images-for-windows-insiders"></a>Basisbilder für Windows-Insider

Microsoft bietet "Insider"-Versionen der einzelnen Containerbasis Bilder. Diese Insider-Container Bilder tragen die neueste und beste Funktionsentwicklung in unseren Container Bildern. Wenn Sie einen Host ausführen, bei dem es sich um eine Insider Version von Windows (entweder Windows Insider oder Windows Server Insider) handelt, ist es vorzuziehen, diese Bilder zu verwenden. Die Insider-Bilder sind im docker-Hub verfügbar:

- [mcr.microsoft.com/windows/servercore/insider](https://hub.docker.com/_/microsoft-windows-servercore-insider)
- [mcr.microsoft.com/windows/nanoserver/insider](https://hub.docker.com/_/microsoft-windows-nanoserver-insider)
- [mcr.microsoft.com/windows/iotcore/insider](https://hub.docker.com/_/microsoft-windows-iotcore-insider)
- [mcr.microsoft.com/windows/insider](https://hub.docker.com/_/microsoft-windows-insider)

Lesen Sie [Verwenden von Containern mit dem Windows-Insider-Programm](../deploy-containers/insider-overview.md) , um weitere Informationen zu erhalten.

### <a name="windows-server-core-vs-nanoserver"></a>Windows Server Core vs. Server

`Windows Server Core` und `Nanoserver` sind die am häufigsten verwendeten Basisbilder für das Ziel. Der Hauptunterschied zwischen diesen Bildern besteht darin, dass der Server eine wesentlich kleinere API-Oberfläche aufweist. PowerShell, WMI und der Windows-Wartungsstapel sind im Image des Servers nicht vorhanden.

Der Server wurde so konzipiert, dass er gerade genügend API-Oberfläche zum Ausführen von apps bereitstellt, die eine Abhängigkeit von .net Core oder anderen modernen Open-Source-Frameworks aufweisen. Als Kompromiss zur kleineren API-Oberfläche hat das Image des Servers einen deutlich kleineren Speicherplatz als die restlichen Windows-Basisbilder. Aufbauend auf Nano Server können Sie nach Bedarf jederzeit Ebenen hinzufügen. Ein Beispiel hierfür finden Sie unter [.NET Core Nano Server Dockerfile](https://github.com/dotnet/dotnet-docker/blob/master/2.1/sdk/nanoserver-1803/amd64/Dockerfile).
