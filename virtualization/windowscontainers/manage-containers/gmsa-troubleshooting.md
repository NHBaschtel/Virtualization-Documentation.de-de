---
title: Behandeln von Problemen mit gmsas für Windows-Container
description: Behandeln von Problemen mit Gruppen verwalteten Dienst Konten (Group Managed Service Accounts, gmsas) für Windows-Container.
keywords: docker, Container, Active Directory, GMSA, Gruppen verwaltetes Dienst Konto, Gruppen verwaltete Dienst Konten, Problembehandlung, Problembehandlung
author: rpsqrd
ms.date: 10/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 89f255e307c2a48fd743d5abd1a49bba7703aaf3
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910240"
---
# <a name="troubleshoot-gmsas-for-windows-containers"></a>Behandeln von Problemen mit gmsas für Windows-Container

## <a name="known-issues"></a>Bekannte Probleme

### <a name="container-hostname-must-match-the-gmsa-name-for-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Der Container Hostname muss mit dem GMSA-Namen für Windows Server 2016 und Windows 10, Version 1709 und 1803, identisch sein.

Wenn Sie Windows Server 2016, Version 1709 oder 1803 ausführen, muss der Hostname ihres Containers mit dem Kontonamen des GMSA SAM-Kontos identisch sein.

Wenn der Hostname nicht mit dem GMSA-Namen identisch ist, schlagen eingehende NTLM-Authentifizierungsanforderungen und die namens-/sid-Übersetzung (die von vielen Bibliotheken verwendet wird, wie der ASP.net-Mitgliedschafts Rollen Anbieter) fehl Kerberos funktioniert auch dann weiterhin normal, wenn der Hostname und der GMSA-Name nicht mit dem Namen identisch sind.

Diese Einschränkung wurde in Windows Server 2019 behoben, bei dem der Container nun immer seinen GMSA-Namen im Netzwerk verwendet, unabhängig vom zugewiesenen Hostnamen.

### <a name="using-a-gmsa-with-more-than-one-container-simultaneously-leads-to-intermittent-failures-on-windows-server-2016-and-windows-10-versions-1709-and-1803"></a>Die gleichzeitige Verwendung eines GMSA mit mehr als einem Container führt zu zeitweiligen Ausfällen auf Windows Server 2016 und Windows 10, Version 1709 und 1803.

Da für alle Container derselbe Hostname verwendet werden muss, wirkt sich ein zweites Problem auf Windows-Versionen vor Windows Server 2019 und Windows 10, Version 1809, aus. Wenn mehreren Containern dieselbe Identität und derselbe Hostname zugewiesen wird, kann eine Racebedingung eintreten, wenn zwei Container gleichzeitig mit demselben Domänen Controller kommunizieren. Wenn ein anderer Container mit dem gleichen Domänen Controller kommuniziert, wird die Kommunikation mit allen vorherigen Containern, die dieselbe Identität verwenden, abgebrochen. Dies kann zu zeitweiligen Authentifizierungs Fehlern führen und kann gelegentlich als Vertrauens Fehler erkannt werden, wenn Sie `nltest /sc_verify:contoso.com` innerhalb des Containers ausführen.

Wir haben das Verhalten in Windows Server 2019 so geändert, dass die Container Identität vom Computernamen getrennt wird, sodass mehrere Container gleichzeitig das gleiche GMSA verwenden können.

### <a name="you-cant-use-gmsas-with-hyper-v-isolated-containers-on-windows-10-versions-1703-1709-and-1803"></a>Sie können gmsas nicht mit isolierten Hyper-V-Containern unter Windows 10, Version 1703, 1709 und 1803 verwenden.

Die Container Initialisierung bleibt bestehen oder schlägt fehl, wenn Sie versuchen, ein GMSA mit einem isolierten Hyper-V-Container unter Windows 10 und Windows Server, Version 1703, 1709 und 1803, zu verwenden.

Dieser Fehler wurde in Windows Server 2019 und Windows 10, Version 1809, korrigiert. Sie können auch isolierte Hyper-V-Container mit gmsas unter Windows Server 2016 und Windows 10, Version 1607, ausführen.

## <a name="general-troubleshooting-guidance"></a>Allgemeine Anleitungen zur Problembehandlung

Wenn beim Ausführen eines Containers mit einem GMSA Fehler auftreten, können Sie anhand der folgenden Anweisungen die Ursache ermitteln.

### <a name="make-sure-the-host-can-use-the-gmsa"></a>Stellen Sie sicher, dass der Host das GMSA verwenden kann.

1. Vergewissern Sie sich, dass der Host einer Domäne beigetreten ist und den Domänen Controller erreichen kann
2. Installieren Sie die AD PowerShell-Tools von RSAT, und führen Sie [Test-ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount) aus, um zu überprüfen, ob der Computer Zugriff zum Abrufen des GMSA hat. Wenn das Cmdlet **false**zurückgibt, hat der Computer keinen Zugriff auf das GMSA-Kennwort.

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

3. Wenn " **Test-ADServiceAccount** " den Wert " **false**" zurückgibt, überprüfen Sie, ob der Host zu einer Sicherheitsgruppe gehört, die auf das GMSA

    ```powershell
    # Get the current computer's group membership
    Get-ADComputer $env:computername | Get-ADPrincipalGroupMembership | Select-Object DistinguishedName

    # Get the groups allowed to retrieve the gMSA password
    # Change "WebApp01" for your own gMSA name
    (Get-ADServiceAccount WebApp01 -Properties PrincipalsAllowedToRetrieveManagedPassword).PrincipalsAllowedToRetrieveManagedPassword
    ```

4. Wenn Ihr Host zu einer Sicherheitsgruppe gehört, die zum Abrufen des GMSA-Kennworts autorisiert ist, aber weiterhin " **Test-ADServiceAccount**" lautet, müssen Sie möglicherweise den Computer neu starten, um ein neues Ticket zu erhalten, das die aktuellen Gruppenmitgliedschaften widerspiegelt.

#### <a name="check-the-credential-spec-file"></a>Überprüfen Sie die Datei mit den Anmelde Informationen.

1. Führen **Sie "Get-alidentialspec** " aus dem [PowerShell-Modul "PowerShell](https://aka.ms/credspec) " aus, um alle Spezifikationen der Anmelde Informationen auf dem Computer zu suchen. Die Spezifikationen der Anmelde Informationen müssen im Verzeichnis "| dentialspecs" unter dem docker-Stammverzeichnis gespeichert werden. Sie finden das docker-Stammverzeichnis, indem Sie **docker Info-f "{{ausführen. Dockerrootdir}} "** .
2. Öffnen Sie die Datei "andentialspec", und stellen Sie sicher, dass die folgenden Felder ordnungsgemäß ausgefüllt sind:
    - **Sid**: die SID des GMSA-Kontos
    - **Machineaccountname**: der GMSA SAM-Kontoname (vollständiger Domänen Name oder Dollarzeichen nicht enthalten)
    - **Dnstreename**: der voll qualifizierte Name Ihrer Active Directory-Gesamtstruktur
    - **DnsName**: der FQDN der Domäne, zu der das GMSA gehört.
    - **NetbiosName**: NetBIOS-Name für die Domäne, zu der das GMSA gehört.
    - **Groupmanagedserviceaccounts/Name**: der GMSA SAM-Kontoname (vollständiger Domänen Name oder Dollarzeichen nicht enthalten)
    - **Groupmanagedserviceaccounts/Bereich**: ein Eintrag für den Domänen-voll qualifizierten Domänen Namen und einen Eintrag für NetBIOS

    Ihre Eingabe sollte wie im folgenden Beispiel für eine komplette Spezifikation für Anmelde Informationen aussehen:

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

3. Vergewissern Sie sich, dass der Pfad zur Datei mit den Anmelde Informationen für die Orchestrierungs Lösung richtig ist. Wenn Sie docker verwenden, stellen Sie sicher, dass der Container Run-Befehl `--security-opt="credentialspec=file://NAME.json"`enthält, wobei "Name. JSON" durch die Name-Ausgabe durch " **Get-kredentialspec**" ersetzt wird. Der Name ist ein flatfilename, der relativ zum Ordner "Ordner" unter dem docker-Stammverzeichnis ist.

### <a name="check-the-firewall-configuration"></a>Überprüfen der Firewallkonfiguration

Wenn Sie eine strikte Firewallrichtlinie für den Container oder das Host Netzwerk verwenden, werden möglicherweise die erforderlichen Verbindungen zum Active Directory-Domäne Controller oder DNS-Server blockiert.

| Protokoll und Port | Zweck |
|-------------------|---------|
| TCP und UDP 53 | DNS |
| TCP und UDP 88 | Kerberos |
| TCP 139 | Anmeldedienst |
| TCP und UDP 389 | LDAP |
| TCP 636 | LDAP SSL |

Abhängig vom Typ des Datenverkehrs, den Ihr Container an einen Domänen Controller sendet, müssen Sie möglicherweise den Zugriff auf zusätzliche Ports zulassen.
Eine vollständige Liste der von Active Directory verwendeten Ports finden Sie unter [Active Directory-und Active Directory Domain Services Port Anforderungen](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772723(v=ws.10)#communication-to-domain-controllers) .

### <a name="check-the-container"></a>Überprüfen des Containers

1. Wenn Sie eine Windows-Version vor Windows Server 2019 oder Windows 10, Version 1809, ausführen, muss Ihr containerhostname dem GMSA-Namen entsprechen. Stellen Sie sicher, dass der `--hostname` Parameter mit dem GMSA-Kurznamen (keine Domänen Komponente, z. b. "webapp01" anstelle von "webapp01.contoso.com") übereinstimmt.

2. Überprüfen Sie die Konfiguration des Container Netzwerks, um sicherzustellen, dass der Container einen Domänen Controller für die Domäne des GMSA auflösen und darauf zugreifen kann. Falsch konfigurierte DNS-Server im Container sind ein gängiger Übeltäter für Identitätsprobleme.

3. Überprüfen Sie, ob der Container über eine gültige Verbindung mit der Domäne verfügt, indem Sie das folgende Cmdlet im Container (mit `docker exec` oder einer entsprechenden) ausführen:

    ```powershell
    nltest /sc_verify:contoso.com
    ```

    Die Überprüfung der Vertrauensstellung sollte `NERR_SUCCESS` zurückgeben, wenn das GMSA verfügbar ist und die Netzwerk Konnektivität dem Container ermöglicht, mit der Domäne zu kommunizieren. Wenn dies nicht der Fall ist, überprüfen Sie die Netzwerkkonfiguration des Hosts und Containers. Beide müssen mit dem Domänen Controller kommunizieren können.

4. Überprüfen Sie, ob der Container ein gültiges Kerberos-Ticket zum Gewähren von Tickets (TGT) abrufen kann:

    ```powershell
    klist get krbtgt
    ```

    Dieser Befehl sollte zurückgeben, dass ein Ticket für krbtgt erfolgreich abgerufen wurde, und den Domänen Controller auflisten, der zum Abrufen des Tickets verwendet wurde. Wenn Sie ein TGT abrufen können, aber `nltest` aus dem vorherigen Schritt fehlschlägt, kann dies ein Hinweis darauf sein, dass das GMSA-Konto falsch konfiguriert ist. Weitere Informationen finden [Sie unter Überprüfen des GMSA-Kontos](#check-the-gmsa-account) .

    Wenn Sie ein TGT nicht innerhalb des Containers abrufen können, kann dies auf DNS-oder Netzwerkkonnektivitätsprobleme hindeuten. Stellen Sie sicher, dass der Container einen Domänen Controller mit dem DNS-Domänen Namen auflösen kann und dass der Domänen Controller aus dem Container Routing fähig ist.

5. Stellen Sie sicher, dass Ihre APP [für die Verwendung des GMSA konfiguriert](gmsa-configure-app.md)ist. Das Benutzerkonto innerhalb des Containers ändert sich nicht, wenn Sie ein GMSA verwenden. Stattdessen verwendet das System Konto das GMSA bei der Kommunikation mit anderen Netzwerkressourcen. Dies bedeutet, dass Ihre APP als Netzwerkdienst oder lokales System ausgeführt werden muss, um die GMSA-Identität nutzen zu können.

    > [!TIP]
    > Wenn Sie `whoami` ausführen oder ein anderes Tool verwenden, um den aktuellen Benutzer Kontext im Container zu identifizieren, wird der GMSA-Name nicht angezeigt. Dies liegt daran, dass Sie sich immer als lokaler Benutzer anstelle einer Domänen Identität beim Container anmelden. Das GMSA wird vom Computer Konto immer dann verwendet, wenn es mit Netzwerkressourcen kommuniziert, weshalb Ihre APP als Netzwerkdienst oder lokales System ausgeführt werden muss.

### <a name="check-the-gmsa-account"></a>Überprüfen des GMSA-Kontos

1. Wenn Ihr Container anscheinend ordnungsgemäß konfiguriert ist, Benutzer oder andere Dienste sich nicht automatisch bei ihrer containerisierten App authentifizieren können, überprüfen Sie die SPNs in Ihrem GMSA-Konto. Clients finden das GMSA-Konto mit dem Namen, an dem Sie die Anwendung erreichen. Dies bedeutet möglicherweise, dass Sie zusätzliche `host` SPNs für Ihr GMSA benötigen, wenn Clients z. b. über einen Load Balancer oder einen anderen DNS-Namen eine Verbindung mit ihrer App herstellen.

2. Stellen Sie sicher, dass der GMSA-und der Container Host zur gleichen Active Directory Domäne gehören. Der Container Host kann das GMSA-Kennwort nicht abrufen, wenn der GMSA zu einer anderen Domäne gehört.

3. Stellen Sie sicher, dass in Ihrer Domäne nur ein Konto mit dem gleichen Namen wie Ihr GMSA vorhanden ist. an die SAM-Kontonamen von GMSA-Objekten werden Dollarzeichen ($) angehängt, sodass ein GMSA mit dem Namen "MyAccount $" und einem nicht verknüpften Benutzerkonto in derselben Domäne benannt werden kann. Dies kann zu Problemen führen, wenn der Domänen Controller oder die Anwendung nach dem Namen des GMSA suchen muss. Mit dem folgenden Befehl können Sie AD nach ähnlich benannten Objekten durchsuchen:

    ```powershell
    # Replace "GMSANAMEHERE" with your gMSA account name (no trailing dollar sign)
    Get-ADObject -Filter 'sAMAccountName -like "GMSANAMEHERE*"'
    ```

4. Wenn Sie die uneingeschränkte Delegierung für das GMSA-Konto aktiviert haben, stellen Sie sicher, dass für das [userAccountControl-Attribut](https://support.microsoft.com/en-us/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties) weiterhin das `WORKSTATION_TRUST_ACCOUNT`-Flag aktiviert ist. Dieses Flag ist erforderlich, damit die Anmeldung im Container mit dem Domänen Controller kommunizieren kann. Dies ist der Fall, wenn eine APP einen Namen in eine SID auflösen muss oder umgekehrt. Sie können mit den folgenden Befehlen überprüfen, ob das Flag ordnungsgemäß konfiguriert ist:

    ```powershell
    $gMSA = Get-ADServiceAccount -Identity 'yourGmsaName' -Properties UserAccountControl
    ($gMSA.UserAccountControl -band 0x1000) -eq 0x1000
    ```

    Wenn die obigen Befehle `False`zurückgeben, verwenden Sie Folgendes, um der userAccountControl-Eigenschaft des GMSA-Kontos das `WORKSTATION_TRUST_ACCOUNT`-Flag hinzuzufügen. Mit diesem Befehl werden auch die `NORMAL_ACCOUNT`-, `INTERDOMAIN_TRUST_ACCOUNT`-und `SERVER_TRUST_ACCOUNT`-Flags aus der userAccountControl-Eigenschaft gelöscht.

    ```powershell
    Set-ADObject -Identity $gMSA -Replace @{ userAccountControl = ($gmsa.userAccountControl -band 0x7FFFC5FF) -bor 0x1000 }
    ```
