---
title: Versionskompatibilität von Windows-Containern
description: Hier erfahren Sie, wie Windows Container versionsübergreifend erstellen und ausführen kann.
keywords: Metadaten, Container, Version
author: taylorb-microsoft
ms.openlocfilehash: 76549bbfbaf374acb79f1be4280949aecf4e87f0
ms.sourcegitcommit: c48dcfe43f73b96e0ebd661164b6dd164c775bfa
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 05/06/2019
ms.locfileid: "9610280"
---
# <a name="windows-container-version-compatibility"></a>Versionskompatibilität von Windows-container

Windows Server 2016 und Windows 10 Anniversary Update (beide Version 14393) waren die ersten Windows-Versionen, die erstellen und Ausführen von Windows Server-Container konnten. Container, die mit diesen Versionen erstellt wurden, können zwar mit neueren Versionen wie Windows Server Version 1709 ausgeführt werden, es gibt dabei aber ein paar Dinge zu beachten.

Da wir die Windows-Containerfeatures stetig verbessern, mussten wir einige Änderungen vornehmen, die sich auf die Kompatibilität auswirken können. Ältere Container unverändert auf neueren Hosts mit [Hyper-V-Isolation](../manage-containers/hyperv-container.md)ausgeführt werden, und die gleiche (ältere) Kernel-Version verwenden. Jedoch, wenn Sie einen Container basierend auf einem neueren Windows-Build ausführen möchten, können sie nur auf dem neueren Host-Build ausführen.

|Betriebssystemversion des Containers|Betriebssystemversion des Hosts|Kompatibilität|
|---|---|---|
|Windows Server 2016<br>Builds: 14393.* |Windows Server 2016<br>Builds: 14393.* |Unterstützt `process` oder `hyperv` Isolation|
|Windows Server 2016<br>Builds: 14393.* |Windows Server, Version 1709<br>Builds: 16299.* |Unterstützt nur `hyperv` Isolation|
|Windows Server 2016<br>Builds: 14393.* |Windows 10 Fall Creators Update<br>Builds: 16299.* |Unterstützt nur `hyperv` Isolation|
|Windows Server 2016<br>Builds: 14393.* |Windows Server Version 1803<br>17134.*-Builds |Unterstützt nur `hyperv` Isolation|
|Windows Server 2016<br>Builds: 14393.* |Windows10, Version1803<br>17134.*-Builds |Unterstützt nur `hyperv` Isolation|
|Windows Server 2016<br>Builds: 14393.* |Windows Server 2019<br>17763.*-Builds |Unterstützt nur `hyperv` Isolation|
|Windows Server 2016<br>Builds: 14393.* |Windows10, Version1809<br>17763.*-Builds |Unterstützt nur `hyperv` Isolation|
|Windows Server, Version 1709<br>Builds: 16299.* |Windows Server 2016<br>Builds: 14393.* |Nicht unterstützt|
|Windows Server, Version 1709<br>Builds: 16299.* |Windows Server, Version 1709<br>Builds: 16299.* |Unterstützt `process` oder `hyperv` Isolation|
|Windows Server, Version 1709<br>Builds: 16299.* |Windows 10 Fall Creators Update<br>Builds: 16299.* |Unterstützt nur `hyperv` Isolation|
|Windows Server, Version 1709<br>Builds: 16299.* |Windows Server, Version 1803<br>17134.*-Builds |Unterstützt nur `hyperv` Isolation|
|Windows Server, Version 1709<br>Builds: 16299.* |Windows10, Version1803<br>17134.*-Builds |Unterstützt nur `hyperv` Isolation|
|Windows Server, Version 1709<br>Builds: 16299.* |Windows Server 2019<br>17763.*-Builds |Unterstützt nur `hyperv` Isolation|
|Windows Server, Version 1709<br>Builds: 16299.* |Windows10, Version1809<br>17763.*-Builds |Unterstützt nur `hyperv` Isolation|
|Windows Server, Version 1803<br>17134.*-Builds |Windows Server 2016<br>Builds: 14393.* |Nicht unterstützt|
|Windows Server, Version 1803<br>17134.*-Builds |Windows Server, Version 1709<br>Builds: 16299.* |Nicht unterstützt.|
|Windows Server, Version 1803<br>17134.*-Builds |Windows 10 Fall Creators Update<br>Builds: 16299.* |Nicht unterstützt.|
|Windows Server, Version 1803<br>17134.*-Builds |Windows Server, Version 1803<br>17134.*-Builds |Unterstützt `process` oder `hyperv` Isolation|
|Windows Server, Version 1803<br>17134.*-Builds |Windows10, Version1803<br>17134.*-Builds |Unterstützt nur `hyperv` Isolation|
|Windows Server, Version 1803<br>17134.*-Builds |Windows Server 2019<br>17763.*-Builds |Unterstützt nur `hyperv` Isolation|
|Windows Server, Version 1803<br>17134.*-Builds |Windows10, Version1809<br>17763.*-Builds |Unterstützt nur `hyperv` Isolation|
|Windows Server 2019<br>17763.*-Builds |Windows Server 2016<br>Builds: 14393.* |Nicht unterstützt|
|Windows Server 2019<br>17763.*-Builds |Windows Server, Version 1709<br>Builds: 16299.* |Nicht unterstützt.
|Windows Server 2019<br>17763.*-Builds |Windows 10 Fall Creators Update<br>Builds: 16299.* |Nicht unterstützt.|
|Windows Server 2019<br>17763.*-Builds |Windows Server, Version 1803<br>17134.*-Builds |Nicht unterstützt|
|Windows Server 2019<br>17763.*-Builds |Windows10, Version1803<br>17134.*-Builds |Nicht unterstützt|
|Windows Server 2019<br>17763.*-Builds |Windows Server 2019<br>17763.*-Builds |Unterstützt `process` oder `hyperv` Isolation|
|Windows Server 2019<br>17763.*-Builds |Windows10, Version1809<br>17763.*-Builds |Unterstützt nur `hyperv` Isolation|

## <a name="matching-container-host-version-with-container-image-versions"></a>Übereinstimmende Version der Container-Host mit Container-Image-Versionen

### <a name="windows-server-containers"></a>Windows Server-Container

Da Windows Server-Container und der zugrunde liegende Host einen einzelnen Kernel Teilen, muss der Container-Basis-Image des Hosts übereinstimmen. Wenn die Versionen unterscheiden, wird der Container startet, aber vollständige funktional ist nicht gewährleistet. Das Windows-Betriebssystem hat vier versionierungsgrade: Hauptversion, Nebenversion, Build und Revision. Version (10.0.14393.103) müsste z. B. eine Hauptversion von 10, eine Nebenversion 0, eine Build-Nummer des 14393 und eine Revisionsnummer 103. Die Buildnummer wird nur geändert, wenn neue Versionen des Betriebssystems veröffentlicht werden, z. B. Version 1709, 1803, den Fall Creators Update, und so weiter. Die Revisionsnummer wird aktualisiert, wenn Windows-Updates angewendet werden.

#### <a name="build-number-new-release-of-windows"></a>Buildnummer (neue Version von Windows)

Windows Server-Container werden blockiert, starten, wenn die Buildnummer zwischen dem containerhost und dem containerimage unterschiedlich. Beispielsweise, wenn der containerhost ist Version 10.0.14393. * (Windows Server 2016) und Container-Image ist Version 10.0.16299. * (Windows Server Version 1709), der Container startet nicht.  

#### <a name="revision-number-patching"></a>Revisionsnummer (Patches)

Windows Server-Container werden nicht blockiert, starten, wenn die Revisionsnummer der Container-Host und dem containerimage unterschiedlich sind. Z. B. wenn der containerhost Version 10.0.14393.1914 (Windows Server 2016 mit KB4051033 ist) und das Container-Image Version 10.0.14393.1944 (Windows Server 2016 mit KB4053579 ist), startet das Bild weiterhin, obwohl die Revision dann Zahlen unterscheiden.

Für Windows Server 2016-basierte Hosts oder Bilder muss das Container-Image Revision den Host in einer unterstützten Konfiguration werden übereinstimmen. Allerdings Hosts oder Bilder, die mithilfe von Windows Server Version 1709 und höher, diese Regel gilt nicht, und der Host und Container-Image nicht Revisionsnummern muss. Es wird empfohlen, dass Sie Ihre Systeme mit den neuesten Patches und Updates aktuell halten.

#### <a name="practical-application"></a>Praktische Anwendung

Beispiel 1: Der Container-Host Windows Server 2016 mit KB4041691 ausgeführt wird. Alle Windows Server-Container, die auf diesem Host bereitgestellten muss auf den Container-Basisimages Version 10.0.14393.1770 basieren. Wenn Sie dem Hostcontainer KB4053579 anwenden, müssen Sie auch die Bilder, um sicherzustellen, dass der Hostcontainer unterstützt diese aktualisieren.

Beispiel 2: Der Container-Host Windows Server Version 1709 mit KB4043961 ausgeführt wird. Alle Windows Server-Container, die auf diesem Host bereitgestellten muss auf einem Windows Server Version 1709 (10.0.16299) containerbasis-Image basieren, jedoch nicht auf den Host KB übereinstimmen. Wenn KB4054517 auf dem Host angewendet wird, die containerimages werden weiterhin unterstützt, aber es wird empfohlen, dass Sie sie um alle potenzielle Sicherheitsprobleme aktualisieren.

#### <a name="querying-version"></a>Abfrageversion

Methode 1: In Version eingeführte 1709, Cmd-Eingabeaufforderung und der **Ver** -Befehl geben jetzt die Revisionsdetails zurück.

```batch
Microsoft Windows [Version 10.0.16299.125]
(c) 2017 Microsoft Corporation. All rights reserved.

C:\>ver

Microsoft Windows [Version 10.0.16299.125]
```

Methode 2: Abfragen den folgenden Registrierungsschlüssel: HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion

Beispiel:

```batch
C:\>reg query "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion" /v BuildLabEx
```

```batch
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Users\Administrator> (Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\').BuildLabEx
14393.321.amd64fre.rs1_release_inmarket.161004-2338
```

Überprüfen Sie die Tags auf Docker Hub oder die Image-Hash-Tabelle in der Beschreibung des Images bereitgestellt, um zu überprüfen, welche Version Ihr Basisimage verwendet. Die [Windows 10-Updateverlauf](https://support.microsoft.com/en-us/help/12387/windows-10-update-history) Seite zeigt jeden Build und die Revision veröffentlicht wurde.

### <a name="hyper-v-isolation-for-containers"></a>Hyper-V-Isolation für Container

Sie können Windows-Container mit oder ohne Hyper-V-Isolierung ausführen. Hyper-V-Isolation erstellt mithilfe eines optimierten VMs eine Sicherheitsbegrenzung um den Container herum. Im Gegensatz zu Windows standard-Containern, die Container und der Host den Kernel Teilen, verwendet jeder isolierte Hyper-V-Container eine eigene Instanz des Windows-Kernels. Dies bedeutet, dass Sie können verschiedene Betriebssystemversionen im Container-Host und Image (Weitere Informationen finden Sie unter den folgenden Kompatibilitätsmatrix).  

Um einen Container mit Hyper-V-Isolierung auszuführen, fügen Sie einfach das Tag `--isolation=hyperv` zu Ihrem Docker-Ausführungsbefehl hinzu.

## <a name="errors-from-mismatched-versions"></a>Fehler bei nicht übereinstimmenden Versionen

Wenn Sie versuchen, eine nicht unterstützte Kombination auszuführen, erhalten Sie die folgende Fehlermeldung angezeigt:

```dockerfile
docker: Error response from daemon: container b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\b81ed896222eb87906ccab1c3dd2fc49324eafa798438f7979b87b210906f839","Layers":[{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"b81ed896222e","MappedDirectories":[],"HvPartition":false,"EndpointList":["002a0d9e-13b7-42c0-89b2-c1e80d9af243"],"Servicing":false,"AllowUnqualifiedDNSQuery":true}.
```

Es gibt drei Möglichkeiten, die Sie diesen Fehler beheben können:

- Erstellen Sie den Container basierend auf die richtige Version von neu `microsoft/nanoserver` oder `microsoft/windowsservercore`
- Wenn der Host neuer ist, führen Sie **Docker ausführen--Isolation = Hyper-v...**
- Führen Sie den Container auf einem anderen Host mit der gleichen Windows-version

## <a name="choose-which-container-os-version-to-use"></a>Auswählen der Container-BS-Version verwenden

>[!NOTE]
>Die Tags "latest" wird zusammen mit Windows Server 2016, die aktuelle [Long-term Servicing Channel-Produkt](https://docs.microsoft.com/en-us/windows-server/get-started/semi-annual-channel-overview)aktualisiert werden. Die folgenden Anweisungen gelten für Container-Images, die mit die Windows Server Version 1709 übereinstimmen.

Sie müssen wissen, welche Version Sie für den Container verwenden müssen. Beispielsweise, wenn Sie Windows Server Version 1709 verwenden und den neuesten Patches nutzen möchten, verwenden Sie das Tag `1709` Wenn angeben, welche Version der Basisbetriebssystem-containerimages Sie möchten, wie folgt:

``` dockerfile
FROM microsoft/windowsservercore:1709
...
```

Wenn Sie ein bestimmtes Patch von Windows Server Version 1709 möchten, können Sie die KB-Nummer im Tag angeben. Beispielsweise um eine Nano Server Container-basisbetriebssystemimage von Windows Server Version 1709 mit der angewandtem kb4043961 zu erhalten, Sie würden angeben es wie folgt angeben:

``` dockerfile
FROM microsoft/nanoserver:1709_KB4043961
...
```

Wenn Sie die Nano Server Container-basisbetriebssystemimage von Windows Server 2016 benötigen, können Sie weiterhin die neueste Version dieser Basisbetriebssystem-containerimages unter Verwendung des Tags "latest" abrufen:

``` dockerfile
FROM microsoft/nanoserver
...
```

Sie können die genaue Patches, die Sie benötigen auch mit dem Schema angeben, die wir zuvor durch Angabe der Betriebssystemversion im Tag verwendet haben:

``` dockerfile
FROM microsoft/nanoserver:10.0.14393.1770
...
```

## <a name="matching-versions-using-docker-swarm"></a>Versionsabgleich mit Docker-Schwarm

Docker-Schwarm keinen derzeit integrierte Möglichkeit, die Version von Windows übereinstimmen, die einen Container mit einem Host mit der gleichen Version verwendet. Wenn Sie den Dienst, um die Verwendung eines neueren Containers aktualisieren, wird er erfolgreich ausgeführt.

Wenn Sie mehrere Versionen von Windows für einen längeren Zeitraum ausführen müssen, es gibt zwei Ansätze, die Sie ausführen können: entweder Konfigurieren der Windows-Hosts, um immer Hyper-V-Isolierung verwenden, oder verwenden die Beschriftung Einschränkungen.

### <a name="finding-a-service-that-wont-start"></a>Suchen eines Diensts, der nicht gestartet wird

Wenn ein Dienst nicht startet, sehen Sie, dass die `MODE` ist `replicated` jedoch `REPLICAS` bei 0 bleibt. Um festzustellen, ob die Version des Betriebssystems das Problem ist, führen Sie die folgenden Befehle ein:

Führen Sie **Docker-Dienst ls** , um den Dienstnamen zu suchen:

```dockerfile
ID                  NAME                MODE                REPLICAS            IMAGE                                             PORTS
xh6mwbdq2uil        angry_liskov        replicated          0/1                 microsoft/iis:windowsservercore-10.0.14393.1715
```

Führen Sie **Docker-Dienst Ps (Dienstname)** zum Abrufen des Status und der neuesten Versuche:

```dockerfile
C:\Program Files\Docker>docker service ps angry_liskov
ID                  NAME                 IMAGE                                             NODE                DESIRED STATE       CURRENT STATE               ERROR                              PORTS
klkbhn742lv0        angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Ready               Ready 3 seconds ago
y5blbdum70zo         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 24 seconds ago       "starting container failed: co…"
yjq6zwzqj8kt         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed 31 seconds ago       "starting container failed: co…"

ytnnv80p03xx         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
xeqkxbsao57w         \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715   WIN-BSTMQDRQC2E     Shutdown            Failed about a minute ago   "starting container failed: co…"
```

Wenn Sie sehen `starting container failed: ...`, sehen Sie den vollständigen Fehler mit **Docker-Dienst Ps – Nein-kürzen (Containernamen)**:

```dockerfile
C:\Program Files\Docker>docker service ps --no-trunc angry_liskov
ID                          NAME                 IMAGE                                                                                                                     NODE                DESIRED STATE       CURRENT STATE                     ERROR                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          PORTS
dwsd6sjlwsgic5vrglhtxu178   angry_liskov.1       microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Running             Starting less than a second ago                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
y5blbdum70zoh1f6uhx5nxsfv    \_ angry_liskov.1   microsoft/iis:windowsservercore-10.0.14393.1715@sha256:868bca7e89e1743792e15f78edb5a73070ef44eae6807dc3f05f9b94c23943d5   WIN-BSTMQDRQC2E     Shutdown            Failed 39 seconds ago             "starting container failed: container e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0 encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Owner":"docker","VolumePath":"\\\\?\\Volume{2443d38a-1379-4bcf-a4b7-fc6ad4cd7b65}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\e7b5d3adba7e510569c18d8e55f7c689d7cb92be40a516c91b363e27f84604d0","Layers":[{"ID":"bcf2630f-ea95-529b-b33c-e5cdab0afdb4","Path":"C:\\ProgramData\\docker\\windowsfilter\\200235127f92416724ae1d53ed3fdc86d78767132d019bdda1e1192ee4cf3ae4"},{"ID":"e3ea10a8-4c2f-5b93-b2aa-720982f116f6","Path":"C:\\ProgramData\\docker\\windowsfilter\\0ccc9fa71a9f4c5f6f3bc8134fe3533e454e09f453de662cf99ab5d2106abbdc"},{"ID":"cff5391f-e481-593c-aff7-12e080c653ab","Path":"C:\\ProgramData\\docker\\windowsfilter\\a49576b24cd6ec4a26202871c36c0a2083d507394a3072186133131a72601a31"},{"ID":"499cb51e-b891-549a-b1f4-8a25a4665fbd","Path":"C:\\ProgramData\\docker\\windowsfilter\\fdf2f52c4323c62f7ff9b031c0bc3af42cf5fba91098d51089d039fb3e834c08"},{"ID":"1532b584-8431-5b5a-8735-5e1b4fe9c2a9","Path":"C:\\ProgramData\\docker\\windowsfilter\\b2b88bc2a47abcc682e422507abbba9c9b6d826d34e67b9e4e3144cc125a1f80"},{"ID":"a64b8da5-cd6e-5540-bc73-d81acae6da54","Path":"C:\\ProgramData\\docker\\windowsfilter\\5caaedbced1f546bccd01c9d31ea6eea4d30701ebba7b95ee8faa8c098a6845a"}],"HostName":"e7b5d3adba7e","HvPartition":false,"EndpointList":["298bb656-8800-4948-a41c-1b0500f3d94c"],"AllowUnqualifiedDNSQuery":true}"
```

Dies ist der gleiche Fehler als `CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101)`.

### <a name="fix---update-the-service-to-use-a-matching-version"></a>Problembehebung: Aktualisieren des Diensts für die Verwendung einer übereinstimmenden Version

Bei Docker-Schwarm gibt es zwei Dinge zu berücksichtigen. In der Fall, in denen Sie eine Compose-Datei haben, die einen Dienst verfügt, der ein Bild verwendet, die Sie erstellt haben, sollten Sie den Verweis entsprechend aktualisieren. Beispiel:

``` yaml
version: '3'

services:
  YourServiceName:
    image: microsoft/windowsservercore:1709
...
```

Die weitere Überlegung ist, sofern das Bild auf Sie verweisen, die Sie selbst erstellt haben (z. B. Contoso/Myimage):

```yaml
version: '3'

services:
  YourServiceName:
    image: contoso/myimage
...
```

In diesem Fall sollten Sie die im [Fehler bei nicht übereinstimmenden Versionen](#errors-from-mismatched-versions) beschriebene Methode verwenden, um diese Dockerfile anstelle der docker Compose-Zeile zu ändern.

### <a name="mitigation---use-hyper-v-isolation-with-docker-swarm"></a>Lösung: Hyper-V-Isolierung mit Docker-Schwarm

Es gibt ein Vorschlag für die Unterstützung von Hyper-V-Isolierung pro Container regelmäßig, aber der Code ist noch nicht fertig entwickelt. Unter [GitHub](https://github.com/moby/moby/issues/31616) können Sie den Fortschritt verfolgen. Bis dahin müssten die Hosts so konfiguriert werden, dass sie immer mit Hyper-V-Isolierung ausgeführt werden.

Dies erfordert eine Änderung der Docker-Dienstkonfiguration und einen anschließenden Neustart des Docker-Moduls.

1. Bearbeiten Sie `C:\ProgramData\docker\config\daemon.json`
2. Fügen Sie eine Zeile hinzu mit `"exec-opts":["isolation=hyperv"]`

    >[!NOTE]
    >Die Datei "Daemon.JSON" ist standardmäßig nicht vorhanden. Wenn Sie im Verzeichnis feststellen, dass dies der Fall ist, müssen Sie die Datei erstellen. Anschließend sollten Sie in den folgenden kopieren:

    ```JSON
    {
        "exec-opts":["isolation=hyperv"]
    }
    ```

3. Schließen Sie und speichern Sie die Datei, und starten Sie das Docker-Modul neu, indem Sie die folgenden Cmdlets in PowerShell ausführen:

    ```powershell
    Stop-Service docker
    Start-Service docker
    ```

4. Nachdem Sie den Dienst neu gestartet haben, starten Sie die Container. Sobald sie ausgeführt werden, können Sie die Isolationsstufe eines Containers überprüfen, indem der Container mit dem folgenden Cmdlet:

    ```powershell
    docker inspect --format='{{json .HostConfig.Isolation}}' $instanceNameOrId
    ```

Es wird entweder "process" oder "hyperv" zurückgegeben. Wenn Sie "daemon.json" wie oben beschrieben geändert und gespeichert haben, sollte Letzteres angezeigt werden.

### <a name="mitigation---use-labels-and-constraints"></a>Lösung: Verwenden von Labels und Einschränkungen

Hier ist so verwenden Sie Labels und Einschränkungen Versionen entsprechen:

1. Hinzufügen von Labels zu jedem Knoten.

    Fügen Sie auf jedem Knoten zwei Labels hinzu: `OS` und `OsVersion`. Es wird hier davon ausgegangen, dass sie lokal ausgeführt werden, es kann aber auch so geändert werden, dass sie auf einem Remote-Host ausgeführt werden.

    ```powershell
    docker node update --label-add OS="windows" $ENV:COMPUTERNAME
    docker node update --label-add OsVersion="$((Get-ComputerInfo).OsVersion)" $ENV:COMPUTERNAME
    ```

    Anschließend können Sie überprüfen, diese mithilfe des Befehls **Docker-Knoten zu überprüfen** , die die neu hinzugefügten Beschriftungen angezeigt werden soll:

    ```yaml
           "Spec": {
                "Labels": {
                   "OS": "windows",
                   "OsVersion": "10.0.16296"
               },
                "Role": "manager",
                "Availability": "active"
            }
    ```

2. Hinzufügen einer diensteinschränkung.

    Nun, da Sie die einzelnen Knoten gekennzeichnet haben, können Sie Einschränkungen aktualisieren, die Platzierung der Dienste bestimmen. Ersetzen Sie im folgenden Beispiel "Contoso_service" durch den Namen des tatsächlichen Diensts:

    ```powershell
    docker service update \
        --constraint-add "node.labels.OS == windows" \
        --constraint-add "node.labels.OsVersion == $((Get-ComputerInfo).OsVersion)" \
        contoso_service
    ```

    Dies erzwingt und schränkt den Ort ein, an dem ein Knoten ausgeführt werden kann.

Weitere Informationen zum Verwenden von diensteinschränkungen sehen Sie sich den [Dienst Verweis erstellen](https://docs.docker.com/engine/reference/commandline/service_create/#specify-service-constraints-constraint).

## <a name="matching-versions-using-kubernetes"></a>Versionsabgleich mit Kubernetes

Das gleiche Problem in [übereinstimmende Versionen mit Docker-Schwarm](#matching-versions-using-docker-swarm) beschriebenen kann passieren, wenn Pods in Kubernetes geplant werden. Dieses Problem kann vermieden werden, mit ähnlicher Strategien vermeiden:

- Erstellen Sie den Container basierend auf der gleichen Betriebssystemversion bei Entwicklung und Produktion neu. Informationen hierzu finden Sie unter [auswählen, welche Container Betriebssystemversion verwenden](#choose-which-container-os-version-to-use).
- Verwenden von knotenlabels und Knotenselektoren, stellen Sie sicher, dass die Pods auf kompatiblen Knoten geplant werden, wenn Windows Server 2016 und Windows Server Version 1709 Knoten im selben Cluster sind
- Verwenden von separaten Clustern basierend auf der Betriebssystemversion

### <a name="finding-pods-failed-on-os-mismatch"></a>Suchen der Pods, die wegen Betriebssysteminkonsistenz fehlgeschlagen sind

In diesem Fall enthalten eine Bereitstellung einen Pod, der auf einem Knoten mit einer nicht übereinstimmenden Betriebssystemversion und ohne aktivierter Hyper-V-Isolierung geplant wurde.

Der gleiche Fehler wird in den Ereignissen angezeigt, die mit `kubectl describe pod <podname>` abgerufen werden. Trotz mehrerer Versuche der Pod-Status vermutlich wird `CrashLoopBackOff`.

```
$ kubectl -n plang describe pod fabrikamfiber.web-789699744-rqv6p

Name:           fabrikamfiber.web-789699744-rqv6p
Namespace:      plang
Node:           38519acs9011/10.240.0.6
Start Time:     Mon, 09 Oct 2017 19:40:30 +0000
Labels:         io.kompose.service=fabrikamfiber.web
                pod-template-hash=789699744
Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-789699744","uid":"b5062a08-ad29-11e7-b16e-000d3a...
Status:         Running
IP:             10.244.3.169
Created By:     ReplicaSet/fabrikamfiber.web-789699744
Controlled By:  ReplicaSet/fabrikamfiber.web-789699744
Containers:
  fabrikamfiberweb:
    Container ID:       docker://eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a
    Image:              patricklang/fabrikamfiber.web:latest
    Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
    Port:               80/TCP
    State:              Waiting
      Reason:           CrashLoopBackOff
    Last State:         Terminated
      Reason:           ContainerCannotRun
      Message:          container eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {"SystemType":"Container","Name":"eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Owner":"docker","IsDummy":false,"VolumePath":"\\\\?\\Volume{037b6606-bc9c-461f-ae02-829c28410798}","IgnoreFlushesDuringBoot":true,"LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\eab0151378308315ed6c31adf4ad9e31e6d65fd300e56e742757004a969a803a","Layers":[{"ID":"f8bc427f-7aa3-59c6-b271-7331713e9451","Path":"C:\\ProgramData\\docker\\windowsfilter\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881"},{"ID":"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47","Path":"C:\\ProgramData\\docker\\windowsfilter\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5"},{"ID":"4f624ca7-2c6d-5c42-b73f-be6e6baf2530","Path":"C:\\ProgramData\\docker\\windowsfilter\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67"},{"ID":"88e360ff-32af-521d-9a3f-3760c12b35e2","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e"},{"ID":"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a","Path":"C:\\ProgramData\\docker\\windowsfilter\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461"},{"ID":"c2b3d728-4879-5343-a92a-b735752a4724","Path":"C:\\ProgramData\\docker\\windowsfilter\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3"},{"ID":"2973e760-dc59-5800-a3de-ab9d93be81e5","Path":"C:\\ProgramData\\docker\\windowsfilter\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75"},{"ID":"454a7d36-038c-5364-8a25-fa84091869d6","Path":"C:\\ProgramData\\docker\\windowsfilter\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0"},{"ID":"9b748c8c-69eb-55fb-a1c1-5688cac4efd8","Path":"C:\\ProgramData\\docker\\windowsfilter\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec"},{"ID":"bfde5c26-b33f-5424-9405-9d69c2fea4d0","Path":"C:\\ProgramData\\docker\\windowsfilter\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2"},{"ID":"bdabfbf5-80d1-57f1-86f3-448ce97e2d05","Path":"C:\\ProgramData\\docker\\windowsfilter\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf"},{"ID":"ad9b34f2-dcee-59ea-8962-b30704ae6331","Path":"C:\\ProgramData\\docker\\windowsfilter\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8"}],"HostName":"fabrikamfiber.web-789699744-rqv6p","MappedDirectories":[{"HostPath":"c:\\var\\lib\\kubelet\\pods\\b50f0027-ad29-11e7-b16e-000d3afd2878\\volumes\\kubernetes.io~secret\\default-token-rw9dn","ContainerPath":"c:\\var\\run\\secrets\\kubernetes.io\\serviceaccount","ReadOnly":true,"BandwidthMaximum":0,"IOPSMaximum":0}],"HvPartition":false,"EndpointList":null,"NetworkSharedContainerName":"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13","Servicing":false,"AllowUnqualifiedDNSQuery":false}
      Exit Code:        128
      Started:          Mon, 09 Oct 2017 20:27:08 +0000
      Finished:         Mon, 09 Oct 2017 20:27:08 +0000
    Ready:              False
    Restart Count:      10
    Environment:        <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
Conditions:
  Type          Status
  Initialized   True
  Ready         False
  PodScheduled  True
Volumes:
  default-token-rw9dn:
    Type:       Secret (a volume populated by a Secret)
    SecretName: default-token-rw9dn
    Optional:   false
QoS Class:      BestEffort
Node-Selectors: beta.kubernetes.io/os=windows
Tolerations:    <none>
Events:
  FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
  ---------     --------        -----   ----                    -------------                           --------        ------                  -------
  49m           49m             1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-789699744-rqv6p to 38519acs9011
  49m           49m             1       kubelet, 38519acs9011                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
  29m           29m             1       kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Failed to pull image "patricklang/fabrikamfiber.web:latest": rpc error: code = 2 desc = Error response from daemon: {"message":"Get https://registry-1.docker.io/v2/: dial tcp: lookup registry-1.docker.io: no such host"}
  49m           3m              12      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
  33m           3m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
  33m           2m              11      kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         Failed                  Error: failed to start container "fabrikamfiberweb": Error response from daemon: {"message":"container fabrikamfiberweb encountered an error during CreateContainer: failure in a Windows system call: The operating system of the container does not match the operating system of the host. (0xc0370101) extra info: {\"SystemType\":\"Container\",\"Name\":\"fabrikamfiberweb\",\"Owner\":\"docker\",\"IsDummy\":false,\"VolumePath\":\"\\\\\\\\?\\\\Volume{037b6606-bc9c-461f-ae02-829c28410798}\",\"IgnoreFlushesDuringBoot\":true,\"LayerFolderPath\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\fabrikamfiberweb\",\"Layers\":[{\"ID\":\"f8bc427f-7aa3-59c6-b271-7331713e9451\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\e206d2514a6265a76645b9d6b3dc6a78777c34dbf5da9fa2d564651645685881\"},{\"ID\":\"a6f35e41-a86c-5e4d-a19a-a6c2464bfb47\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\0f21f1e28ef13030bbf0d87cbc97d1bc75f82ea53c842e9a3250a2156ced12d5\"},{\"ID\":\"4f624ca7-2c6d-5c42-b73f-be6e6baf2530\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\4d9e2ad969fbd74fd58c98b5ab61e55a523087910da5200920e2b6f641d00c67\"},{\"ID\":\"88e360ff-32af-521d-9a3f-3760c12b35e2\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e16a3d53d3e9b33344a6f0d4ed34c8a46448ee7636b672b61718225b8165e6e\"},{\"ID\":\"20f1a4e0-a6f3-5db3-9bf2-01fd3e9e855a\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\7eec7f59f9adce38cc0a6755da58a3589287d920d37414b5b21b5b543d910461\"},{\"ID\":\"c2b3d728-4879-5343-a92a-b735752a4724\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8ed04b60acc0f65f541292a9e598d5f73019c8db425f8d49ea5819eab16a42f3\"},{\"ID\":\"2973e760-dc59-5800-a3de-ab9d93be81e5\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\cc71305d45f09ce377ef497f28c3a74ee027c27f20657d2c4a5f157d2457cc75\"},{\"ID\":\"454a7d36-038c-5364-8a25-fa84091869d6\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\9e8cde1ce8f5de861a5f22841f1ab9bc53d5f606d06efeacf5177f340e8d54d0\"},{\"ID\":\"9b748c8c-69eb-55fb-a1c1-5688cac4efd8\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\8e02bf5404057055a71d542780a2bb2731be4b3707c01918ba969fb4d83b98ec\"},{\"ID\":\"bfde5c26-b33f-5424-9405-9d69c2fea4d0\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\77483cedfb0964008c33d92d306734e1fab3b5e1ebb27e898f58ccfd108108f2\"},{\"ID\":\"bdabfbf5-80d1-57f1-86f3-448ce97e2d05\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\aed2ebbb31ad24b38ee8521dd17744319f83d267bf7f360bc177e27ae9a006cf\"},{\"ID\":\"ad9b34f2-dcee-59ea-8962-b30704ae6331\",\"Path\":\"C:\\\\ProgramData\\\\docker\\\\windowsfilter\\\\d44d3a675fec1070b61d6ea9bacef4ac12513caf16bd6453f043eed2792f75d8\"}],\"HostName\":\"fabrikamfiber.web-789699744-rqv6p\",\"MappedDirectories\":[{\"HostPath\":\"c:\\\\var\\\\lib\\\\kubelet\\\\pods\\\\b50f0027-ad29-11e7-b16e-000d3afd2878\\\\volumes\\\\kubernetes.io~secret\\\\default-token-rw9dn\",\"ContainerPath\":\"c:\\\\var\\\\run\\\\secrets\\\\kubernetes.io\\\\serviceaccount\",\"ReadOnly\":true,\"BandwidthMaximum\":0,\"IOPSMaximum\":0}],\"HvPartition\":false,\"EndpointList\":null,\"NetworkSharedContainerName\":\"586870f5833279678773cb700db3c175afc81d557a75867bf39b43f985133d13\",\"Servicing\":false,\"AllowUnqualifiedDNSQuery\":false}"}
  33m           11s             151     kubelet, 38519acs9011                                           Warning         FailedSync              Error syncing pod
  32m           11s             139     kubelet, 38519acs9011   spec.containers{fabrikamfiberweb}       Warning         BackOff                 Back-off restarting failed container
```

### <a name="mitigation---using-node-labels-and-nodeselector"></a>Lösung: Verwenden von knotenlabels und -Selektoren

Führen Sie die **Kubectl Abrufen von Knoten** , um eine Liste aller Knoten zu erhalten. Danach können Sie die **Kubectl beschreiben Knoten (Knotenname)** , um weitere Informationen zu erhalten ausführen.

Im folgenden Beispiel werden zwei Windows-Knoten unterschiedliche Versionen ausführen:

```
$ kubectl get node

NAME                        STATUS    AGE       VERSION
38519acs9010                Ready     21h       v1.7.7-7+e79c96c8ff2d8e
38519acs9011                Ready     4h        v1.7.7-25+bc3094f1d650a2
k8s-linuxpool1-38519084-0   Ready     21h       v1.7.7
k8s-master-38519084-0       Ready     21h       v1.7.7

$ kubectl describe node 38519acs9010

Name:                   38519acs9010
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_D2_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9010
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 01:41:02 +0000

...
  
System Info:
 Machine ID:                    38519acs9010
 System UUID:
 Boot ID:
 Kernel Version:                10.0 14393 (14393.1715.amd64fre.rs1_release_inmarket.170906-1810)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
 ...
 
$ kubectl describe node 38519acs9011
Name:                   38519acs9011
Role:
Labels:                 beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/instance-type=Standard_DS1_v2
                        beta.kubernetes.io/os=windows
                        failure-domain.beta.kubernetes.io/region=westus2
                        failure-domain.beta.kubernetes.io/zone=0
                        kubernetes.io/hostname=38519acs9011
Annotations:            node.alpha.kubernetes.io/ttl=0
                        volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:                 <none>
CreationTimestamp:      Fri, 06 Oct 2017 18:13:25 +0000
Conditions:
...

System Info:
 Machine ID:                    38519acs9011
 System UUID:
 Boot ID:
 Kernel Version:                10.0 16299 (16299.0.amd64fre.rs3_release.170922-1354)
 OS Image:
 Operating System:              windows
 Architecture:                  amd64
...

```

Wir verwenden Sie dieses Beispiel, wie Sie die Versionen entsprechen:

1. Achten Sie darauf von jeder Knotenname und `Kernel Version` aus dem Systeminformationen.

    In unserem Beispiel wird die Informationen wie folgt aussehen:

    Name         | Version
    -------------|--------------------------------------------------------
    38519acs9010 | 14393.1715.amd64fre.rs1_release_inmarket.170906-1810
    38519acs9011 | 16299.0.amd64fre.rs3_release.170922-1354

2. Fügen Sie jedem Knoten namens `beta.kubernetes.io/osbuild` ein Label hinzu. Windows Server 2016 benötigt die Haupt- und Nebenversionsnummer Versionen (in diesem Beispiel 14393.1715) ohne Hyper-V-Isolierung unterstützt werden müssen. Windows Server Version 1709 benötigt nur die Hauptversion (16299 in diesem Beispiel) entsprechen.

    In diesem Beispiel sieht der Befehl aus, um die Beschriftungen hinzufügen:

    ```
    $ kubectl label node 38519acs9010 beta.kubernetes.io/osbuild=14393.1715


    node "38519acs9010" labeled
    $ kubectl label node 38519acs9011 beta.kubernetes.io/osbuild=16299

    node "38519acs9011" labeled

    ```

3. Überprüfen Sie, dass die Beschriftungen vorhanden sind, durch **Kubectl erhalten Knoten – anzeigen Beschriftungen**ausgeführt.

    In diesem Beispiel wird die Ausgabe wie folgt aussehen:

    ```
    $ kubectl get nodes --show-labels

    NAME                        STATUS                     AGE       VERSION                    LABELS
    38519acs9010                Ready,SchedulingDisabled   3d        v1.7.7-7+e79c96c8ff2d8e    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=14393.1715,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9010
    38519acs9011                Ready                      3d        v1.7.7-25+bc3094f1d650a2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=windows,beta.kubernetes.io/osbuild=16299,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=38519acs9011
    k8s-linuxpool1-38519084-0   Ready                      3d        v1.7.7                     agentpool=linuxpool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-linuxpool1-38519084-0,kubernetes.io/role=agent
    k8s-master-38519084-0       Ready                      3d        v1.7.7                     beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_D2_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=westus2,failure-domain.beta.kubernetes.io/zone=0,kubernetes.io/hostname=k8s-master-38519084-0,kubernetes.io/role=master
    ```

4. Fügen Sie für Bereitstellungen knotenselektoren hinzu. In diesem Beispielfall fügen wir eine `nodeSelector` auf der containerspezifikation mit `beta.kubernetes.io/os` = Windows und `beta.kubernetes.io/osbuild` = 14393. * oder 16299 Basisbetriebsversion des Containers übereinstimmen.

    Hier ein vollständiges Beispiel für die Ausführung eines Containers, der für Windows Server2016 erstellt wurde:

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      annotations:
        kompose.cmd: kompose convert -f docker-compose-combined.yml
        kompose.version: 1.2.0 (99f88ef)
      creationTimestamp: null
      labels:
        io.kompose.service: fabrikamfiber.web
      name: fabrikamfiber.web
    spec:
      replicas: 1
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            io.kompose.service: fabrikamfiber.web
        spec:
          containers:
          - image: patricklang/fabrikamfiber.web:latest
            name: fabrikamfiberweb
            ports:
            - containerPort: 80
            resources: {}
          restartPolicy: Always
          nodeSelector:
            "beta.kubernetes.io/os": windows
            "beta.kubernetes.io/osbuild": "14393.1715"
    status: {}
    ```

    Der Pod kann nun mit der aktualisierten Bereitstellung starten. Die knotenselektoren werden ebenfalls angezeigt, `kubectl describe pod <podname>`, sodass Sie diesen Befehl, überprüfen sie hinzugefügt wurden ausführen können.

    Die Ausgabe in unserem Beispiel lautet wie folgt:

    ```
    $ kubectl -n plang describe po fa

    Name:           fabrikamfiber.web-1780117715-5c8vw
    Namespace:      plang
    Node:           38519acs9010/10.240.0.4
    Start Time:     Tue, 10 Oct 2017 01:43:28 +0000
    Labels:         io.kompose.service=fabrikamfiber.web
                    pod-template-hash=1780117715
    Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"plang","name":"fabrikamfiber.web-1780117715","uid":"6a07aaf3-ad5c-11e7-b16e-000d3...
    Status:         Running
    IP:             10.244.1.84
    Created By:     ReplicaSet/fabrikamfiber.web-1780117715
    Controlled By:  ReplicaSet/fabrikamfiber.web-1780117715
    Containers:
      fabrikamfiberweb:
        Container ID:       docker://c94594fb53161f3821cf050d9af7546991aaafbeab41d333d9f64291327fae13
        Image:              patricklang/fabrikamfiber.web:latest
        Image ID:           docker-pullable://patricklang/fabrikamfiber.web@sha256:562741016ce7d9a232a389449a4fd0a0a55aab178cf324144404812887250ead
        Port:               80/TCP
        State:              Running
          Started:          Tue, 10 Oct 2017 01:43:42 +0000
        Ready:              True
        Restart Count:      0
        Environment:        <none>
        Mounts:
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-rw9dn (ro)
    Conditions:
      Type          Status
      Initialized   True
      Ready         True
      PodScheduled  True
    Volumes:
      default-token-rw9dn:
        Type:       Secret (a volume populated by a Secret)
        SecretName: default-token-rw9dn
        Optional:   false
    QoS Class:      BestEffort
    Node-Selectors: beta.kubernetes.io/os=windows
                    beta.kubernetes.io/osbuild=14393.1715
    Tolerations:    <none>
    Events:
      FirstSeen     LastSeen        Count   From                    SubObjectPath                           Type            Reason                  Message
      ---------     --------        -----   ----                    -------------                           --------        ------                  -------
      5m            5m              1       default-scheduler                                               Normal          Scheduled               Successfully assigned fabrikamfiber.web-1780117715-5c8vw to 38519acs9010
      5m            5m              1       kubelet, 38519acs9010                                           Normal          SuccessfulMountVolume   MountVolume.SetUp succeeded for volume "default-token-rw9dn"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulling                 pulling image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Pulled                  Successfully pulled image "patricklang/fabrikamfiber.web:latest"
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Created                 Created container
      5m            5m              1       kubelet, 38519acs9010   spec.containers{fabrikamfiberweb}       Normal          Started                 Started container
    ```
