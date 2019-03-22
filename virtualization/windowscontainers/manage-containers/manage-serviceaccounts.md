---
title: Gruppenverwaltete Dienstkonten für Windows-Container
description: Gruppenverwaltete Dienstkonten für Windows-Container
keywords: Docker, Container, active Directory, gmsa
author: rpsqrd
ms.date: 03/21/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 5f80d245984b0cf5c4503971a74cc8bbcca0c19c
ms.sourcegitcommit: f53b8b3dc695cdf22106095b15698542140ae088
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/22/2019
ms.locfileid: "9257407"
---
# <a name="group-managed-service-accounts-for-windows-containers"></a>Gruppenverwaltete Dienstkonten für Windows-Container

In einem Windows-Netzwerk ist es üblich, Active Directory (AD) zu verwenden, um Authentifizierung und Autorisierung zwischen Benutzern, Computern und andere Netzwerkressourcen zu vereinfachen. Enterprise-Anwendungsentwicklern entwerfen häufig ihre apps werden AD-integrierte und auf Servern Domäne, der integrierten Windows-Authentifizierung, nutzen Sie dies macht es für Benutzer und andere Dienste automatisch und transparent Anmeldung einfach ausführen die Anwendung mit ihren Identitäten.

Obwohl Windows-Container-Domäne hinzugefügt werden können, können sie Active Directory-Domäne Identitäten weiterhin verwenden, um verschiedene Authentifizierungsszenarien zu unterstützen.
Um dies zu erreichen, können Sie einen Windows-Container mit einer [Gruppe verwaltete Dienstkonto](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA) – ausführen konfigurieren eine besondere Art von Dienstkonto eingeführt in Windows Server 2012, die konzipiert wurde, ohne eine Identität Teilen zwischen mehreren Computern zu ermöglichen das Kennwort zu kennen.
Wenn Sie einen Container mit einem gMSA auszuführen, wird der Container-Host Ruft das Kennwort gMSA von einer Active Directory-Domänencontroller ab und gibt es an die Container-Instanz.
Der Container wird die gMSA-Anmeldeinformationen verwenden, wenn das Computerkonto (SYSTEM) den Zugriff auf Netzwerkressourcen muss.

In diesem Artikel wird erläutert, wie beginnen, Active Directory gruppenverwaltete Dienstkonten mit Windows-Container.

## <a name="prerequisites"></a>Voraussetzungen

Um ein Windows-Container mit einem gruppenverwaltetes Dienstkonto auszuführen, benötigen Sie Folgendes:

**Active Directory-Domäne mit mindestens einem Domänencontroller mit WindowsServer 2012 oder höher.**
Es sind keine Gesamtstruktur oder Domäne functional Level für gmsas auch übertragen verwenden, aber die gMSA-Kennwörter können nur vertrieben wird von Windows Server 2012-Domänencontroller oder höher sein.
Weitere Informationen finden Sie in der [Active Directory-Anforderungen für gmsas auch übertragen](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req).



**Berechtigung ein gMSA-Konto erstellen.**
Um ein gMSA-Konto zu erstellen, müssen Sie ein Domänenadministrator sein oder ein Konto, das wurde die Berechtigung *Erstellen MsDS-GroupManagedServiceAccount Objekte* delegiert.



**Zugriff auf das Internet das CredentialSpec PowerShell-Modul herunterladen.**
Wenn Sie in eine getrennte Umgebung arbeiten, können Sie auf einem Computer mit Internet [Speichern Sie das Modul](https://docs.microsoft.com/en-us/powershell/module/powershellget/save-module?view=powershell-5.1) zugreifen und in die Entwicklung Computer oder Container-Host kopieren.

## <a name="one-time-preparation-of-active-directory"></a>Einmalige Vorbereitung von Active Directory

Wenn Sie bereits ein gMSA in Ihrer Domäne erstellt haben, müssen Sie wahrscheinlich den Schlüsselverteilungsdienst Root-Schlüssel zu generieren.
Die KDS ist verantwortlich für das Erstellen, drehen und das Kennwort gMSA auf autorisierte Hosts freigeben.
Wenn ein Container-Host gMSA verwenden, um einen Container auszuführen muss, wird es die KDS rufen Sie das aktuelle Kennwort kontaktieren.

Zum Überprüfen, ob die KDS--Schlüssel Root wurde bereits erstellt führen den folgenden PowerShell-Befehl als *Domänenadministrator* auf einem Domänencontroller oder Domänenmitglied mit den installierten AD-PowerShell-Tools:

```powershell
Get-KdsRootKey
```

Wenn der Befehl einen Schlüssel-ID die, Sie alle festgelegt und können fahren Sie fort mit Abschnitt [gruppenverwaltetes Dienstkonto erstellen](#create-a-group-managed-service-account) .
Andernfalls weiterhin auf die KDS-Stammschlüssels zu erstellen.

Führen Sie in einer produktionsumgebung oder die testumgebung mit mehreren Domänencontrollern den folgenden Befehl in PowerShell als *Domänenadministrator* zum Erstellen des KDS-Stammschlüssels.
Obwohl der Befehl, dass der Schlüssel sofort wirksam werden impliziert, müssen Sie warten 10 Stunden, bis die KDS-Stammschlüssels replizierten und für die Verwendung auf allen Domänencontrollern verfügbar ist.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

Wenn Sie nur einen Domänencontroller in der Domäne verfügen, können Sie die schneller durch Festlegen des Schlüssels effektive vor 10 Stunden sein.
Verwenden Sie dieses Verfahren nicht in einer produktionsumgebung.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>Erstellen Sie ein gruppenverwaltetes Dienstkonto

Alle Container, die integrierte Windows-Authentifizierung verwenden benötigt mindestens ein gMSA.
Primäre gMSA wird verwendet, wenn als *SYSTEMDIENST* oder *Netzwerkdienst* ausgeführten apps Zugriff auf Ressourcen im Netzwerk.
Der Name des gMSA wird der Container-Namen im Netzwerk, unabhängig von den Hostnamen der Container zugeordnet werden.
Container können auch mit gmsas auch zusätzliche übertragen, konfiguriert werden, für den Fall, dass ein Dienst oder eine Anwendung im Container als eine andere Identität über das Computerkonto Container ausgeführt werden soll.

Wenn Sie ein gMSA erstellen, erstellen Sie eine freigegebene Identität, die gleichzeitig auf vielen verschiedenen Computern verwendet werden kann.
Zugriff auf das gMSA-Kennwort wird durch eine Active Directory Access Control List geschützt.
Es wird empfohlen, erstellen eine Sicherheitsgruppe für jede gMSA-Kontos und dem Hinzufügen der relevanten containerhosts der Sicherheitsgruppe zum Zugriff auf das Kennwort zu beschränken.

Zum Schluss, da Container alle Dienstprinzipalnamen (SPN) nicht automatisch registrieren, müssen Sie mindestens einen "Host" SPN für Ihr gMSA-Konto manuell erstellen.
In der Regel dem Host oder HTTP-SPN mit den gleichen Namen wie die gMSA-Kontos registriert ist, jedoch müssen Sie möglicherweise einen anderen Dienstnamen zu verwenden, wenn Clients Zugriff auf die Anwendung in Containern hinter einem Lastenausgleich oder DNS-Namen, die sich aus dem gMSA-Namen unterscheidet.
Beispielsweise, wenn die gMSA-Kontos *WebApp01* aber Ihre Benutzer Zugriff auf der Website unter *mysite.contoso.com* registrieren Sie einen `http/mysite.contoso.com` SPN für das gMSA-Konto.
Einige Anwendungen erfordern zusätzliche SPN für ihre eindeutigen Protokolle – z. B. SQL Server erfordert, die SPN "MSSQLSvc/Hostname".

Die folgende Tabelle enthält die erforderlichen Attribute, wenn Sie ein gMSA zu erstellen.

gMSA-Eigenschaft | Erforderlicher Wert | Beispiel
--------------|----------------|--------
Name | Alle gültigen Kontonamen. | `WebApp01`
DnsHostName | Der Domänenname für den Kontonamen angefügt. | `WebApp01.contoso.com`
ServicePrincipalNames | Legen Sie mindestens als Host SPN, hinzuzufügen Sie andere Protokolle wie erforderlich. | `'host/WebApp01', 'host/WebApp01.contoso.com'`
PrincipalsAllowedToRetrieveManagedPassword | Der Sicherheitsgruppe mit containerhosts. | `WebApp01Hosts`

Sobald Sie wissen, was Sie Ihre gMSA aufrufen, führen Sie die folgenden Befehle in PowerShell zum Erstellen der Sicherheitsgruppe und gMSA.

> [!TIP]
> Sie müssen verwenden ein Konto, das wurde oder der Sicherheitsgruppe **Domänen-Admins** angehört delegiert die Berechtigung **Erstellen MsDS-GroupManagedServiceAccount-Objekte** , die folgenden Befehle ausführen.
> Das Cmdlet [New-ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) ist Teil der AD-PowerShell-Tools von [Remoteserver-Verwaltungstools](https://aka.ms/rsat).

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -Scope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

Es wird empfohlen, dass Sie separate gMSA-Konten für Ihre Umgebungen Dev, Test und Produktion erstellen.

## <a name="prepare-your-container-host"></a>Vorbereiten Sie Ihres containerhosts

Jeder Container-Host, die einen Windows-Container mit ein gMSA ausgeführt werden darf Domäne verbunden und haben Zugriff auf das gMSA Kennwort abzurufen.

1.  Beitreten eines Computers zu Ihrer Active Directory-Domäne.
2.  Stellen Sie sicher, dass Ihre Host auf die Sicherheitsgruppe Steuern des Zugriffs auf das gMSA Kennwort gehört.
3.  Starten Sie den Computer neu, damit sie die neue Gruppenmitgliedschaft erhält.
4.  Richten Sie [Docker Desktop für Windows 10](https://docs.docker.com/docker-for-windows/install/) oder [Docker für Windows Server](https://docs.docker.com/install/windows/docker-ee/).
5.  (Empfohlen) Stellen Sie sicher, dass der Host des gMSA-Kontos durch Ausführen von [Test-ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/activedirectory/test-adserviceaccount)verwenden kann. Wenn der Befehl **"false"** zurückgibt, finden Sie in den Abschnitt [Problembehandlung](#troubleshooting) diagnostic Schritte.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Erstellen Sie eine Credential-Spezifikation

Eine Credential-Spezifikationen-Datei ist ein JSON-Dokument, die Metadaten über die gMSA-Konten, dass Sie einen Container verwenden möchten.
Durch das Beibehalten der Identität Konfigurations getrennt von der Container-Image, können Sie einfach die gMSA vom Container verwendet wird, indem einfach Auslagerungsdatei die Anmeldeinformationen technisches – keine Code-Änderungen erforderlich ändern.

Die Spezifikationen Credential-Datei wird erstellt, verwenden das [CredentialSpec PowerShell-Modul](https://aka.ms/credspec) auf einem Host Domäne eingebunden Containers.
Nachdem Sie die Datei erstellt haben, können Sie es in anderen Container-Hosts oder Ihre Container-Orchestrator kopieren.
Die Spezifikationen Credential-Datei enthält keine vertrauliche Daten, z. B. das Kennwort gMSA, da der Container-Host gMSA im Auftrag des Containers abruft.

Docker erwartet die Anmeldeinformationen technisches Datei im Verzeichnis **CredentialSpecs** im Verzeichnis Data Docker.
In einer Standardinstallation finden Sie diese im Ordner " `C:\ProgramData\Docker\CredentialSpecs`.

Führen Sie die folgenden Schritte aus, um eine Spezifikation Credential-Datei auf dem Hostcontainer erstellen:
1.  Installieren Sie die Remoteserver-Verwaltungstools AD-PowerShell-tools
    -   Führen Sie für Windows Server `Install-WindowsFeature RSAT-AD-PowerShell`
    -   Für Windows 10, Version 1809 oder höher, ausgeführt werden `Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'`
    -   Ältere Versionen von Windows 10 finden Sie unterhttps://aka.ms/rsat
2.  Installieren Sie die neueste Version des [CredentialSpec PowerShell-Modul](https://aka.ms/credspec):

    ```powershell
    Install-Module CredentialSpec
    ```

    Wenn Sie Zugriff auf das Internet auf dem Hostcontainer besitzen, führen Sie `Save-Module CredentialSpec` auf einem Computer mit Internetzugriff, und kopieren Sie die Modul-Ordner auf `C:\Program Files\WindowsPowerShell\Modules` oder einer anderen Stelle im `$env:PSModulePath` auf dem containerhost.

3.  Erstellen Sie die Anmeldeinformationen technisches Datei:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    Standardmäßig wird das Cmdlet einen Cred-Spezifikation, die mit dem bereitgestellten gMSA-Namen als das Computerkonto für den Container erstellen.
    Die Datei wird im Verzeichnis Docker CredentialSpecs unter Verwendung des gMSA-Domäne und Konto für den Dateinamen gespeichert werden.

    Sie können eine Credential-Spezifikation erstellen, die zusätzliche gMSA-Konten enthalten, wenn Sie ein Dienst oder Prozess als eine sekundäre gMSA im Container ausführen.
    Verwenden Sie dazu die `-AdditionalAccounts` Parameter:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    Führen Sie eine vollständige Liste der unterstützten Parametern, `Get-Help New-CredentialSpec`.

4.  Sie können eine Liste der alle Credential-Spezifikationen und ihre vollständiger Pfad mit dem folgenden Befehl anzeigen:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="configuring-your-application-to-use-the-gmsa"></a>Konfigurieren die Anwendung gMSA

Bei der Standardkonfiguration erhält ein Container nur ein gMSA-Kontos das verwendet wird, wenn das Computerkonto Container versucht, Netzwerkressourcen authentifizieren.
Dies bedeutet, dass Ihre app muss als **Lokaler Systemdienst** oder **Netzwerkdienst** ausgeführt werden soll, wenn es verwendet, die gMSA-Identität muss.

### <a name="run-an-iis-app-pool-as-network-service"></a>Führen Sie einen IIS-App-Pool als Netzwerkdienst

Wenn Sie eine IIS-Website in den Container hostet sind, brauchen Sie tun müssen, um die gMSA nutzen festgelegt ist Ihre app-Pool-Identität mit **Netzwerkdienst**.
Sie können, die in die dockerfile-Datei vornehmen, indem Sie den folgenden Befehl aus:

```dockerfile
RUN (Get-IISAppPool DefaultAppPool).ProcessModel.IdentityType = "NetworkService"
```

Wenn Sie zuvor statische Benutzeranmeldeinformationen für die IIS-Anwendungspool verwendet, sollten Sie gMSA als Ersatz für diese Anmeldeinformationen.
Sie können die gMSA zwischen Dev, Test-und produktionsumgebungen ändern und IIS ruft automatisch die aktuelle Identität ohne das Container-Image zu ändern.

### <a name="run-a-windows-service-as-network-service"></a>Führen Sie einen Windows-Dienst als Netzwerkdienst

Wenn Ihre app in Containern als Windows-Dienst ausgeführt wird, können Sie den Dienst für die Ausführung als **Netzwerkdienst** in die dockerfile-Datei festlegen:

```dockerfile
RUN cmd /c 'sc.exe config "YourServiceName" obj= "NT AUTHORITY\NETWORK SERVICE" password= ""'
```

### <a name="run-arbitrary-console-apps-as-network-service"></a>Führen Sie beliebige Konsolen-apps als Netzwerkdienst

Generische Konsolen-apps, die nicht im Service-Manager oder IIS gehostet werden, ist es häufig am einfachsten, den Container als **Netzwerkdienst** ausgeführt wird, so, dass die app automatisch die gMSA-Kontext erbt.
Dieses Feature ist ab Windows Server Version 1709 verfügbar.

Fügen Sie die folgende Zeile, um Ihre Dockerfile, damit sie standardmäßig als Netzwerkdienst ausgeführt:

```dockerfile
USER "NT AUTHORITY\NETWORK SERVICE"
```

Sie können auch keine Verbindung zu einem Container als Netzwerkdienst einmal pro mit `docker exec`.
Dies ist besonders nützlich, wenn Sie Konnektivitätsprobleme in einem ausgeführten Container Problembehandlung sind, wenn der Container nicht in der Regel als Netzwerkdienst ausgeführt wird.

```powershell
# Opens an interactive PowerShell console in the container (id = 85d) as the Network Service account
docker exec -it --user "NT AUTHORITY\NETWORK SERVICE" 85d powershell
```

## <a name="run-a-container-with-a-gmsa"></a>Führen Sie einen Container mit einer gMSA

Um einen Container mit einem gMSA auszuführen, geben Sie den Credential Spezifikationen für die `--security-opt` Parameter der [Docker ausführen](https://docs.docker.com/engine/reference/run):

```powershell
# For Windows Server 2016, change the image name to mcr.microsoft.com/windows/servercore:ltsc2016
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Unter Windows Server 2016 1709 und 1803, der Hostnamen von der Container *übereinstimmen muss* gMSA kurze Namen.
Im obigen Beispiel ist gMSA SAM-Kontoname "webapp01", daher der Container-Hostnamen auf die gleiche festgelegt ist.
Auf Windows Server 2019 und später die Hostname-Feld ist nicht erforderlich, aber des Containers weiterhin bestimmt selbst durch den gMSA Namen anstelle der Hostname, auch wenn Sie ein anderes explizit angeben.

Um zu überprüfen, ob gMSA funktionsfähig ist, führen Sie den folgenden Befehl im Container:

```
# Replace contoso.com with your own domain
PS C:\> nltest /sc_verify:contoso.com

Flags: b0 HAS_IP  HAS_TIMESERV
Trusted DC Name \\dc01.contoso.com
Trusted DC Connection Status Status = 0 0x0 NERR_Success
Trust Verification Status = 0 0x0 NERR_Success
The command completed successfully
```

Wenn die *Vertrauenswürdige DC-Verbindungsstatus* und *Überprüfungsstatus vertrauen* nicht *NERR_Success*sind, sehen Sie sich den Abschnitt [Problembehandlung](#troubleshooting) , Tipps zum Debuggen des Problems.

Sie können die gMSA-Identität von innerhalb des Containers durch den folgenden Befehl ausführen und überprüfen den Namen des Client überprüfen:

```
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

Um PowerShell oder eine andere Konsolen-app mit dem gMSA-Konto zu öffnen, können Sie Fragen, den Container unter dem Konto anstelle der normalen ContainerAdministrator (oder ContainerUser für NanoServer) Systemkonto ausgeführt wird:

```powershell
# NOTE: you can only run as SYSTEM on Windows Server 1709 and later
docker run --security-opt "credentialspec=file://contoso_webapp01.json" --hostname webapp01 --user "NT AUTHORITY\SYSTEM" -it mcr.microsoft.com/windows/servercore:ltsc2019 powershell
```

Wenn Sie als SYSTEM ausführen, können Sie Netzwerkauthentifizierung als gMSA SYSVOL auf einem Domänencontroller herstellen einer Verbindung zu testen:

```
# This command should succeed if you're successfully running as the gMSA
PS C:\> dir \\contoso.com\SYSVOL


    Directory: \\contoso.com\sysvol


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d----l        2/27/2019   8:09 PM                contoso.com
```

## <a name="orchestrating-containers-with-gmsa"></a>Orchestrierung von Containern mit gMSA

In produktionsumgebungen werden verwenden häufig Sie ein Container-Orchestrator bereitstellen und Verwalten Ihrer apps und Diensten.
Jede Orchestrator verfügt über eigene Management-Paradigmen und ist verantwortlich für das akzeptieren Credential Spezifikationen, wenn Sie auf der Windows-Container-Plattform zu ermöglichen.

Wenn Sie Orchestrierung von Containern mit gmsas auch übertragen durchgeführt haben, überprüfen Sie Folgendes:
> [!div class="checklist"]
> * Alle containerhosts, die zum Ausführen von Containern mit gmsas auch übertragen geplant werden können, sind Domäne
> * Der Container-Hosts haben Zugriff auf die Kennwörter für alle gmsas auch übertragen von Containern verwendet abrufen
> * Die Spezifikationen Credential-Dateien erstellt und an der Orchestrator hochgeladen oder kopiert werden für jede Container-Host, je nachdem, wie der Orchestrator verdeutlicht diese behandelt werden sollen.
> * Container-Netzwerke ermöglichen die Container für die Kommunikation mit der Active Directory-Domänencontroller zum Abrufen von gMSA-tickets

### <a name="using-gmsa-with-service-fabric"></a>Verwenden von gMSA mit Service Fabric

Service Fabric unterstützt ausgeführten Windows-Container mit einem gMSA, wenn Sie den Spezifikationen Credential-Speicherort in Ihrem Anwendungsmanifest angeben.
Sie müssen die Spezifikationen Credential-Datei erstellen und in das Unterverzeichnis **CredentialSpecs** des Verzeichnisses Data Docker auf jedem Host platzieren, damit es von Service Fabric gefunden werden kann.
Die `Get-CredentialSpec` Teil des [CredentialSpec PowerShell-Modul](https://aka.ms/credspec)-Cmdlets zum Überprüfen, ob Ihre Anmeldeinformationen-Spezifikation am richtigen Speicherort ist verwendet werden kann.

Finden Sie unter [Schnellstart: Bereitstellen von Windows-Containern mit Service Fabric](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-quickstart-containers) und [Einrichten des gMSA für Windows-Container auf Service Fabric](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-setup-gmsa-for-windows-containers) , Weitere Informationen dazu, wie Sie Ihre Anwendung zu konfigurieren.

### <a name="using-gmsa-with-docker-swarm"></a>Verwenden von gMSA mit Docker-Schwarm

Um ein gMSA mit Containern, die vom Docker-Schwarm verwaltet verwenden möchten, verwenden Sie den Befehl [Docker-Dienst zu erstellen](https://docs.docker.com/engine/reference/commandline/service_create/) , mit der `--credential-spec` Parameter:

```powershell
docker service create --credential-spec "file://contoso_webapp01.json" --hostname "WebApp01" <image name>
```

Siehe [Beispiel Docker-Schwarm](https://docs.docker.com/engine/reference/commandline/service_create/#provide-credential-specs-for-managed-service-accounts-windows-only) für Weitere Informationen zur Verwendung von Anmeldeinformationen Spezifikationen mit Docker-Services.

### <a name="using-gmsa-with-kubernetes"></a>Verwenden von gMSA mit Kubernetes

Unterstützung für die Planung von Windows-Container mit gmsas auch übertragen in Kubernetes ist in der alpha-Unterstützung ab Kubernetes 1.14.
Überprüfen Sie die [Windows Gruppenverwaltete Dienstkonten für Container Identität Behaltn](https://github.com/kubernetes/enhancements/blob/master/keps/sig-windows/20181221-windows-group-managed-service-accounts-for-container-identity.md) für die neuesten Informationen zu diesem Feature und Informationen zum Testen in Ihre Kubernetes-Verteilung.

## <a name="example-uses"></a>Beispielverwendungen

### <a name="sql-connection-strings"></a>SQL-Verbindungszeichenfolgen
Wenn ein Dienst als lokaler Systemdienst oder Netzwerkdienst in einem Container ausgeführt wird, kann die integrierte Windows-Authentifizierung genutzt werden, um eine Verbindung mit einer Microsoft SQL Server-Instanz herzustellen.

Hier ist ein Beispiel für eine Verbindungszeichenfolge, die die Container-Identität verwendet wird, um zu SQL Server zu authentifizieren:

```
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

Erstellen Sie auf der Microsoft SQL Server-Instanz einen Anmeldenamen, und verwenden Sie dazu den Namen der Domäne und des gMSA gefolgt von einem $-Symbol.
Sobald der Anmeldename erstellt wurden, kann er einem Benutzer in einer Datenbank hinzugefügt werden und die erforderlichen Zugriffsrechte erhalten.

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

Sehen Sie sich dazu die [aufgezeichnet Demo](https://youtu.be/cZHPz80I-3s?t=2672) der Microsoft Ignite 2016-Sitzung „Walk the Path to Containerization - transforming workloads into containers“ an.

## <a name="troubleshooting"></a>Problembehandlung

### <a name="known-issues"></a>Bekannte Probleme

**Container-Hostnamen muss dem gMSA-Namen für WS2016, 1709 und 1803 entsprechen.**

Wenn Sie Windows Server 2016 ausführen, muss 1709 und 1803, der Hostnamen des Containers Ihrer gMSA SAM-Kontoname übereinstimmen.
Wenn die Hostname, der Name des gMSA nicht übereinstimmen, eingehenden NTLM-authentifizierungsanforderungen und Name/SID-Übersetzung (von vielen Bibliotheken wie der ASP.NET Mitgliedschaft Rollenanbieter verwendet) schlägt fehl.
Kerberos wird weiterhin normal funktionieren, auch wenn die Hostname nicht übereinstimmt.

Diese Einschränkung wurde in Windows Server 2019 behoben, in denen die Container jetzt immer den Namen des gMSA im Netzwerk unabhängig von den zugewiesenen Hostnamen verwendet wird.

**Verwenden ein gMSA gleichzeitig mit mehreren Containern führt zu Ausfälle auf WS2016, 1709 und 1803.**

Aufgrund der vorherige Einschränkung, erfordern alle Container mit dem Hostnamen wirkt sich auf ein weiteres Problem Versionen von Windows vor Windows Server 2019 und Windows 10, Version 1809.
Wenn mehrere Container die gleiche Identität und den Hostnamen zugeordnet sind, kann eine Racebedingung auftreten, wenn zwei Container gleichzeitig mit demselben Domänencontroller kommunizieren.
Wenn Sie einen anderen Container auf demselben Domänencontroller kommuniziert, wird die Kommunikation mit dem vorherigen Container mit der gleichen Identität verwerfen.
Dies kann dazu führen, dass intermittierender Authentifizierungsfehlern und manchmal als eine Vertrauensfehler beachtet werden, wenn Sie ausführen `nltest /sc_verify:contoso.com` innerhalb des Containers.

Wir geändert das Verhalten in Windows Server 2019, trennen die Container-Identität aus den Computernamen ein, sodass mehrere Container dasselbe gMSA gleichzeitig verwenden.

**Sie können nicht mit Hyper-V isolierte Container unter Windows, Version 1703, 1709 und 1803 gmsas auch übertragen verwenden.**

Container-Initialisierung wird hängen oder fehl, wenn Sie versuchen, ein gMSA mit einem isolierten Hyper-V-Container auf Windows 10 und Windows Server, Version 1703, 1709 und 1803 verwenden.

Dieser Fehler wurde in Windows Server 2019 und Windows 10, Version 1809 behoben. Sie können auch isolierte Hyper-V-Container unter Windows Server 2016 und Windows 10, Version 1607 mit gmsas auch übertragen ausführen.

### <a name="general-troubleshooting-guidance"></a>Allgemeine Anleitungen zur Problembehandlung

Wenn Sie Fehler auftreten, wenn einen Container mit einer gMSA ausgeführt, unterstützen die folgenden Schritte Sie die Ursache ermitteln.

**Stellen Sie sicher, dass der Host gMSA verwenden kann**

1.  Überprüfen Sie der Host Domäne verknüpft und kann den Domänencontroller zu erreichen.
2.  Installieren Sie die AD-PowerShell-Tools von RSAT, und führen Sie die [Test-ADServiceAccount](https://docs.microsoft.com/en-us/powershell/module/activedirectory/test-adserviceaccount) , um festzustellen, ob der Computer Zugriff gMSA abgerufen hat. Wenn das Cmdlet **"false"** zurückgegeben wird, hat der Computer Zugriff auf das gMSA Kennwort keinen.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Install-WindowsCapability -Online 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```
3.  Wenn Test-ADServiceAccount **"false"** zurückgegeben wird, stellen Sie sicher, dass der Host mit einer Sicherheitsgruppe gehört, die Zugriff auf das gMSA Kennwort abzurufen.

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4.  Ihre Host gehört zu einer Sicherheitsgruppe berechtigt, rufen Sie das Kennwort gMSA jedoch weiterhin Test-ADServiceAccount schlägt fehl, müssen Sie einen Neustart des Computers, damit sie ein neues Ticket reflektieren seine aktuelle Gruppenmitgliedschaften zu erhalten.

**Überprüfen Sie die Datei Credential-Spezifikation**

1.  Führen Sie `Get-CredentialSpec` aus dem [CredentialSpec PowerShell-Modul](https://aka.ms/credspec) zu suchen, die alle credential Spezifikationen auf dem Computer. Die Anmeldeinformationen Spezifikationen müssen im Verzeichnis "CredentialSpecs" unter dem Docker-Stammverzeichnis gespeichert werden. Sie finden die Docker Stammverzeichnis durch Ausführen `docker info -f "{{.DockerRootDir}}"`.
2.  Öffnen Sie die Datei CredentialSpec, und stellen Sie sicher, dass die folgenden Felder ordnungsgemäß ausgefüllt werden:
    -   **SID**: die SID des gMSA-Kontos
    -   **MachineAccountName**: gMSA SAM-Kontoname (enthalten keine vollständige Domänennamen oder Dollar-Zeichen)
    -   **DnsTreeName**: der FQDN des der AD-Gesamtstruktur
    -   **DNS-Name**: den FQDN der Domäne mit dem gMSA gehört
    -   **NetBiosName**: NetBIOS-Namen für die Domäne, zu dem gMSA gehört,
    -   **GroupManagedServiceAccounts/Name**: gMSA SAM-Kontoname (enthalten keine vollständige Domänennamen oder Dollar-Zeichen)
    -   **GroupManagedServiceAccounts/Umfang**: einen Eintrag für die Domäne FQDN und eine für die NETBIOS

    Finden Sie im vollständige Beispiel unten für eine vollständige Anmeldeinformationen-Spezifikation:

    ```json
    {
    "CmsPlugins":[
        "ActiveDirectory"
    ],
    "DomainJoinConfig":{
        "Sid":"S-1-5-21-702590844-1001920913-2680819671",
        "MachineAccountName":"webapp01",
        "Guid":"56d9b66c-d746-4f87-bd26-26760cfdca2e",
        "DnsTreeName":"contoso.com",
        "DnsName":"contoso.com",
        "NetBiosName":"CONTOSO"
    },
    "ActiveDirectoryConfig":{
        "GroupManagedServiceAccounts":[
        {
            "Name":"webapp01",
            "Scope":"contoso.com"
        },
        {
            "Name":"webapp01",
            "Scope":"CONTOSO"
        }
        ]
    }
    }
    ```

3.  Stellen Sie sicher, dass der Pfad zu der Spezifikationen Credential-Datei für die Orchestrierung Projektmappe richtig ist. Wenn Sie Docker verwenden, stellen Sie sicher, dass der Container-Ausführungsbefehl enthält `--security-opt="credentialspec=file://NAME.json"`, wobei "NAME.json" durch die Name-Ausgabe von **Get-CredentialSpec**ersetzt wird. Der Name ist eine flache Dateiname, relativ zum Ordner CredentialSpecs unter dem Docker-Stammverzeichnis.

**Überprüfen Sie den container**

1.  Wenn Sie eine Version von Windows vor Windows Server 2019 oder Windows 10, Version 1809, ausführen, muss die Container-Hostnamen die gMSA-Namen übereinstimmen. Sicherstellen der `--hostname` Parameter entspricht den gMSA kurzen Namen (keine Domänenkomponente, z. B. "webapp01" nicht "webapp01.contoso.com").

2.  Überprüfen Sie die Container-Netzwerkkonfiguration, überprüfen den Container auflösen und Zugriff auf einen Domänencontroller für die gMSA-Domäne kann. Falsch konfigurierte DNS-Servern im Container sind eine häufige Ursache der Identität Probleme.

3.  Überprüfen Sie, ob der Container eine gültige Verbindung mit der Domäne verfügt über mithilfe des folgenden Befehls im Container (mit `docker exec` oder ein Äquivalent):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    Der vertrauensüberprüfung sollte NERR_SUCCESS zurückgeben, wenn gMSA verfügbar ist und Netzwerkkonnektivität dem Container ermöglicht so sprechen Sie mit der Domäne.
    Schlägt fehl, überprüfen Sie die Netzwerkkonfiguration der Host und der Container – beide müssen mit dem Domänencontroller kommunizieren können.

4.  Stellen Sie sicher, dass Ihre app [für die Verwendung von gMSA konfiguriert](#configuring-your-application-to-use-the-gmsa)ist. Das Benutzerkonto innerhalb des Containers wird nicht geändert, wenn Sie ein gMSA verwenden – stattdessen das Systemkonto gMSA verwendet, wenn es auf andere Netzwerkressourcen kommuniziert. Daher muss die app wird als Netzwerkdienst oder lokales System Sie nutzen können, der die Identität des gMSA ausgeführt.

    > [!TIP]
    > Wenn Sie ausführen `whoami` oder versuchen Sie es mit einem anderen Tool und identifizieren Sie Ihre aktuellen Benutzerkontext im Container, sehen Sie den Namen des gMSA nie.
    > Dies ist, da Sie immer in den Container als lokaler Benutzer – keiner Domänenidentität anmelden.
    > GMSA wird durch das Computerkonto verwendet, wenn es kommuniziert mit Netzwerkressourcen, weshalb die app als Netzwerkdienst oder lokales System ausgeführt werden muss.

5.  Wenn Ihres Containers scheint ordnungsgemäß konfiguriert werden, aber Benutzer oder andere Dienste sind nicht automatisch an Ihre app in Containern authentifizieren, überprüfen Sie abschließend die SPN für das gMSA-Konto. Clients werden durch den Namen des gMSA-Kontos suchen an denen sie Ihre Anwendung erreichen. Dies kann bedeuten, dass Sie zusätzliche müssen `host` SPN für Ihre gMSA, wenn z. B. Clients für Ihre app über ein Lastenausgleich oder anderen DNS-Namen eine Verbindung herstellen.

## <a name="additional-resources"></a>Zusätzliche Ressourcen
-   [Gruppe Managed Service Accounts (Übersicht)](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)
