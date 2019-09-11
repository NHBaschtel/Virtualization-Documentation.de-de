---
title: Erstellen von gMSAs für Windows-Container
description: Erstellen von Gruppen-Managed-Service-Konten (gMSAs) für Windows-Container
keywords: docker, Container, Active Directory, GMSA, Gruppen verwaltetes Dienstkonto, Gruppen verwaltete Dienstkonten
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 9ed9029e534d56bfe1830281d0bfd3ddde0cee9e
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079664"
---
# <a name="create-gmsas-for-windows-containers"></a>Erstellen von gMSAs für Windows-Container

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
# To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
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
4. Einrichten des [andockbaren Desktops für Windows 10](https://docs.docker.com/docker-for-windows/install/) oder [Andocken für Windows Server](https://docs.docker.com/install/windows/docker-ee/)
5. Empfohlen Überprüfen Sie, ob der Host das gMSA-Konto verwenden kann, indem Sie [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)ausführen. Wenn der Befehl " **false**" zurückgibt, folgen Sie den [Anweisungen zur Fehlerbehebung](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa).

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Erstellen einer Anmelde Informations Spezifikation

Eine Anmeldeinformationen-spec-Datei ist ein JSON-Dokument, das Metadaten zu den gMSA-Konten enthält, die ein Container verwenden soll. Indem Sie die Identity-Konfiguration vom Container-Bild getrennt halten, können Sie ändern, welche gMSA der Container verwendet, indem Sie einfach die Spezifikationsdatei für die Anmeldeinformationen austauschen, keine Codeänderungen erforderlich.

Die Datei für die Anmeldeinformationen wird mithilfe des [CredentialSpec PowerShell-Moduls](https://aka.ms/credspec) auf einem Domänen verbundenen Container Host erstellt.
Nachdem Sie die Datei erstellt haben, können Sie Sie auf andere Container Hosts oder ihren Container Orchestrator kopieren.
Die Datei mit den Anmeldeinformationen enthält keine Geheimnisse, wie etwa das gMSA-Kennwort, da der Container Host die gMSA im Auftrag des Containers abruft.

Docker erwartet, dass die Anmeldeinformationsdatei unter dem **CredentialSpecs** -Verzeichnis im docker-Datenverzeichnis zu finden ist. In einer Standardinstallation finden Sie diesen Ordner unter `C:\ProgramData\Docker\CredentialSpecs`.

So erstellen Sie eine Anmelde Informations Spezifikationsdatei auf dem Container Host:

1. Installieren der Remotetools für AD PowerShell
    - Führen Sie für Windows Server die **Installations-Cmdlets-Remote Server-AD-PowerShell**aus.
    - Für Windows 10, Version 1809 oder höher, führen **Sie Add-WindowsCapability-Online-Name "Remote Host. ActiveDirectory. DS-LDS. Tools ~**~ ~ ~ 0.0.1.0" aus.
    - Informationen zu älteren Versionen von Windows 10 finden <https://aka.ms/rsat>Sie unter.
2. Führen Sie das folgende Cmdlet aus, um die neueste Version des [CredentialSpec PowerShell-Moduls](https://aka.ms/credspec)zu installieren:

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

## <a name="next-steps"></a>Nächste Schritte

Nachdem Sie Ihr gMSA-Konto eingerichtet haben, können Sie es wie folgt verwenden:

- [Konfigurieren von apps](gmsa-configure-app.md)
- [Ausführen von Containern](gmsa-run-container.md)
- [Orchestrierungs Container](gmsa-orchestrate-containers.md)

Wenn während des Setups Probleme auftreten, finden Sie in unserem [Leitfaden zur Problembehandlung](gmsa-troubleshooting.md) mögliche Lösungen.

## <a name="additional-resources"></a>Zusätzliche Ressourcen

- Weitere Informationen zu gMSAs finden Sie in der [Übersicht über Gruppen verwaltete Dienstkonten](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).
- Schauen Sie sich für eine Video Demonstration unsere [aufgezeichnete Demo](https://youtu.be/cZHPz80I-3s?t=2672) von Ignite 2016 an.
