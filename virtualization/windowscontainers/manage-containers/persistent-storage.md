---
title: Persistenter Speicher in Containern
description: So können Windows-Container persistenter Speicher sein
keywords: Container, Volume, Speicher, Mount, Binden von Bereitstellungen
author: cwilhit
ms.openlocfilehash: 945a78d4ecb9c96da4de8f7246f84b6b444dd5b5
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209884"
---
# <a name="persistent-storage-in-containers"></a>Persistenter Speicher in Containern

<!-- Great diagram would be great! -->

Möglicherweise gibt es Fälle, in denen es wichtig ist, dass eine APP Daten in einem Container speichert, oder Sie möchten Dateien in einem Container anzeigen, die nicht in der Buildzeit des Containers enthalten sind. Beständiger Speicher kann Containern auf verschiedene Weise zugewiesen werden:

- Binden von Bereitstellungen
- Benannte Volumes

Docker hat eine gute Übersicht zum [Verwenden von Volumes](https://docs.docker.com/engine/admin/volumes/volumes/), daher sollten Sie die Informationen zuerst lesen. Der Rest der Seite konzentriert sich auf die Unterschiede zwischen Linux und Windows und gibt Beispiele für Windows.

## <a name="bind-mounts"></a>Binden von Bereitstellungen

Durch das [Binden von Bereitstellungen](https://docs.docker.com/engine/admin/volumes/bind-mounts/) kann ein Container ein Verzeichnis mit dem Host gemeinsam nutzen. Dies ist hilfreich, wenn Sie einen Speicherort für Dateien auf dem lokalen Computer benötigen, die verfügbar sind, wenn Sie einen Container neu starten, oder diese auf mehreren Containern freigeben möchten. Wenn der Container auf mehreren Computern mit Zugriff auf dieselben Dateien ausgeführt werden soll, sollte stattdessen ein benanntes Volume oder eine SMB-Mount verwendet werden.

### <a name="permissions"></a>Berechtigungen

Das Berechtigungsmodell für das Binden von Bereitstellungen variiert je nach Isolationsstufe für den Container.

Container mit **Hyper-V-Isolierung** verwenden ein einfaches schreibgeschütztes oder Lese-/Schreibzugriff-Berechtigungsmodell. Der Zugriff auf Dateien erfolgt auf dem Host mithilfe des `LocalSystem`-Kontos. Wenn der Zugriff verweigert auf den Container verweigert wird, stellen Sie sicher, dass `LocalSystem` Zugriff auf das Verzeichnis auf dem Host hat. Wenn das Schreibschutzkennzeichen verwendet wird, sind Änderungen am Volume innerhalb des Containers für das Verzeichnis auf dem Host nicht sichtbar oder nicht beständig.

Windows-Container, die die **Prozessisolierung** verwenden, unterscheiden sich geringfügig, da Sie die Prozessidentität im Container für den Zugriff auf Daten verwenden, was bedeutet, dass Datei-ACLs berücksichtigt werden. Die Identität des im Container ausgeführten Prozesses (standardmäßig "ContainerAdministrator" auf Windows Server Core und "ContainerUser" auf Nano Server-Containern) wird verwendet, um auf die Dateien und Verzeichnisse in den bereitgestellten Volumes anstelle des `LocalSystem` zuzugreifen. Es muss Ihnen der Zugriff auf die Daten gewährt werden.

Da diese Identitäten nur im Kontext des Containers vorhanden sind--nicht auf dem Host, auf dem die Dateien gespeichert sind-, sollten Sie eine bekannte Sicherheitsgruppe verwenden, `Authenticated Users` beispielsweise beim Konfigurieren der ACLs, um Zugriff auf die Container zu gewähren.

> [!WARNING]
> Binden Sie keine Bereitstellungen für sensible Verzeichnisse wie z.B. `C:\` in einem nicht vertrauenswürdigen Container. Dadurch könnten diese Dateien auf dem Host geändert werden, auf die normalerweise kein Zugriff gewährleistet ist und es könnte daraus eine Sicherheitsverletzung resultieren.

Beispielanwendung:

- `docker run -v c:\ContainerData:c:\data:RO` für den schreibgeschützten Zugriff
- `docker run -v c:\ContainerData:c:\data:RW` für den Schreibzugriff
- `docker run -v c:\ContainerData:c:\data` für den Schreibzugriff (Standard)

### <a name="symlinks"></a>Symbolische Verknüpfungen

Symbolische Verknüpfungen werden im Container aufgelöst. Beim Binden von Bereitstellungen in einem Host-Pfad zu einem Container, der eine symbolische Verknüpfung ist oder symbolische Verknüpfungen enthält, kann dieser Container nicht darauf zugreifen.

### <a name="smb-mounts"></a>SMB-Bereitstellungen

Unter Windows Server, Version 1709 und höher, ermöglicht das Feature mit dem Namen "SMB Global Mapping" die Bereitstellung einer SMB-Freigabe auf dem Host und das Übergeben von Verzeichnissen auf dieser Freigabe in einem Container. Der Container muss nicht mit einem bestimmten Server, einer bestimmten Freigabe, einem Nutzernamen oder Kennwort konfiguriert werden – diese Aufgaben werden alle auf dem Host behandelt. Der Container funktioniert so, als ob es einen lokalen Speicher hätte.

#### <a name="configuration-steps"></a>Konfigurationsschritte

1. Ordnen Sie auf dem Container Host die Remote-SMB-Freigabe Global zu:
    ```
    $creds = Get-Credential
    New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
    ```
    Dieser Befehl verwendet die Anmeldeinformationen, um sich beim SMB-Remoteserver zu authentifizieren. Ordnen Sie anschließend den Pfad für die Remotefreigabe auf G: Laufwerkbuchstabe zu (dies kann jeder verfügbare Laufwerkbuchstabe sein). Die auf diesem Containerhost erstellten Container können jetzt ihre Datenvolumes auf einen Pfad auf dem Laufwerk G: zuordnen.

    > [!NOTE]
    > Wenn Sie die globale SMB-Zuordnung für Container verwenden, können alle Benutzer auf dem Container Host auf die Remotefreigabe zugreifen. Jede auf dem Containerhost ausgeführte Anwendung hat außerdem Zugriff auf die zugeordnete Remotefreigabe.

2. Erstellen Sie Container mit Datenvolumes, die global bereitgestellten SMB-Freigaben zugeordnet sind. Führen Sie den Docker aus. Nennen Sie die Demo -v g:\ContainerData:G:\AppData1 microsoft/windowsservercore:1709 cmd.exe

    Innerhalb des Containers wird G:\AppData1 dem Verzeichnis der Remotefreigabe "ContainerData" zugeordnet. Alle Daten, die auf global zugeordneter Remotefreigabe gespeichert sind, sind für Anwendungen innerhalb des Containers verfügbar. Mehrere Container können mit dem gleichen Befehl Lese-/Schreibzugriff auf diese gemeinsam genutzten Daten erhalten.

Diese Unterstützung für die globale Zuordnung von SMB ist eine Feature für SMB-Clients, mit auf allen kompatiblen SMB-Servern funktioniert, einschließlich:

- Skalierten Dateiservern auf direkten Speicherplätzen (S2D) oder einem herkömmlichen SAN
- Azure Files (SMB-Freigabe)
- Herkömmliche Dateiserver
- Drittanbieter-Implementierung eines SMB-Protokolls (z.B.: NAS-Geräte)

> [!NOTE]
> Die globale Zuordnung in SMB unterstützt keine DFS-, DFSN-, DFSR-Freigabe unter Windows Server Version 1709.

## <a name="named-volumes"></a>Benannte Volumes

Mit benannten Volumes können Sie anhand des Namens ein Volume erstellen, dieses einem Container zuweisen und es später mit dem gleichen Namen wiederverwenden. Sie müssen nicht den tatsächlichen Pfad verfolgen, in dem es erstellt wurde, nur den Namen. Das Docker-Modul unter Windows verfügt über ein integriertes benannte Volume-Plug-In, das Volumes auf dem lokalen Computer erstellen kann. Ein zusätzliches Plug-In ist erforderlich, wenn Sie benannte Volumes auf mehreren Computern verwenden möchten.

Beispielschritte:

1. `docker volume create unwound` ‑ Erstellen Sie ein Volume mit dem Namen "unwound"
2. `docker run -v unwound:c:\data microsoft/windowsservercore` - Starten Sie einen Container, dessen Volume c:\data zugeordnet ist
3. Schreiben Sie einige Dateien auf c:\data im Container, und beenden Sie anschließend den Container
4. `docker run -v unwound:c:\data microsoft/windowsservercore` ‑ Starten Sie einen neuen Container
5. Führen Sie `dir c:\data` im neuen Container aus – die Dateien sind auch weiterhin vorhanden

> [!NOTE]
> Windows Server wandelt Ziel pfadnamee (den Pfad innerhalb des Containers) in Kleinbuchstaben um. i. e. `-v unwound:c:\MyData`oder `-v unwound:/app/MyData` in Linux-Containern führt dazu, dass ein Verzeichnis innerhalb des Containers `c:\mydata`von `/app/mydata` oder in Linux-Containern zugeordnet (und erstellt wird, wenn nicht vorhanden).
