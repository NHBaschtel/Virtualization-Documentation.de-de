---
title: Erstellen von gmsas für Windows-Container
description: Erstellen von Gruppen verwalteten Dienst Konten (Group Managed Service Accounts, gmsas) für Windows-Container.
keywords: docker, Container, Active Directory, GMSA, Gruppen verwaltetes Dienst Konto, Gruppen verwaltete Dienst Konten
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 9ed9029e534d56bfe1830281d0bfd3ddde0cee9e
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910250"
---
# <a name="create-gmsas-for-windows-containers"></a>Erstellen von gmsas für Windows-Container

Windows-basierte Netzwerke verwenden häufig Active Directory (AD), um die Authentifizierung und Autorisierung zwischen Benutzern, Computern und anderen Netzwerkressourcen zu vereinfachen. Entwickler von Unternehmensanwendungen entwerfen Ihre apps häufig so, dass Sie AD-integriert sind und auf in die Domäne eingebundenen Servern ausgeführt werden, um die integrierte Windows-Authentifizierung nutzen zu können. Dadurch können sich Benutzer und andere Dienste automatisch und transparent bei anmelden. die Anwendung mit ihren Identitäten.

Obwohl Windows-Container nicht in die Domäne aufgenommen werden können, können Sie weiterhin Active Directory Domänen Identitäten verwenden, um verschiedene Authentifizierungs Szenarien zu unterstützen.

Um dies zu erreichen, können Sie einen Windows-Container so konfigurieren, dass er mit einem [Gruppen verwalteten Dienst Konto (Group Managed Service Account](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) , GMSA) ausgeführt wird. dabei handelt es sich um einen speziellen Dienst Kontotyp, der in Windows Server 2012 eingeführt wurde

Wenn Sie einen Container mit einem GMSA ausführen, ruft der Container Host das GMSA-Kennwort von einem Active Directory Domänen Controller ab und übergibt es an die Container Instanz. Der Container verwendet die GMSA-Anmelde Informationen immer dann, wenn das Computer Konto (System) auf Netzwerkressourcen zugreifen muss.

In diesem Artikel wird erläutert, wie Sie mit der Verwendung Active Directory Gruppen verwalteten Dienst Konten mit Windows-Containern beginnen.

## <a name="prerequisites"></a>Voraussetzungen

Zum Ausführen eines Windows-Containers mit einem Gruppen verwalteten Dienst Konto benötigen Sie Folgendes:

- Eine Active Directory Domäne, bei der mindestens ein Domänen Controller unter Windows Server 2012 oder höher ausgeführt wird. Es gibt keine Anforderungen an die Gesamtstruktur-oder Domänen Funktionsebene für die Verwendung von gmsas, aber die GMSA-Kenn Wörter können nur von Domänen Controllern mit Windows Server 2012 oder höher verteilt werden. Weitere Informationen finden Sie unter [Active Directory Anforderungen für gmsas](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req).
- Berechtigung zum Erstellen eines GMSA-Kontos. Zum Erstellen eines GMSA-Kontos müssen Sie ein Domänen Administrator sein oder über ein Konto verfügen, dem die Berechtigung *Create MSDS-groupmanagedserviceaccount Objects* delegiert wurde.
- Internet Zugriff zum Herunterladen des PowerShell-Moduls "fidentialspec". Wenn Sie in einer getrennten Umgebung arbeiten, können Sie [das Modul](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1) auf einem Computer mit Internet Zugriff speichern und auf den Entwicklungs Computer oder den Container Host kopieren.

## <a name="one-time-preparation-of-active-directory"></a>Einmalige Vorbereitung Active Directory

Wenn Sie in Ihrer Domäne nicht bereits ein GMSA erstellt haben, müssen Sie den Schlüssel Verteilungsdienst-Stamm Schlüssel (Key Distribution Service, KDS) generieren. Die KDS ist für das Erstellen, drehen und Freigeben des GMSA-Kennworts auf autorisierten Hosts verantwortlich. Wenn ein Container Host das GMSA zum Ausführen eines Containers verwenden muss, wird er mit den KDS kontaktiert, um das aktuelle Kennwort abzurufen.

Um zu überprüfen, ob der KDS-Stamm Schlüssel bereits erstellt wurde, führen Sie das folgende PowerShell-Cmdlet als Domänen Administrator auf einem Domänen Controller oder Domänen Mitglied aus, auf dem die AD PowerShell-Tools installiert sind:

```powershell
Get-KdsRootKey
```

Wenn der Befehl eine Schlüssel-ID zurückgibt, sind Sie alle festgelegt und können den Abschnitt [Erstellen eines Gruppen verwalteten Dienst Kontos](#create-a-group-managed-service-account) überspringen. Andernfalls fahren Sie fort, um den KDS-Stamm Schlüssel zu erstellen.

Führen Sie in einer Produktionsumgebung oder Testumgebung mit mehreren Domänen Controllern das folgende Cmdlet in PowerShell als Domänen Administrator aus, um den KDS-Stamm Schlüssel zu erstellen.

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

Obwohl der Befehl impliziert, dass der Schlüssel sofort wirksam wird, müssen Sie 10 Stunden warten, bis der KDS-Stamm Schlüssel repliziert wurde und für die Verwendung auf allen Domänen Controllern verfügbar ist.

Wenn Sie nur über einen Domänen Controller in Ihrer Domäne verfügen, können Sie den Prozess beschleunigen, indem Sie den Schlüssel vor 10 Stunden festlegen.

>[!IMPORTANT]
>Verwenden Sie dieses Verfahren nicht in einer Produktionsumgebung.

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>Gruppenverwaltetes Dienstkonto erstellen

Jeder Container, der die integrierte Windows-Authentifizierung verwendet, benötigt mindestens ein GMSA. Das primäre GMSA wird immer dann verwendet, wenn apps, die als System oder Netzwerkdienst ausgeführt werden, auf Ressourcen im Netzwerk zugreifen. Der Name des GMSA wird zum Container Namen im Netzwerk, unabhängig vom Hostnamen, der dem Container zugewiesen ist. Container können auch mit zusätzlichen gmsas konfiguriert werden, falls Sie einen Dienst oder eine Anwendung im Container als eine andere Identität als das Container Computer Konto ausführen möchten.

Wenn Sie ein GMSA erstellen, erstellen Sie auch eine freigegebene Identität, die gleichzeitig auf vielen verschiedenen Computern verwendet werden kann. Der Zugriff auf das GMSA-Kennwort wird durch eine Active Directory Access Control Liste geschützt. Es wird empfohlen, eine Sicherheitsgruppe für jedes GMSA-Konto zu erstellen und die entsprechenden Container Hosts der Sicherheitsgruppe hinzuzufügen, um den Zugriff auf das Kennwort einzuschränken.

Da Container nicht automatisch Dienst Prinzipal Namen (SPN) registrieren, müssen Sie schließlich mindestens einen Host-SPN für Ihr GMSA-Konto erstellen.

In der Regel wird der Host oder der HTTP-SPN mit dem gleichen Namen wie das GMSA-Konto registriert, aber Sie müssen möglicherweise einen anderen Dienstnamen verwenden, wenn Clients auf die Containeranwendung hinter einem Load Balancer oder einem DNS-Namen, der sich vom GMSA-Namen unterscheidet, zugreifen.

Wenn das GMSA-Konto z. b. den Namen "WebApp01" hat, die Benutzer aber auf `mysite.contoso.com`auf die Website zugreifen, sollten Sie einen `http/mysite.contoso.com` SPN für das GMSA-Konto registrieren.

Für einige Anwendungen sind möglicherweise zusätzliche SPNs für Ihre eindeutigen Protokolle erforderlich. Beispielsweise ist für SQL Server der `MSSQLSvc/hostname` SPN erforderlich.

In der folgenden Tabelle sind die für das Erstellen eines GMSA erforderlichen Attribute aufgeführt.

|GMSA (Eigenschaft) | Erforderlicher Wert | Beispiel |
|--------------|----------------|--------|
|Name | Ein beliebiger gültiger Kontoname. | `WebApp01` |
|DNSHostName | Der Domänen Name, der an den Kontonamen angehängt wird. | `WebApp01.contoso.com` |
|ServicePrincipalNames | Legen Sie mindestens den Host-SPN fest, und fügen Sie ggf. weitere Protokolle hinzu. | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | Die Sicherheitsgruppe, die Ihre Container Hosts enthält. | `WebApp01Hosts` |

Nachdem Sie sich für den Namen des GMSA entschieden haben, führen Sie die folgenden Cmdlets in PowerShell aus, um die Sicherheitsgruppe und GMSA zu erstellen.

> [!TIP]
> Sie müssen ein Konto verwenden, das der Sicherheitsgruppe " **Domänen-Admins** " angehört oder die **Create MSDS-groupmanagedserviceaccount Objects** -Berechtigung zum Ausführen der folgenden Befehle delegiert hat.
> Das Cmdlet " [New-ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps) " ist Teil der AD PowerShell-Tools von [Remoteserver-Verwaltungstools](https://aka.ms/rsat).

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

Es wird empfohlen, separate GMSA-Konten für Ihre Entwicklungs-, Test-und Produktionsumgebungen zu erstellen.

## <a name="prepare-your-container-host"></a>Vorbereiten des Container Hosts

Jeder Container Host, auf dem ein Windows-Container mit einem GMSA ausgeführt wird, muss in die Domäne aufgenommen werden und Zugriff haben, um das GMSA-Kennwort abzurufen.

1. Fügen Sie Ihren Computer Ihrer Active Directory Domäne hinzu.
2. Stellen Sie sicher, dass Ihr Host der Sicherheitsgruppe angehört, die den Zugriff auf das GMSA-Kennwort steuert
3. Starten Sie den Computer neu, damit die neue Gruppenmitgliedschaft abgerufen wird.
4. Richten Sie [docker Desktop für Windows 10](https://docs.docker.com/docker-for-windows/install/) oder [docker für Windows Server](https://docs.docker.com/install/windows/docker-ee/)ein.
5. Empfohlen Vergewissern Sie sich, dass der Host das GMSA-Konto verwenden kann, indem Sie [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)ausführen. Wenn der Befehl **false**zurückgibt, befolgen Sie die [Anweisungen zur Problem](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa)Behandlung.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>Erstellen einer Anmelde Informations Spezifikation

Eine Datei mit Anmelde Informationen ist ein JSON-Dokument, das Metadaten zu den GMSA-Konten enthält, die von einem Container verwendet werden sollen. Wenn Sie die Identitäts Konfiguration getrennt vom Container Image aufbewahren, können Sie ändern, welche GMSA der Container verwendet, indem Sie einfach die Datei mit den Anmelde Informationen austauschen, ohne dass Codeänderungen erforderlich sind.

Die Datei mit den Anmelde Informationen wird mithilfe des [PowerShell-Moduls "PowerShell](https://aka.ms/credspec) " in einem in die Domäne eingebundenen Container Host erstellt.
Nachdem Sie die Datei erstellt haben, können Sie Sie auf andere Container Hosts oder Ihren containerorchestrator kopieren.
Die Datei mit den Anmelde Informationen enthält keine geheimen Schlüssel, wie z. b. das GMSA-Kennwort, da der Container Host das GMSA im Auftrag des Containers abruft.

Docker erwartet, dass sich die Datei mit den Anmelde Informationen im Verzeichnis "| **dentialspecs** " im Verzeichnis "docker Data" befindet. In einer Standardinstallation finden Sie diesen Ordner unter `C:\ProgramData\Docker\CredentialSpecs`.

So erstellen Sie eine Datei mit Anmelde Informations Spezifikationen auf dem Container Host:

1. Installieren der RSAT-AD PowerShell-Tools
    - Führen Sie für Windows Server **install-Windows Feature RSAT-AD-PowerShell**aus.
    - Führen Sie für Windows 10, Version 1809 oder höher, **Add-windowscapability-Online-Name "RSAT. ActiveDirectory. DS-LDS. Tools ~ ~ ~ ~ 0.0.1.0"** aus.
    - Informationen zu älteren Versionen von Windows 10 finden Sie unter <https://aka.ms/rsat>.
2. Führen Sie das folgende Cmdlet aus, um die neueste Version des [PowerShell-Moduls "PowerShell](https://aka.ms/credspec)" zu installieren:

    ```powershell
    Install-Module CredentialSpec
    ```

    Wenn Sie auf dem Container Host keinen Internet Zugriff haben, führen Sie `Save-Module CredentialSpec` auf einem mit dem Internet verbundenen Computer aus, und kopieren Sie den Modul Ordner auf `C:\Program Files\WindowsPowerShell\Modules` oder einen anderen Speicherort in `$env:PSModulePath` auf dem Container Host.

3. Führen Sie das folgende Cmdlet aus, um die neue Spezifikation für Anmelde Informationen zu erstellen:

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    Standardmäßig erstellt das Cmdlet mit dem bereitgestellten GMSA-Namen als Computer Konto für den Container eine "and"-Spezifikation. Die Datei wird im Verzeichnis "docker condentialspecs" mit der GMSA-Domäne und dem Kontonamen für den Dateinamen gespeichert.

    Sie können eine Spezifikation für die Anmelde Informationen erstellen, die zusätzliche GMSA-Konten enthält, wenn Sie einen Dienst oder Prozess als sekundäres GMSA im Container ausführen. Verwenden Sie hierzu den `-AdditionalAccounts`-Parameter:

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    Führen Sie `Get-Help New-CredentialSpec`aus, um eine vollständige Liste unterstützter Parameter zu erhalten.

4. Mit dem folgenden Cmdlet können Sie eine Liste aller Spezifikationen für Anmelde Informationen und deren vollständigen Pfad anzeigen:

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>Nächste Schritte

Nachdem Sie Ihr GMSA-Konto eingerichtet haben, können Sie es für Folgendes verwenden:

- [Konfigurieren von Apps](gmsa-configure-app.md)
- [Ausführen von Containern](gmsa-run-container.md)
- [Orchestrieren von Containern](gmsa-orchestrate-containers.md)

Wenn während des Setups Probleme auftreten, finden Sie in unserem [Handbuch zur Problem](gmsa-troubleshooting.md) Behandlung mögliche Lösungen.

## <a name="additional-resources"></a>Zusätzliche Ressourcen

- Weitere Informationen zu gmsas finden Sie unter [Gruppen verwaltete Dienst Konten: Übersicht](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview).
- Sehen Sie sich für eine Video Demonstration unsere [aufgezeichnete Demo](https://youtu.be/cZHPz80I-3s?t=2672) von Ignite 2016 an.
