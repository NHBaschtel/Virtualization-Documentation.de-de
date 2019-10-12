---
title: Ausführen eines Containers mit einem gMSA
description: Ausführen eines Windows-Containers mit einem Group Managed Service-Konto (gMSA)
keywords: docker, Container, Active Directory, GMSA, Gruppen verwaltetes Dienstkonto, Gruppen verwaltete Dienstkonten
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 52625517748356251aa41115caebd7801ec3cdaf
ms.sourcegitcommit: 22dcc1400dff44fb85591adf0fc443360ea92856
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/12/2019
ms.locfileid: "10209861"
---
# <a name="run-a-container-with-a-gmsa"></a>Ausführen eines Containers mit einem gMSA

Wenn Sie einen Container mit einem Group Managed Service-Konto (gMSA) ausführen möchten, geben Sie die Anmelde `--security-opt` Informationen-Spezifikationsdatei für den Parameter der [Andock](https://docs.docker.com/engine/reference/run)baren Ausführung an:

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

Wenn der Status der vertrauenswürdigen DC-Verbindung und der `NERR_Success`Status der Vertrauensstellung nicht angezeigt wird, folgen Sie den [Anweisungen zur Problembehandlung](gmsa-troubleshooting.md#check-the-container) , um das Problem zu beheben.

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

## <a name="next-steps"></a>Nächste Schritte

Zusätzlich zum Ausführen von Containern können Sie gMSAs auch für folgende Zwecke verwenden:

- [Konfigurieren von apps](gmsa-configure-app.md)
- [Orchestrierungs Container](gmsa-orchestrate-containers.md)

Wenn während des Setups Probleme auftreten, finden Sie in unserem [Leitfaden zur Problembehandlung](gmsa-troubleshooting.md) mögliche Lösungen.
