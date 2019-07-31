---
title: Gruppen verwaltete Dienstkonten für Windows-Container
description: Gruppen verwaltete Dienstkonten für Windows-Container
keywords: docker, Container, Active Directory, GMSA
author: rpsqrd
ms.date: 06/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: b908a35f63b2f25da3fb19c0f96b55fe3e513350
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883173"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>Gruppen verwaltete Dienstkonten für Windows-Container

In Windows-basierten Netzwerken werden in der Regel Active Directory (AD) verwendet, um die Authentifizierung und Autorisierung zwischen Benutzern, Computern und anderen Netzwerkressourcen zu vereinfachen. Entwickler von Unternehmensanwendungen entwerfen Ihre apps oft so, dass Sie AD-integriert sind und auf Domänen verbundenen Servern ausgeführt werden, um die integrierte Windows-Authentifizierung zu nutzen, die es Benutzern und anderen Diensten erleichtert, sich automatisch und transparent bei die Anwendung mit ihren Identitäten

Obwohl Windows-Container keine Domäne sein können, können Sie weiterhin Active Directory-Domänenidentitäten verwenden, um verschiedene Authentifizierungsszenarien zu unterstützen.

Um dies zu erreichen, können Sie einen Windows-Container für die Ausführung mit einem [Group Managed Service-Konto](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA) konfigurieren, bei dem es sich um eine spezielle Art von Dienstkonto handelt, das in Windows Server 2012 eingeführt wurde, damit mehrere Computer eine Identität freigeben können, ohne dass um sein Kennwort zu kennen.

Wenn Sie einen Container mit einem gMSA ausführen, ruft der Container Host das gMSA-Kennwort von einem Active Directory-Domänencontroller ab und übergibt ihn an die Containerinstanz. Der Container verwendet die gMSA-Anmeldeinformationen, wenn sein Computerkonto (System) auf Netzwerkressourcen zugreifen muss.

In diesem Artikel wird erläutert, wie Sie mit der Verwendung von Active Directory Group Managed Service-Konten mit Windows-Containern beginnen.

## <a name="prerequisites"></a>Voraussetzungen

Wenn Sie einen Windows-Container mit einem Group Managed Service-Konto ausführen möchten, benötigen Sie Folgendes:

- Eine Active Directory-Domäne mit mindestens einem Domänencontroller, auf dem Windows Server 2012 oder höher ausgeführt wird. Für die Verwendung von gMSAs sind keine Anforderungen an die Gesamtstruktur-oder Domänenfunktionsebene vorhanden, die gMSA-Kennwörter können jedoch nur von Domänencontrollern mit Windows Server 2012 oder höher verteilt werden. Weitere Informationen finden Sie unter [Active Directory-Anforderungen für gMSAs](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req).
- Berechtigung zum Erstellen eines gMSA-Kontos. Wenn Sie ein gMSA-Konto erstellen möchten, müssen Sie ein Domänen Administrator sein oder ein Konto verwenden, dem die Berechtigung *msDS-GroupManagedServiceAccount Objekte erstellen* delegiert wurde.
- Zugriff auf das Internet, um das CredentialSpec PowerShell-Modul herunterzuladen. Wenn Sie in einer getrennten Umgebung arbeiten, können Sie [das Modul](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1) auf einem Computer mit Internetzugriff speichern und auf Ihren Entwicklungscomputer oder ihren Container Host kopieren.

## <a name="one-time-preparation-of-active-directory"></a>Einmalige Vorbereitung von Active Directory

Wenn Sie noch kein gMSA in Ihrer Domäne erstellt haben, müssen Sie den Key Distribution Service (KDS)-Stammschlüssel generieren. Der KDS ist für das Erstellen, drehen und Freigeben des gMSA-Kennworts für autorisierte Hosts verantwortlich. Wenn ein Container Host die gMSA zum Ausführen eines Containers verwenden muss, wird er mit dem KDS Kontakt aufnehmen, um das aktuelle Kennwort abzurufen.

Wenn Sie überprüfen möchten, ob der KDS-Stammschlüssel bereits erstellt wurde, führen Sie das folgende PowerShell-Cmdlet als Domänenadministrator auf einem Domänencontroller oder Domänenmitglied aus, auf dem die AD PowerShell-Tools installiert sind:

```powershell
Get-KdsRootKey
```

Wenn der Befehl eine Schlüssel-ID zurückgibt, sind Sie ganz eingestellt und können mit dem Abschnitt [Erstellen eines verwalteten Dienstkontos](#create-a-group-managed-service-account) fortfahren. Andernfalls fahren Sie fort, um den KDS-Stammschlüssel zu erstellen.

Führen Sie in einer Produktionsumgebung oder Testumgebung mit mehreren Domänencontrollern das folgende Cmdlet in PowerShell als Domänen Administrator aus, um den KDS-Stammschlüssel zu erstellen.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

Obwohl der Befehl impliziert, dass der Schlüssel sofort wirksam wird, müssen Sie 10 Stunden warten, bevor der KDS-Stammschlüssel repliziert und für die Verwendung auf allen Domänencontrollern zur Verfügung steht.

Wenn Sie nur über einen Domänencontroller in Ihrer Domäne verfügen, können Sie den Vorgang beschleunigen, indem Sie den Schlüssel so festlegen, dass er vor 10 Stunden gültig ist.

>[!IMPORTANT]
>Verwenden Sie diese Methode in einer Produktionsumgebung nicht.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>Erstellen eines Gruppen verwalteten Dienstkontos

Jeder Container, der die integrierte Windows-Authentifizierung verwendet, benötigt mindestens eine gMSA. Der primäre gMSA wird immer dann verwendet, wenn apps, die als System oder als Netzwerkdienst ausgeführt werden, auf Ressourcen im Netzwerk zugreifen. Der Name des gMSA wird unabhängig vom dem Container zugewiesenen Hostname im Netzwerk als Name des Containers bezeichnet. Container können auch mit zusätzlichen gMSAs konfiguriert werden, falls Sie einen Dienst oder eine Anwendung im Container als eine andere Identität aus dem Container Computerkonto ausführen möchten.

Wenn Sie eine gMSA erstellen, erstellen Sie auch eine freigegebene Identität, die gleichzeitig auf vielen verschiedenen Computern verwendet werden kann. Der Zugriff auf das gMSA-Kennwort ist durch eine Active Directory-Zugriffssteuerungsliste geschützt. Wir empfehlen, eine Sicherheitsgruppe für jedes gMSA-Konto zu erstellen und die entsprechenden Container Hosts zur Sicherheitsgruppe hinzuzufügen, um den Zugriff auf das Kennwort zu begrenzen.

Da Container keine Dienstprinzipalnamen (Service Principal Names, SPN) automatisch registrieren, müssen Sie schließlich mindestens einen Host-SPN für Ihr gMSA-Konto erstellen.

In der Regel wird der Host-oder http-SPN mit dem gleichen Namen wie das gMSA-Konto registriert, doch müssen Sie möglicherweise einen anderen Dienstnamen verwenden, wenn Clients auf die Containeranwendung hinter einem Lastenausgleichsmodul oder einem DNS-Namen zugreifen, der sich vom gMSA-Namen unterscheidet.

Wenn beispielsweise das gMSA-Konto den Namen "WebApp01" hat `mysite.contoso.com`, Ihre Benutzer jedoch auf die Website zugreifen, sollten Sie `http/mysite.contoso.com` einen SPN für das gMSA-Konto registrieren.

Für einige Anwendungen sind möglicherweise zusätzliche SPNs für Ihre eindeutigen Protokolle erforderlich. Zum Beispiel erfordert SQL Server den `MSSQLSvc/hostname` SPN.

In der folgenden Tabelle sind die erforderlichen Attribute zum Erstellen eines gMSA aufgelistet.

|gMSA-Eigenschaft | Erforderlicher Wert | Beispiel |
|--------------|----------------|--------|
|Name | Einen beliebigen gültigen Kontonamen. | `WebApp01` |
|DNSHostName | Der Domänenname, der an den Kontonamen angefügt wurde. | `WebApp01.contoso.com` |
|ServicePrincipalNames | Setzen Sie mindestens den Host-SPN, und fügen Sie bei Bedarf weitere Protokolle hinzu. | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | Die Sicherheitsgruppe, die Ihre Container Hosts enthält. | `WebApp01Hosts` |

Nachdem Sie sich für den Namen Ihres gMSA entschieden haben, führen Sie die folgenden Cmdlets in PowerShell aus, um die Sicherheitsgruppe und gMSA zu erstellen.

> [!TIP]
> Sie müssen ein Konto verwenden, das zur Sicherheitsgruppe " **Domänenadministratoren** " gehört, oder die Berechtigung zum Erstellen von **msDS-GroupManagedServiceAccount-Objekten** delegiert wurde, um die folgenden Befehle auszuführen.
> Das Cmdlet [New-ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) ist Teil der AD PowerShell-Tools aus den [Remote Server-Verwaltungstools](https://aka.ms/rsat).

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -GroupScope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

Wir empfehlen, separate gMSA-Konten für Ihre dev-, Test-und Produktionsumgebungen zu erstellen.

## <a name="prepare-your-container-host"></a>Vorbereiten des Container Hosts

Jeder Container Host, auf dem ein Windows-Container mit einem gMSA ausgeführt wird, muss der Domäne beigetreten sein und Zugriff auf das gMSA-Kennwort haben.

1. Fügen Sie Ihren Computer zu Ihrer Active Directory-Domäne hinzu.
2. Stellen Sie sicher, dass Ihr Host zur Sicherheitsgruppe gehört, die den Zugriff auf das gMSA-Kennwort steuert.
3. Starten Sie den Computer neu, damit er seine neue Gruppenmitgliedschaft erhält.
4. Einrichten des andockbaren [Desktops für Windows 10](https://docs.docker.com/docker-for-windows/install/) oder andocken [für Windows Server](https://docs.docker.com/install/windows/docker-ee/)
5. Empfohlen Überprüfen Sie, ob der Host das gMSA-Konto verwenden kann, indem Sie [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)ausführen. Wenn der Befehl **false**zurückgibt, lesen Sie den Abschnitt [Problembehandlung](#troubleshooting) für Diagnoseschritte.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Erstellen einer Anmelde Informations Spezifikation

Eine Anmeldeinformationen-spec-Datei ist ein JSON-Dokument, das Metadaten zu den gMSA-Konten enthält, die ein Container verwenden soll. Indem Sie die Identity-Konfiguration vom Container-Bild getrennt halten, können Sie ändern, welche gMSA der Container verwendet, indem Sie einfach die Spezifikationsdatei für die Anmeldeinformationen austauschen, keine Codeänderungen erforderlich.

Die Datei für die Anmeldeinformationen wird mithilfe des [CredentialSpec PowerShell](https://aka.ms/credspec) -Moduls auf einem Domänen verbundenen Container Host erstellt.
Nachdem Sie die Datei erstellt haben, können Sie Sie auf andere Container Hosts oder ihren Container Orchestrator kopieren.
Die Datei mit den Anmeldeinformationen enthält keine Geheimnisse, wie etwa das gMSA-Kennwort, da der Container Host die gMSA im Auftrag des Containers abruft.

Docker erwartet, dass die Anmeldeinformationsdatei unter dem **CredentialSpecs** -Verzeichnis im docker-Datenverzeichnis zu finden ist. In einer Standardinstallation finden Sie diesen Ordner unter `C:\ProgramData\Docker\CredentialSpecs`.

So erstellen Sie eine Anmelde Informations Spezifikationsdatei auf dem Container Host:

1. Installieren der Remotetools für AD PowerShell
    - Führen Sie für Windows Server die **Installations-Cmdlets-Remote Server-AD-PowerShell**aus.
    - Für Windows 10, Version 1809 oder höher, führen Sie die **Installations-WindowsCapability-Online-"Remote-Webdienste. ActiveDirectory. DS-LDS. Tools ~**~ ~ ~ 0.0.1.0" aus.
    - Informationen zu älteren Versionen von Windows 10 finden <https://aka.ms/rsat>Sie unter.
2. Führen Sie das folgende Cmdlet aus, um die neueste Version des [CredentialSpec PowerShell](https://aka.ms/credspec)-Moduls zu installieren:

    ```powershell
    Install-Module CredentialSpec
    ```

    Wenn Sie keinen Internetzugriff auf dem Container Host haben, führen `Save-Module CredentialSpec` Sie einen Computer mit Internetverbindung aus, und kopieren Sie den `C:\Program Files\WindowsPowerShell\Modules` Modulordner an einen `$env:PSModulePath` anderen Speicherort auf dem Container Host.

3. Führen Sie das folgende Cmdlet aus, um die neue Spezifikationsdatei für Anmeldeinformationen zu erstellen:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    Standardmäßig erstellt das Cmdlet eine cred-Spezifikation unter Verwendung des angegebenen gMSA-namens als Computerkonto für den Container. Die Datei wird im docker CredentialSpecs-Verzeichnis mit der gMSA-Domäne und dem Kontonamen für den Dateinamen gespeichert.

    Sie können eine Anmelde Informations Spezifikation erstellen, die zusätzliche gMSA-Konten enthält, wenn Sie einen Dienst oder einen Prozess als sekundäre gMSA im Container ausführen. Verwenden Sie dazu den `-AdditionalAccounts` Parameter:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    Eine vollständige Liste der unterstützten Parameter `Get-Help New-CredentialSpec`finden Sie unter.

4. Sie können eine Liste aller Anmeldeinformationen und deren vollständigen Pfad mit dem folgenden Cmdlet anzeigen:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configure-your-application-to-use-the-gmsa"></a>Konfigurieren der Anwendung für die Verwendung von gMSA

In der typischen Konfiguration erhält ein Container nur ein gMSA-Konto, das verwendet wird, wenn das Container Computerkonto versucht, sich bei Netzwerkressourcen zu authentifizieren. Dies bedeutet, dass Ihre APP als **Lokales System** oder als **Netzwerkdienst** ausgeführt werden muss, wenn Sie die gMSA-Identität verwenden muss.

### <a name="run-an-iis-app-pool-as-network-service"></a>Ausführen eines IIS-App-Pools als Netzwerkdienst

Wenn Sie eine IIS-Website in Ihrem Container hosten, müssen Sie lediglich die gMSA-Identität Ihres App-Pools auf **Netzwerkdienst**einstellen. Sie können dies in Ihrem Dockerfile tun, indem Sie den folgenden Befehl hinzufügen:

```dockerfile
RUN %windir%\system32\inetsrv\appcmd.exe set AppPool DefaultAppPool -processModel.identityType:NetworkService
```

Wenn Sie zuvor statische Benutzeranmeldeinformationen für Ihren IIS-App-Pool verwendet haben, sollten Sie die gMSA als Ersatz für diese Anmeldeinformationen verwenden. Sie können die gMSA zwischen dev-, Test-und Produktionsumgebungen ändern, und IIS übernimmt automatisch die aktuelle Identität, ohne das Container Bild ändern zu müssen.

### <a name="run-a-windows-service-as-network-service"></a>Ausführen eines Windows-Diensts als Netzwerkdienst

Wenn Ihre Container-App als Windows-Dienst ausgeführt wird, können Sie den Dienst so einrichten, dass er in Ihrem Dockerfile als **Netzwerkdienst** ausgeführt wird:

```dockerfile
RUN sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>Ausführen beliebiger Konsolen-Apps als Netzwerkdienst

Bei generischen Konsolen-apps, die nicht in IIS oder Service Manager gehostet werden, ist es häufig am einfachsten, den Container als **Netzwerkdienst** auszuführen, damit die APP automatisch den gMSA-Kontext erbt. Dieses Feature steht ab Version 1709 von Windows Server zur Verfügung.

Fügen Sie Ihrer Dockerfile die folgende Zeile hinzu, damit Sie standardmäßig als Netzwerkdienst ausgeführt wird:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

Sie können auch eine einmalige Verbindung mit `docker exec`einem Container als Netzwerkdienst herstellen. Dies ist besonders hilfreich, wenn Sie Verbindungsprobleme in einem ausgeführten Container beheben möchten, wenn der Container normalerweise nicht als Netzwerkdienst ausgeführt wird.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>Ausführen eines Containers mit einem gMSA

Wenn Sie einen Container mit einem gMSA ausführen möchten, geben Sie die Anmeldeinformationen `--security-opt` -Spezifikationsdatei für den Parameter der andockbaren [Ausführung](https://docs.docker.com/engine/reference/run)an:

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

>[!IMPORTANT]
>Unter Windows Server 2016-Versionen 1709 und 1803 muss der Hostname des Containers mit dem gMSA-Kurznamen übereinstimmen.

Im vorherigen Beispiel lautet der Name des gMSA SAM-Kontos "webapp01", sodass der Container-Hostname auch als "webapp01" bezeichnet wird.

Unter Windows Server 2019 und höher ist das Feld "Hostname" nicht erforderlich, aber der Container identifiziert sich nach wie vor durch den gMSA-Namen anstelle des Hostnamens, auch wenn Sie explizit einen anderen angeben.

Führen Sie das folgende Cmdlet im Container aus, um zu überprüfen, ob die gMSA ordnungsgemäß funktioniert:

```powershell
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Wenn der Status der vertrauenswürdigen DC-Verbindung und der `NERR_Success`Bestätigungsstatus der Vertrauensstellung nicht angezeigt wird, lesen Sie den Abschnitt [Problembehandlung](#troubleshooting) , um Tipps zum Debuggen des Problems zu finden.

Sie können die gMSA-Identität innerhalb des Containers überprüfen, indem Sie den folgenden Befehl ausführen und den Clientnamen überprüfen:

```powershell
PS C:\> klist get krbtgt

Current LogonId is 0:0xaa79ef8
A ticket to krbtgt has been retrieved successfully.

Cached Tickets: (2)

#0>     Client: webapp01$ @ CONTOSO.COM
        Server: krbtgt/webapp01 @ CONTOSO.COM
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
        Start Time: 3/21/2019 4:17:53 (local)
        End Time:   3/21/2019 14:17:53 (local)
        Renew Time: 3/28/2019 4:17:42 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: dc01.contoso.com

[...]
```

Wenn Sie PowerShell oder eine andere Konsolen-App als gMSA-Konto öffnen möchten, können Sie den Container für die Ausführung unter dem Netzwerkdienstkonto anstelle des normalen ContainerAdministrator-Kontos (oder ContainerUser für das Server Konto) bitten:

```powershell
# NOTE: you can only run as Network Service or SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\NETWORK SERVICE" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Wenn Sie als Netzwerkdienst ausgeführt werden, können Sie die Netzwerkauthentifizierung als gMSA testen, indem Sie versuchen, eine Verbindung mit SYSVOL auf einem Domänencontroller herzustellen:

```powershell
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrate-containers-with-gmsa"></a>Orchestrieren von Containern mit gMSA

In Produktionsumgebungen verwenden Sie häufig einen Container-Orchestrator, um Ihre apps und Dienste bereitzustellen und zu verwalten. Jeder Orchestrator verfügt über eigene Verwaltungs Paradigmen und ist für die Annahme von Anmeldeinformationen für die Windows-Container Plattform verantwortlich.

Wenn Sie Container mit gMSAs orchestrieren, stellen Sie Folgendes sicher:

> [!div class="checklist"]
> * Alle Container Hosts, die für die Ausführung von Containern mit gMSAs geplant werden können, sind Domänen verbunden.
> * Die Container Hosts können die Kennwörter für alle von Containern verwendeten gMSAs abrufen.
> * Die Anmeldeinformationsdateien werden erstellt und in den Orchestrator hochgeladen oder auf jeden Container Host kopiert, je nachdem, wie der Orchestrator Sie bevorzugt.
> * Container Netzwerke ermöglichen es den Containern, mit den Active Directory-Domänencontrollern zu kommunizieren, um gMSA-Tickets abzurufen.

### <a name="how-to-use-gmsa-with-service-fabric"></a>Verwenden von gMSA mit Service Fabric

Service Fabric unterstützt die Ausführung von Windows-Containern mit einem gMSA, wenn Sie den Speicherort der Anmeldeinformationen in Ihrem Anwendungsmanifest angeben. Sie müssen die Anmeldeinformationen-spec-Datei erstellen und im **CredentialSpecs** -Unterverzeichnis des docker-Datenverzeichnisses auf jedem Host platzieren, damit es von Service Fabric gefunden werden kann. Sie können das Cmdlet **Get-CredentialSpec** , Teil des CredentialSpec- [PowerShell](https://aka.ms/credspec)-Moduls, ausführen, um zu überprüfen, ob sich Ihre Anmeldeinformationen am richtigen Speicherort befinden.

Weitere Informationen zum Konfigurieren der Anwendung finden Sie unter [Schnellstart: Bereitstellen von Windows-Containern in Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-quickstart-containers) und [Einrichten von gMSA für Windows-Container, die auf Service Fabric ausgeführt](https://docs.microsoft.com/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) werden.

### <a name="how-to-use-gmsa-with-docker-swarm"></a>Verwenden von gMSA mit Andock barem Schwarm

Wenn Sie einen gMSA mit Containern verwenden möchten, die von Docker Swarm verwaltet werden, führen Sie den `--credential-spec` Befehl Andock [Dienst erstellen](https://docs.docker.com/engine/reference/commandline/service_create/) mit dem folgenden Parameter aus:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Weitere Informationen zum Verwenden von Anmeldeinformationen mit Andock Diensten finden Sie im Beispiel für andocker [Swarm](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) .

### <a name="how-to-use-gmsa-with-kubernetes"></a>Verwenden von gMSA mit Kubernetes

Die Unterstützung für die Planung von Windows-Containern mit gMSAs in Kubernetes ist als Alpha-Feature in Kubernetes 1,14 verfügbar. Unter [Konfigurieren von gMSA für Windows-Pods und-Containern](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa) finden Sie die neuesten Informationen zu diesem Feature und erfahren, wie Sie es in ihrer Kubernetes-Distribution testen.

## <a name="example-uses"></a>Verwendungsbeispiel

### <a name="sql-connection-strings"></a>SQL-Verbindungszeichenfolgen

Wenn ein Dienst als lokaler Systemdienst oder Netzwerkdienst in einem Container ausgeführt wird, kann die integrierte Windows-Authentifizierung genutzt werden, um eine Verbindung mit einer Microsoft SQL Server-Instanz herzustellen.

Nachfolgend finden Sie ein Beispiel für eine Verbindungszeichenfolge, die die Container Identität für die Authentifizierung bei SQL Server verwendet:

```sql
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

Erstellen Sie auf der Microsoft SQL Server-Instanz einen Anmeldenamen, und verwenden Sie dazu den Namen der Domäne und des gMSA gefolgt von einem $-Symbol. Nachdem Sie die Anmeldung erstellt haben, können Sie Sie einem Benutzer in einer Datenbank hinzufügen und ihm entsprechende Zugriffsberechtigungen zuweisen.

Beispiel:

```sql
CREATE LOGIN "DEMO\WebApplication1$"
    FROM WINDOWS
    WITH DEFAULT_DATABASE = "MusicStore"
GO

USE MusicStore
GO
CREATE USER WebApplication1 FOR LOGIN "DEMO\WebApplication1$"
GO

EXEC sp_addrolemember 'db_datareader', 'WebApplication1'
EXEC sp_addrolemember 'db_datawriter', 'WebApplication1'
```

Um es in Aktion zu sehen, schauen Sie sich die [aufgezeichnete Demo](https://youtu.be/cZHPz80I-3s?t=2672) an, die von Microsoft Ignite 2016 in der Sitzung "Walk the path to Containerisierung-Umwandlung von Arbeitslasten in Container" verfügbar ist.

## <a name="troubleshooting"></a>Problembehandlung

### <a name="known-issues"></a>Bekannte Probleme

#### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Der Container-Hostname muss dem gMSA-Namen für Windows Server 2016 und Windows 10, Versionen 1709 und 1803 entsprechen.

Wenn Sie mit Windows Server 2016, Version 1709 oder 1803 arbeiten, muss der Hostname ihres Containers mit Ihrem gMSA SAM-Kontonamen übereinstimmen.

Wenn der Hostname nicht mit dem gMSA-Namen übereinstimmt, schlagen eingehende NTLM-Authentifizierungsanforderungen und Name/sid-Übersetzung (von vielen Bibliotheken wie dem ASP.net-Mitgliedschaftsrollen Anbieter) fehl. Kerberos funktioniert auch dann weiterhin normal, wenn der Hostname und der gMSA-Name nicht übereinstimmen.

Diese Einschränkung wurde in Windows Server 2019 behoben, wobei der Container jetzt immer seinen gMSA-Namen im Netzwerk verwendet, unabhängig vom zugewiesenen Hostname.

#### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Die Verwendung eines gMSA mit mehr als einem Container führt gleichzeitig zu zeitweiligen Fehlern unter Windows Server 2016 und Windows 10, Versionen 1709 und 1803

Da alle Container denselben Hostnamen verwenden müssen, wirkt sich ein zweites Problem auf Versionen von Windows vor Windows Server 2019 und Windows 10, Version 1809, aus. Wenn mehreren Containern dieselbe Identität und Hostname zugewiesen werden, kann eine Racebedingung auftreten, wenn zwei Container gleichzeitig mit demselben Domänencontroller sprechen. Wenn ein anderer Container mit dem gleichen Domänencontroller kommuniziert, wird die Kommunikation mit allen vorherigen Containern mit derselben Identität abgebrochen. Dies kann zu zeitweiligen Authentifizierungsfehlern führen und kann manchmal als Vertrauensfehler beobachtet werden, wenn `nltest /sc_verify:contoso.com` Sie im Container ausgeführt werden.

Wir haben das Verhalten in Windows Server 2019 geändert, um die Container Identität vom Computernamen zu trennen, sodass mehrere Container dieselbe gMSA gleichzeitig verwenden können.

#### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Sie können gMSAs nicht mit isolierten Hyper-V-Containern unter Windows 10-Versionen 1703, 1709 und 1803 verwenden.

Die Container Initialisierung hängt oder schlägt fehl, wenn Sie versuchen, eine gMSA mit einem isolierten Hyper-V-Container unter Windows 10 und Windows Server-Versionen 1703, 1709 und 1803 zu verwenden.

Dieser Fehler wurde in Windows Server 2019 und Windows 10, Version 1809, behoben. Sie können auch isolierte Hyper-V-Container mit gMSAs unter Windows Server 2016 und Windows 10, Version 1607, ausführen.

### <a name="general-troubleshooting-guidance"></a>Allgemeine Anleitungen zur Problembehandlung

Wenn bei der Ausführung eines Containers mit einem gMSA Fehler auftreten, können Sie anhand der folgenden Anweisungen die Ursache ermitteln.

#### <a name="make-sure-the-host-can-use-the-gmsa"></a>Sicherstellen, dass der Host die gMSA verwenden kann

1. Überprüfen Sie, ob der Host der Domäne beigetreten ist und den Domänencontroller erreichen kann.
2. Installieren Sie die AD PowerShell-Tools von Remote Remote Tools, und führen Sie [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) aus, um festzustellen, ob der Computer Zugriff auf das Abrufen der gMSA hat. Wenn das Cmdlet " **false**" zurückgibt, hat der Computer keinen Zugriff auf das gMSA-Kennwort.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. Wenn **Test-ADServiceAccount** **false**zurückgibt, überprüfen Sie, ob der Host zu einer Sicherheitsgruppe gehört, die auf das gMSA-Kennwort zugreifen kann.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. Wenn Ihr Host zu einer Sicherheitsgruppe gehört, die zum Abrufen des gMSA-Kennworts autorisiert ist, aber weiterhin fehlerhaft ist, **Test-ADServiceAccount**, müssen Sie Ihren Computer möglicherweise neu starten, um ein neues Ticket zu erhalten, das die aktuellen Gruppenmitgliedschaften widerspiegelt.

#### <a name="check-the-credential-spec-file"></a>Überprüfen der Spezifikationsdatei für Anmeldeinformationen

1. Führen **Sie Get-CredentialSpec** aus dem [CredentialSpec-PowerShell-Modul](https://aka.ms/credspec) aus, um alle Anmelde Informations Spezifikationen auf dem Computer zu finden. Die Anmeldeinformationen müssen im Verzeichnis "CredentialSpecs" unter dem docker-Stammverzeichnis gespeichert werden. Sie können das docker-Stammverzeichnis finden, indem Sie andocker **-Info-f "{{ausführen. DockerRootDir}} "**.
2. Öffnen Sie die CredentialSpec-Datei, und stellen Sie sicher, dass die folgenden Felder ordnungsgemäß ausgefüllt sind:
    - **Sid**: die SID Ihres gMSA-Kontos
    - **MachineAccountName**: der Name des gMSA-SAM-Kontos (ohne vollständigen Domänennamen oder Dollarzeichen)
    - **DnsTreeName**: der FQDN Ihrer Active Directory-Gesamtstruktur
    - **DnsName**: der FQDN der Domäne, zu der das gMSA gehört
    - **** NetbiosName: NetBIOS-Name für die Domäne, zu der das gMSA gehört
    - **GroupManagedServiceAccounts/Name**: der Name des gMSA SAM-Kontos (ohne vollständigen Domänennamen oder Dollarzeichen)
    - **GroupManagedServiceAccounts/Scope**: ein Eintrag für den Domänen-FQDN und einer für den NetBIOS-Domänennamen

    Ihre Eingabe sollte wie im folgenden Beispiel einer vollständigen Anmelde Informations Spezifikation aussehen:

    ```json
    {
        "CmsPlugins": [
            "ActiveDirectory"
        ],
        "DomainJoinConfig": {
            "Sid": "S-1-5-21-702590844-1001920913-2680819671",
            "MachineAccountName": "webapp01",
            "Guid": "56d9b66c-d746-4f87-bd26-26760cfdca2e",
            "DnsTreeName": "contoso.com",
            "DnsName": "contoso.com",
            "NetBiosName": "CONTOSO"
        },
        "ActiveDirectoryConfig": {
            "GroupManagedServiceAccounts": [
                {
                    "Name": "webapp01",
                    "Scope": "contoso.com"
                },
                {
                    "Name": "webapp01",
                    "Scope": "CONTOSO"
                }
            ]
        }
    }
    ```

3. Überprüfen Sie, ob der Pfad zur Anmeldeinformationen-Spezifikationsdatei für Ihre Orchestrierungs Lösung richtig ist. Wenn Sie docker verwenden, stellen Sie sicher, dass der Befehl Container `--security-opt="credentialspec=file://NAME.json"`ausführen enthalten ist, wobei "Name. JSON" durch die Name-Ausgabe von **Get-CredentialSpec**ersetzt wird. Der Name ist ein Flatfile-Name, relativ zum CredentialSpecs-Ordner unter dem docker-Stammverzeichnis.

#### <a name="check-the-firewall-configuration"></a>Überprüfen der Firewall-Konfiguration

Wenn Sie eine strikte Firewall-Richtlinie im Container-oder Host Netzwerk verwenden, blockiert Sie möglicherweise erforderliche Verbindungen mit dem Active Directory-Domänen Controller oder DNS-Server.

| Protokoll und Port | Zweck |
|-------------------|---------|
| TCP und UDP 53 | DNS |
| TCP und UDP 88 | Kerberos |
| TCP 139 | Netlogon |
| TCP und UDP 389 | LDAP |
| TCP 636 | LDAP-SSL |

Möglicherweise müssen Sie den Zugriff auf zusätzliche Ports zulassen, je nachdem, welche Art von Datenverkehr Ihr Container an einen Domänencontroller sendet.
Eine vollständige Liste der von Active Directory verwendeten Ports finden Sie unter [Portanforderungen für Active Directory und Active Directory-Domänendienste](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers) .

#### <a name="check-the-container"></a>Überprüfen des Containers

1. Wenn Sie eine Windows-Version vor Windows Server 2019 oder Windows 10, Version 1809, ausführen, muss Ihr Container-Hostname mit dem gMSA-Namen übereinstimmen. Stellen Sie `--hostname` sicher, dass der Parameter dem Kurznamen gMSA (keine Domänenkomponente; beispielsweise "webapp01" anstelle von "webapp01.contoso.com") entspricht.

2. Überprüfen Sie die Container Netzwerkkonfiguration, um zu überprüfen, ob der Container einen Domänencontroller für die Domäne des gMSA auflösen und darauf zugreifen kann. Falsch konfigurierte DNS-Server im Container sind eine häufige Ursache für Identitätsprobleme.

3. Überprüfen Sie, ob der Container über eine gültige Verbindung zur Domäne verfügt, indem Sie das folgende Cmdlet im `docker exec` Container ausführen (mit oder einem Äquivalent):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    Die Vertrauensüberprüfung sollte zurück `NERR_SUCCESS` gegeben werden, wenn die gMSA verfügbar ist und die Netzwerkkonnektivität dem Container ermöglicht, mit der Domäne zu kommunizieren. Wenn dies nicht der Fall ist, überprüfen Sie die Netzwerkkonfiguration des Hosts und des Containers. Beide müssen in der Lage sein, mit dem Domänencontroller zu kommunizieren.

4. Stellen Sie sicher [, dass Ihre APP für die Verwendung von gMSA konfiguriert](#configure-your-application-to-use-the-gmsa)ist. Das Benutzerkonto innerhalb des Containers ändert sich nicht, wenn Sie ein gMSA verwenden. Stattdessen verwendet das System Konto das gMSA, wenn es mit anderen Netzwerkressourcen kommuniziert. Dies bedeutet, dass Ihre APP als Netzwerkdienst oder lokales System ausgeführt werden muss, um die gMSA-Identität zu nutzen.

    > [!TIP]
    > Wenn Sie einen `whoami` anderen Tool ausführen oder verwenden, um den aktuellen Benutzerkontext im Container zu identifizieren, wird der gMSA-Name nicht angezeigt. Dies liegt daran, dass Sie sich immer als lokaler Benutzer anstelle einer Domänenidentität beim Container anmelden. Das gMSA wird vom Computerkonto verwendet, wenn es mit Netzwerkressourcen kommuniziert, weshalb Ihre APP als Netzwerkdienst oder lokales System ausgeführt werden muss.

5. Wenn Ihr Container anscheinend richtig konfiguriert ist, aber Benutzer oder andere Dienste nicht automatisch bei ihrer Container-App authentifiziert werden können, überprüfen Sie die SPNs in Ihrem gMSA-Konto. Clients finden das gMSA-Konto unter dem Namen, in dem Sie Ihre Anwendung erreichen. Dies kann bedeuten, dass Sie zusätzliche `host` SPNs für Ihre gMSA benötigen, wenn beispielsweise Clients eine Verbindung mit Ihrer APP über ein Lastenausgleichsmodul oder einen anderen DNS-Namen herstellen.

## <a name="additional-resources"></a>Zusätzliche Ressourcen

- [Übersicht über Gruppen verwaltete Dienstkonten](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
