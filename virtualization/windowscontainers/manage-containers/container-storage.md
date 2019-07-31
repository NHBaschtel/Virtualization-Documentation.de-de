---
title: Windows Server Containerspeicher
description: So können Windows Server-Container Host- und andere Speichertypen verwenden
keywords: Container, Volume, Speicher, Mount, Binden von Bereitstellungen
author: patricklang
ms.openlocfilehash: bddfb3a3510a6af674be73349a7e422434c1e0f4
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882973"
---
# <a name="overview"></a>Übersicht

<!-- Great diagram would be great! -->


## <a name="layer-storage"></a>Schichtspeicher

Hierbei handelt es sich um die Dateien, die im Container integriert sind. Bei jedem `docker pull` und `docker run` des Containers sind diese identisch.


### <a name="where-layers-are-stored-and-how-to-change-it"></a>Wo Schichten gespeichert werden und wie Sie diese ändern

Bei einer Standardinstallation werden die Schichten unter `C:\ProgramData\docker` gespeichert und auf die Verzeichnisse "Image" und "Windowsfilter" verteilt. Sie können den Speicherort der Schichten mithilfe der `docker-root`-Konfiguration ändern, wie in der Dokumentation [Docker-Modul unter Windows](../manage-docker/configure-docker-daemon.md) erläutert.

> [!NOTE]
> Für die Schichtspeicher wird nur NTFS unterstützt. ReFS wird nicht unterstützt.

Sie sollten keine Dateien der Schichtverzeichnisse ändern – diese werden sorgfältig verwaltet mithilfe von Befehlen wie:

- [Docker Bilder](https://docs.docker.com/engine/reference/commandline/images/)
- [Docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [Docker Abruf](https://docs.docker.com/engine/reference/commandline/pull/)
- [Docker laden](https://docs.docker.com/engine/reference/commandline/load/)
- [Docker speichern](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>Unterstützte Vorgänge im Schichtspeicher

Beim Ausführen von Containern können die meisten NTFS-Vorgänge, mit Ausnahme der Transaktionen, verwendet werden. Dies beinhaltet das Festlegen von ACLs, wobei alle ACLs innerhalb des Containers geprüft werden. Wenn Prozesse mit mehreren Benutzern in einem Container ausgeführt werden sollen, können Sie Benutzer in Ihrer `Dockerfile`mit `RUN net user /create ...` erstellen, Dateizugriffssteuerungslisten festlegen und dann Vorgänge für diesen Benutzer mithilfe der [Dockerfile-USER-Direktive](https://docs.docker.com/engine/reference/builder/#user) konfigurieren.


## <a name="image-size"></a>Bildgröße
Ein gängiges Muster für Windows-Anwendungen ist das Abfragen des Speicherplatzes vor der Installation oder vor dem Erstellen neuer Dateien oder als Auslöser für das Bereinigen temporärer Dateien.  Zur Maximierung der Anwendungskompatibilität stellt das Laufwerk "C:" in einem Windows-Container eine virtuelle Größe von 20GB bereit.  Einige Benutzer möchten diese Standardeinstellung eventuell überschreiben und den Wert des freien Speicherplatzes verkleinern oder vergrößern. Dazu dient die Option "Größe" in der Konfiguration "Speicher-opt".

### <a name="examples"></a>Beispiele
Befehlszeile: `docker run --storage-opt "size=50GB" microsoft/windowsservercore:1709 cmd`

Docker-Konfigurationsdatei
```
"storage-opts": [
    "size=50GB"
  ]
```
> Beachten Sie, dass diese Methode für Docker Build funktioniert.
Weitere Informationen zum Ändern der Docker-Konfigurationsdatei finden Sie im Dokument [Konfigurieren von Docker](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file).


## <a name="persistent-volumes"></a>Persistente Volumes

Beständige Speicher können Containern auf verschiedene Weisen erteilt werden:

- Binden von Bereitstellungen
- Benannte Volumes

Docker hat eine gute Übersicht zum [Verwenden von Volumes](https://docs.docker.com/engine/admin/volumes/volumes/), daher sollten Sie die Informationen zuerst lesen. Der Rest der Seite konzentriert sich auf die Unterschiede zwischen Linux und Windows und gibt Beispiele für Windows.


### <a name="bind-mounts"></a>Binden von Bereitstellungen

Durch das [Binden von Bereitstellungen](https://docs.docker.com/engine/admin/volumes/bind-mounts/) kann ein Container ein Verzeichnis mit dem Host gemeinsam nutzen. Dies ist hilfreich, wenn Sie einen Speicherort für Dateien auf dem lokalen Computer benötigen, die verfügbar sind, wenn Sie einen Container neu starten, oder diese auf mehreren Containern freigeben möchten. Wenn der Container auf mehreren Computern mit Zugriff auf dieselben Dateien ausgeführt werden soll, sollte stattdessen ein benanntes Volume oder eine SMB-Mount verwendet werden.

#### <a name="permissions"></a>Berechtigungen

Das Berechtigungsmodell für das Binden von Bereitstellungen variiert je nach Isolationsstufe für den Container.

Verwenden Sie für Container mit **Hyper-V-Isolation**, einschließlich Linux-Container unter Windows Server Version 1709, ein einfaches schreibgeschütztes oder Lese-/ Schreibberechtigungsmodell.
Der Zugriff auf Dateien erfolgt auf dem Host mithilfe des `LocalSystem`-Kontos. Wenn der Zugriff verweigert auf den Container verweigert wird, stellen Sie sicher, dass `LocalSystem` Zugriff auf das Verzeichnis auf dem Host hat.
Wenn das Schreibschutzkennzeichen verwendet wird, sind Änderungen am Volume innerhalb des Containers für das Verzeichnis auf dem Host nicht sichtbar oder nicht beständig.

Windows Server-Container mit **Prozessisolation** sind etwas anders, da sie die Prozess-ID innerhalb des Containers verwenden, um auf Daten zuzugreifen, d.h. die Dateizugriffssteuerungslisten werden berücksichtigt.
Die Identität des im Container ausgeführten Prozesses (standardmäßig "ContainerAdministrator" auf Windows Server Core und "ContainerUser" auf Nano Server-Containern) wird verwendet, um auf die Dateien und Verzeichnisse in den bereitgestellten Volumes anstelle des `LocalSystem` zuzugreifen. Es muss Ihnen der Zugriff auf die Daten gewährt werden.
Da diese Identitäten nur im Kontext des Containers vorhanden sind und nicht auf dem Host, auf dem die Dateien gespeichert sind, sollten Sie eine allgemein bekannte Sicherheitsgruppe wie `Authenticated Users` verwenden, wenn Sie ACLs konfigurieren, um den Zugriff auf die Container zu gewährleisten.

> [!WARNING]
> Binden Sie keine Bereitstellungen für sensible Verzeichnisse wie z.B. `C:\` in einem nicht vertrauenswürdigen Container. Dadurch könnten diese Dateien auf dem Host geändert werden, auf die normalerweise kein Zugriff gewährleistet ist und es könnte daraus eine Sicherheitsverletzung resultieren.

Beispielanwendung: 

- `docker run -v c:\ContainerData:c:\data:RO` für den schreibgeschützten Zugriff
- `docker run -v c:\ContainerData:c:\data:RW` für den Schreibzugriff
- `docker run -v c:\ContainerData:c:\data` für den Schreibzugriff (Standard)

#### <a name="symlinks"></a>Symbolische Verknüpfungen

Symbolische Verknüpfungen werden im Container aufgelöst. Beim Binden von Bereitstellungen in einem Host-Pfad zu einem Container, der eine symbolische Verknüpfung ist oder symbolische Verknüpfungen enthält, kann dieser Container nicht darauf zugreifen.

#### <a name="smb-mounts"></a>SMB-Bereitstellungen

Unter Windows Server Version 1709 ermöglicht ein neues Feature namens "globale Zuordnung in SMB" eine SMB-Freigabe auf den Host und übergibt diese Verzeichnisse anschließend der Freigabe in einen Container. Der Container muss nicht mit einem bestimmten Server, einer bestimmten Freigabe, einem Nutzernamen oder Kennwort konfiguriert werden – diese Aufgaben werden alle auf dem Host behandelt. Der Container funktioniert so, als ob es einen lokalen Speicher hätte.

##### <a name="configuration-steps"></a>Konfigurationsschritte

1. Ordnen Sie auf dem Container Host die Remote-SMB-Freigabe Global zu:
    ```
    $creds = Get-Credential
    New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
    ```
    Dieser Befehl verwendet die Anmeldeinformationen, um sich beim SMB-Remoteserver zu authentifizieren. Ordnen Sie anschließend den Pfad für die Remotefreigabe auf G: Laufwerkbuchstabe zu (dies kann jeder verfügbare Laufwerkbuchstabe sein). Die auf diesem Containerhost erstellten Container können jetzt ihre Datenvolumes auf einen Pfad auf dem Laufwerk G: zuordnen.

    > Hinweis: Wenn die globale Zuordnung in SMB für Container verwendet wird, können alle Benutzer auf dem Containerhost auf die Remotefreigabe zugreifen. Jede auf dem Containerhost ausgeführte Anwendung hat außerdem Zugriff auf die zugeordnete Remotefreigabe.

2. Erstellen Sie Container mit Datenvolumes, die global bereitgestellten SMB-Freigaben zugeordnet sind. Führen Sie den Docker aus. Nennen Sie die Demo -v g:\ContainerData:G:\AppData1 microsoft/windowsservercore:1709 cmd.exe

    Innerhalb des Containers wird G:\AppData1 dem Verzeichnis der Remotefreigabe "ContainerData" zugeordnet. Alle Daten, die auf global zugeordneter Remotefreigabe gespeichert sind, sind für Anwendungen innerhalb des Containers verfügbar. Mehrere Container können mit dem gleichen Befehl Lese-/Schreibzugriff auf diese gemeinsam genutzten Daten erhalten.

Diese Unterstützung für die globale Zuordnung von SMB ist eine Feature für SMB-Clients, mit auf allen kompatiblen SMB-Servern funktioniert, einschließlich:

- Skalierten Dateiservern auf direkten Speicherplätzen (S2D) oder einem herkömmlichen SAN
- Azure Files (SMB-Freigabe)
- Herkömmliche Dateiserver
- Drittanbieter-Implementierung eines SMB-Protokolls (z.B.: NAS-Geräte)

> [!NOTE]
> Die globale Zuordnung in SMB unterstützt keine DFS-, DFSN-, DFSR-Freigabe unter Windows Server Version 1709.

### <a name="named-volumes"></a>Benannte Volumes

Mit benannten Volumes können Sie anhand des Namens ein Volume erstellen, dieses einem Container zuweisen und es später mit dem gleichen Namen wiederverwenden. Sie müssen nicht den tatsächlichen Pfad verfolgen, in dem es erstellt wurde, nur den Namen. Das Docker-Modul unter Windows verfügt über ein integriertes benannte Volume-Plug-In, das Volumes auf dem lokalen Computer erstellen kann. Ein zusätzliches Plug-In ist erforderlich, wenn Sie benannte Volumes auf mehreren Computern verwenden möchten.

Beispielschritte:

1. `docker volume create unwound` ‑ Erstellen Sie ein Volume mit dem Namen "unwound"
2. `docker run -v unwound:c:\data microsoft/windowsservercore` - Starten Sie einen Container, dessen Volume c:\data zugeordnet ist
3. Schreiben Sie einige Dateien auf c:\data im Container, und beenden Sie anschließend den Container
4. `docker run -v unwound:c:\data microsoft/windowsservercore` ‑ Starten Sie einen neuen Container
5. Führen Sie `dir c:\data` im neuen Container aus – die Dateien sind auch weiterhin vorhanden

> [!NOTE]
> Windows Server wandelt Ziel pfadnamee (den Pfad innerhalb des Containers) in Kleinbuchstaben um. i. e. `-v unwound:c:\MyData`oder `-v unwound:/app/MyData` in Linux-Containern führt dazu, dass ein Verzeichnis innerhalb des Containers `c:\mydata`von `/app/mydata` oder in Linux-Containern zugeordnet (und erstellt wird, wenn nicht vorhanden).
