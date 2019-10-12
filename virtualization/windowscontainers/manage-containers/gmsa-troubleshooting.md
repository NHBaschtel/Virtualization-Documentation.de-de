---
title: Problembehandlung bei gMSAs für Windows-Container
description: Behandeln von Problemen mit Gruppen-Managed-Service-Konten (gMSAs) für Windows-Container
keywords: docker, Container, Active Directory, GMSA, Group Managed Service-Konto, Gruppen-Managed-Service-Konten, Problembehandlung, Problembehandlung
author: rpsqrd
ms.date: 10/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 89f255e307c2a48fd743d5abd1a49bba7703aaf3
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209850"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Problembehandlung bei gMSAs für Windows-Container

## <a name="known-issues"></a>Bekannte Probleme

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Der Container-Hostname muss dem gMSA-Namen für Windows Server 2016 und Windows 10, Versionen 1709 und 1803 entsprechen.

Wenn Sie mit Windows Server 2016, Version 1709 oder 1803 arbeiten, muss der Hostname ihres Containers mit Ihrem gMSA SAM-Kontonamen übereinstimmen.

Wenn der Hostname nicht mit dem gMSA-Namen übereinstimmt, schlagen eingehende NTLM-Authentifizierungsanforderungen und Name/sid-Übersetzung (von vielen Bibliotheken wie dem ASP.net-Mitgliedschaftsrollen Anbieter) fehl. Kerberos funktioniert auch dann weiterhin normal, wenn der Hostname und der gMSA-Name nicht übereinstimmen.

Diese Einschränkung wurde in Windows Server 2019 behoben, wobei der Container jetzt immer seinen gMSA-Namen im Netzwerk verwendet, unabhängig vom zugewiesenen Hostname.

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Die Verwendung eines gMSA mit mehr als einem Container führt gleichzeitig zu zeitweiligen Fehlern unter Windows Server 2016 und Windows 10, Versionen 1709 und 1803

Da alle Container denselben Hostnamen verwenden müssen, wirkt sich ein zweites Problem auf Versionen von Windows vor Windows Server 2019 und Windows 10, Version 1809, aus. Wenn mehreren Containern dieselbe Identität und Hostname zugewiesen werden, kann eine Racebedingung auftreten, wenn zwei Container gleichzeitig mit demselben Domänencontroller sprechen. Wenn ein anderer Container mit dem gleichen Domänencontroller kommuniziert, wird die Kommunikation mit allen vorherigen Containern mit derselben Identität abgebrochen. Dies kann zu zeitweiligen Authentifizierungsfehlern führen und kann manchmal als Vertrauensfehler beobachtet werden, wenn `nltest /sc_verify:contoso.com` Sie im Container ausgeführt werden.

Wir haben das Verhalten in Windows Server 2019 geändert, um die Container Identität vom Computernamen zu trennen, sodass mehrere Container dieselbe gMSA gleichzeitig verwenden können.

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Sie können gMSAs nicht mit isolierten Hyper-V-Containern unter Windows 10-Versionen 1703, 1709 und 1803 verwenden.

Die Container Initialisierung hängt oder schlägt fehl, wenn Sie versuchen, eine gMSA mit einem isolierten Hyper-V-Container unter Windows 10 und Windows Server-Versionen 1703, 1709 und 1803 zu verwenden.

Dieser Fehler wurde in Windows Server 2019 und Windows 10, Version 1809, behoben. Sie können auch isolierte Hyper-V-Container mit gMSAs unter Windows Server 2016 und Windows 10, Version 1607, ausführen.

## <a name="general-troubleshooting-guidance"></a>Allgemeine Anleitungen zur Problembehandlung

Wenn bei der Ausführung eines Containers mit einem gMSA Fehler auftreten, können Sie anhand der folgenden Anweisungen die Ursache ermitteln.

### <a name="make-sure-the-host-can-use-the-gmsa"></a>Sicherstellen, dass der Host die gMSA verwenden kann

1. Überprüfen Sie, ob der Host der Domäne beigetreten ist und den Domänencontroller erreichen kann.
2. Installieren Sie die AD PowerShell-Tools von Remote Remote Tools, und führen Sie [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) aus, um festzustellen, ob der Computer Zugriff auf das Abrufen der gMSA hat. Wenn das Cmdlet " **false**" zurückgibt, hat der Computer keinen Zugriff auf das gMSA-Kennwort.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
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

1. Führen **Sie Get-CredentialSpec** aus dem [CredentialSpec-PowerShell-Modul](https://aka.ms/credspec) aus, um alle Anmelde Informations Spezifikationen auf dem Computer zu finden. Die Anmeldeinformationen müssen im Verzeichnis "CredentialSpecs" unter dem docker-Stammverzeichnis gespeichert werden. Sie können das docker-Stammverzeichnis finden, indem Sie **andocker-Info-f "{{ausführen. DockerRootDir}} "**.
2. Öffnen Sie die CredentialSpec-Datei, und stellen Sie sicher, dass die folgenden Felder ordnungsgemäß ausgefüllt sind:
    - **Sid**: die SID Ihres gMSA-Kontos
    - **MachineAccountName**: der Name des gMSA-SAM-Kontos (ohne vollständigen Domänennamen oder Dollarzeichen)
    - **DnsTreeName**: der FQDN Ihrer Active Directory-Gesamtstruktur
    - **DnsName**: der FQDN der Domäne, zu der das gMSA gehört
    - **NetbiosName**: NetBIOS-Name für die Domäne, zu der das gMSA gehört
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

### <a name="check-the-firewall-configuration"></a>Überprüfen der Firewall-Konfiguration

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

### <a name="check-the-container"></a>Überprüfen des Containers

1. Wenn Sie eine Windows-Version vor Windows Server 2019 oder Windows 10, Version 1809, ausführen, muss Ihr Container-Hostname mit dem gMSA-Namen übereinstimmen. Stellen Sie `--hostname` sicher, dass der Parameter dem Kurznamen gMSA (keine Domänenkomponente; beispielsweise "webapp01" anstelle von "webapp01.contoso.com") entspricht.

2. Überprüfen Sie die Container Netzwerkkonfiguration, um zu überprüfen, ob der Container einen Domänencontroller für die Domäne des gMSA auflösen und darauf zugreifen kann. Falsch konfigurierte DNS-Server im Container sind eine häufige Ursache für Identitätsprobleme.

3. Überprüfen Sie, ob der Container über eine gültige Verbindung zur Domäne verfügt, indem Sie das folgende Cmdlet im `docker exec` Container ausführen (mit oder einem Äquivalent):

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    Die Vertrauensüberprüfung sollte zurück `NERR_SUCCESS` gegeben werden, wenn die gMSA verfügbar ist und die Netzwerkkonnektivität dem Container ermöglicht, mit der Domäne zu kommunizieren. Wenn dies nicht der Fall ist, überprüfen Sie die Netzwerkkonfiguration des Hosts und des Containers. Beide müssen in der Lage sein, mit dem Domänencontroller zu kommunizieren.

4. Überprüfen Sie, ob der Container ein gültiges Kerberos-Ticket Genehmigungs Ticket (TGT) erhalten kann:

    ```powershell
    klist get krbtgt
    ```

    Dieser Befehl sollte "ein Ticket zu KRBTGT wurde erfolgreich abgerufen" zurückgeben und den zum Abrufen des Tickets verwendeten Domänencontroller auflisten. Wenn Sie in der Lage sind, eine TGT `nltest` zu erhalten, aber aus dem vorherigen Schritt fehlschlägt, kann dies ein Hinweis darauf sein, dass das gMSA-Konto falsch konfiguriert ist. Weitere Informationen finden Sie unter [Überprüfen des gMSA-Kontos](#check-the-gmsa-account) .

    Wenn Sie innerhalb des Containers keine TGT abrufen können, kann dies auf DNS-oder Netzwerkverbindungsprobleme hindeuten. Stellen Sie sicher, dass der Container einen Domänencontroller mithilfe des Domänen-DNS-Namens auflösen kann und der Domänencontroller vom Container geroutet werden kann.

5. Stellen Sie sicher [, dass Ihre APP für die Verwendung von gMSA konfiguriert](gmsa-configure-app.md)ist. Das Benutzerkonto innerhalb des Containers ändert sich nicht, wenn Sie ein gMSA verwenden. Stattdessen verwendet das System Konto das gMSA, wenn es mit anderen Netzwerkressourcen kommuniziert. Dies bedeutet, dass Ihre APP als Netzwerkdienst oder lokales System ausgeführt werden muss, um die gMSA-Identität zu nutzen.

    > [!TIP]
    > Wenn Sie einen `whoami` anderen Tool ausführen oder verwenden, um den aktuellen Benutzerkontext im Container zu identifizieren, wird der gMSA-Name nicht angezeigt. Dies liegt daran, dass Sie sich immer als lokaler Benutzer anstelle einer Domänenidentität beim Container anmelden. Das gMSA wird vom Computerkonto verwendet, wenn es mit Netzwerkressourcen kommuniziert, weshalb Ihre APP als Netzwerkdienst oder lokales System ausgeführt werden muss.

### <a name="check-the-gmsa-account"></a>Überprüfen des gMSA-Kontos

1. Wenn Ihr Container anscheinend richtig konfiguriert ist, aber Benutzer oder andere Dienste nicht automatisch bei ihrer Container-App authentifiziert werden können, überprüfen Sie die SPNs in Ihrem gMSA-Konto. Clients finden das gMSA-Konto unter dem Namen, in dem Sie Ihre Anwendung erreichen. Dies kann bedeuten, dass Sie zusätzliche `host` SPNs für Ihre gMSA benötigen, wenn beispielsweise Clients eine Verbindung mit Ihrer APP über ein Lastenausgleichsmodul oder einen anderen DNS-Namen herstellen.

2. Stellen Sie sicher, dass der gMSA und der Container Host zur gleichen Active Directory-Domäne gehören. Der Container Host kann das gMSA-Kennwort nicht abrufen, wenn das gMSA zu einer anderen Domäne gehört.

3. Stellen Sie sicher, dass nur ein Konto in Ihrer Domäne mit dem gleichen Namen wie Ihr gMSA vorhanden ist. gMSA-Objekten sind Dollarzeichen ($) an Ihre SAM-Kontonamen angefügt, sodass es möglich ist, dass ein gMSA-Objekt "Mein Konto $" und ein nicht verknüpftes Benutzerkonto mit dem Namen "Mein Konto" in der gleichen Domäne benannt werden. Dies kann zu Problemen führen, wenn der Domänencontroller oder die Anwendung den gMSA nach Namen nachschlagen muss. Mit dem folgenden Befehl können Sie die Anzeige nach ähnlich benannten Objekten durchsuchen:

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. Wenn Sie für das gMSA-Konto eine uneingeschränkte Delegierung aktiviert haben, stellen Sie sicher, dass das userAccountControl `WORKSTATION_TRUST_ACCOUNT` - [Attribut](https://support.microsoft.com/en-us/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties) weiterhin aktiviert ist. Dieses Flag ist für Netlogon im Container erforderlich, um mit dem Domänencontroller zu kommunizieren, wie es der Fall ist, wenn eine APP einen Namen in eine SID auflösen soll oder umgekehrt. Mit den folgenden Befehlen können Sie überprüfen, ob die Kennzeichnung richtig konfiguriert ist:

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    Wenn die obigen Befehle zurück `False`gegeben werden, verwenden Sie die folgenden `WORKSTATION_TRUST_ACCOUNT` Optionen, um das Flag zur userAccountControl-Eigenschaft des gMSA-Kontos hinzuzufügen. Mit diesem Befehl werden auch die `NORMAL_ACCOUNT`, `INTERDOMAIN_TRUST_ACCOUNT`und `SERVER_TRUST_ACCOUNT` Flags aus der userAccountControl-Eigenschaft gelöscht.

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```
