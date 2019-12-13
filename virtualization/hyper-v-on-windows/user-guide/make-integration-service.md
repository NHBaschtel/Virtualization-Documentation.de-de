---
title: Erstellen eigener Integrationsdienste
description: Windows 10-Integrationsdienste.
keywords: Windows 10, Hyper-V, HVSocket, AF_HYPERV
author: scooley
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: 1ef8f18c-3d76-4c06-87e4-11d8d4e31aea
ms.openlocfilehash: 89a36ee87bce1da18852f0ebff248e239165eb7d
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74911030"
---
# <a name="make-your-own-integration-services"></a>Erstellen eigener Integrationsdienste

Ab Windows 10 Anniversary Update können Sie selbst Anwendungen erstellen, die zwischen dem Hyper-V-Host und dessen virtuellen Computern kommunizieren, und zwar mithilfe von Hyper-V-Sockets. Das sind Windows-Sockets mit einer neuen Adressfamilie und speziellen Endpunkten für die Auswahl von virtuellen Computern.  Die gesamte Kommunikation über Hyper-V-Sockets erfolgt ohne Networking, und alle Daten verbleiben auf dem gleichen physischen Speicher. Anwendungen, die Hyper-V-Sockets verwenden, ähneln den Hyper-V-Integrationsdiensten.

Dieses Dokument erläutert die Erstellung eines einfachen Programms für Hyper-V-Sockets.

**Unterstützte Hostbetriebssysteme**
* Windows 10 und höher
* Windows Server 2016 und höher

**Unterstützte Gastbetriebssysteme**
* Windows 10 und höher
* Windows Server 2016 und höher
* Linux-Gastcomputer mit Linux Integration Services (siehe [Unterstützte virtuelle Linux- und FreeBSD-Computer für Hyper-V auf Windows](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Linux-and-FreeBSD-virtual-machines-for-Hyper-V-on-Windows))
> **Hinweis:** Ein unterstütztes Linux-Gastbetriebssystem benötigt Kernel-Unterstützung für:
> ```bash
> CONFIG_VSOCKET=y
> CONFIG_HYPERV_VSOCKETS=y
> ```

**Stärken und Schwächen**
* Unterstützt den Kernelmodus oder Benutzermodusaktionen
* Nur Datenstrom
* Kein Blockspeicher (für Sicherungen/Video nicht optimal)

--------------

## <a name="getting-started"></a>Erste Schritte

Anforderungen:
* C/C++-Compiler.  Wenn Sie keinen besitzen, fragen Sie in der [Visual Studio Community](https://aka.ms/vs) nach.
* [Windows 10 SDK](https://developer.microsoft.com/windows/downloads/windows-10-sdk) – vorinstalliert in Visual Studio 2015 mit Update 3 oder höher.
* Ein Computer, auf dem eines der o. g. Host-Betriebssysteme ausgeführt wird, und mindestens ein virtueller Computer (zum Testen der Anwendung).

> **Hinweis:** Die API für Hyper-V-Sockets wurde in Windows 10 Anniversary Update öffentlich verfügbar. Anwendungen, die hvsocket verwenden, können auf jedem Windows 10-Host und-Gast ausgeführt werden, Sie können jedoch nur mit einem Windows SDK später als Build 14290 entwickelt werden.

## <a name="register-a-new-application"></a>Registrieren einer neuen Anwendung
Damit Sie Hyper-V-Sockets verwenden können, muss die Anwendung in der Registrierung des Hyper-V-Hosts registriert werden.

Durch die Registrierung des Diensts in der Registrierung wird Folgendes verfügbar:
*  WMI-Verwaltung zum Aktivieren, Deaktivieren und Auflisten verfügbarer Dienste
*  Berechtigung für die direkte Kommunikation mit virtuellen Computern

Mithilfe der folgenden PowerShell-Befehle wird eine neue Anwendung mit dem Namen „HV Socket Demo“ registriert.  Diese muss als Administrator ausgeführt werden.  Anweisungen dazu folgen weiter unten.

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID.  Add it to the services list
$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ((New-Guid).Guid)

# Set a friendly name
$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```


**Registrierungs Speicherort und-Informationen:**
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```
An diesem Registrierungszweig sehen Sie mehrere GUIDs.  Unsere integrierte Dienste sind.

Dienstbezogene Informationen in der Registrierung:
* `Service GUID`
    * `ElementName (REG_SZ)`: Dies ist der Anzeigename des Diensts.

Um einen eigenen Dienst zu registrieren, erstellen Sie einen neuen Registrierungsschlüssel mit eigenen Angaben für GUID und Anzeigename.

Der Anzeigename wird Ihrer neuen Anwendung zugeordnet.  Er wird in Leistungsindikatoren sowie an anderen Stellen angezeigt, an denen eine GUID nicht geeignet ist.

Der Registrierungseintrag sieht folgendermaßen aus:
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName    REG_SZ    VM Session Service
    YourGUID\
        ElementName    REG_SZ    Your Service Friendly Name
```

> **Hinweis:** Die Dienst-GUID für ein Linux-Gastbetriebssystem verwendet das VSOCK-Protokoll, in über ein `svm_cid` und `svm_port` anstatt eines GUIDs handelt. Um diese Inkonsistenz mit Windows zu überbrücken, wird die bekannte GUID als Dienstvorlage auf dem Host verwendet, der an einen Port auf dem Gast übersetzt wird. Um die die Dienst-GUID anzupassen, ändern Sie einfach die erste "00000000" auf der gewünschten Portnummer. Beispiel: "00000ac9" ist Port 2761.
> ```C++
> // Hyper-V Socket Linux guest VSOCK template GUID
> struct __declspec(uuid("00000000-facb-11e6-bd58-64006a7986d3")) VSockTemplate{};
>
>  /*
>   * GUID example = __uuidof(VSockTemplate);
>   * example.Data1 = 2761; // 0x00000AC9
>   */
> ```
>

> **Tipp:** Mit folgender Anweisung können Sie eine GUID in PowerShell generieren und in die Zwischenablage kopieren:
>``` PowerShell
>(New-Guid).Guid | clip.exe
>```

## <a name="create-a-hyper-v-socket"></a>Erstellen eines Hyper-V-Sockets

Im einfachsten Fall erfordert die Definition eines Sockets eine Adressfamilie, einen Verbindungstyp und ein Protokoll.

Hier sehen Sie eine einfache [Socketdefinition](https://docs.microsoft.com/windows/desktop/api/winsock2/nf-winsock2-socket).

``` C
// Windows
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);

// Linux guest
int socket(int domain, int type, int protocol);
```

Für ein Hyper-V-Socket:
* Adressfamilie: `AF_HYPERV` (Windows) oder `AF_VSOCK` (Linux-Gastbetriebssysteme)
* Typ: `SOCK_STREAM`
* Protokoll – `HV_PROTOCOL_RAW` (Windows) oder `0` (Linux-Gastbetriebssysteme)


Hier sehen Sie eine Beispieldeklaration/-instanziierung:
``` C
// Windows
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);

// Linux guest
int sock = socket(AF_VSOCK, SOCK_STREAM, 0);
```

## <a name="bind-to-a-hyper-v-socket"></a>Herstellen einer Bindung an ein Hyper-V-Socket

Über eine Bindung wird ein Socket mit Verbindungsinformationen verknüpft.

Nachstehend finden Sie die Funktionsdefinition. Weitere Informationen zu Bindungen erhalten Sie [hier](https://docs.microsoft.com/windows/desktop/api/winsock/nf-winsock-bind).

``` C
// Windows
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);

// Linux guest
int bind(int sockfd, const struct sockaddr *addr,
         socklen_t addrlen);
```

Im Gegensatz zur Socketadresse (sockaddr) für eine standardmäßige IP-Adressfamilie (`AF_INET`), die aus der IP-Adresse des Hostcomputers und einer Portnummer auf diesem Host besteht, setzt sich die Socketadresse für `AF_HYPERV` aus der ID des virtuellen Computers und der zuvor definierten Anwendungs-ID zusammen, um eine Verbindung herzustellen. Wenn die Bindung von einem Linux-Gastbetriebssystem geschieht, verwendet `AF_VSOCK``svm_cid` und `svm_port`.

Da Hyper-V-Sockets nicht von einen Netzwerkstapel, TCP/IP, DNS usw. abhängen, benötigt der Socket-Endpunkt ein Format ohne IP-Adresse und Hostnamen, mit dem die Verbindung dennoch eindeutig beschrieben wird.

Hier die Definition der Socketadresse eines Hyper-V-Sockets:

``` C
// Windows
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};

// Linux guest
// See include/uapi/linux/vm_sockets.h for more information.
struct sockaddr_vm {
    __kernel_sa_family_t svm_family;
    unsigned short svm_reserved1;
    unsigned int svm_port;
    unsigned int svm_cid;
    unsigned char svm_zero[sizeof(struct sockaddr) -
                   sizeof(sa_family_t) -
                   sizeof(unsigned short) -
                   sizeof(unsigned int) - sizeof(unsigned int)];
};
```

Anstelle von IP-Adresse oder Hostname arbeiten AF_HYPERV-Endpunkte hauptsächlich mit zwei GUIDs:
* VM-ID: Dies ist die eindeutige ID, die VM-bezogen zugewiesen wird.  Eine VM-ID kann mit dem folgenden PowerShell-Codeausschnitt gefunden werden.
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* Dienst-ID: [Zuvor beschriebene](#register-a-new-application) GUID, mit der die Anwendung in der Registrierung des Hyper-V-Hosts registriert wird.

Es stehen auch verschiedene Platzhalter für VM-IDs zur Verfügung, wenn keine Verbindung mit einem spezifischen virtuellen Computer besteht.

### <a name="vmid-wildcards"></a>Platzhalter für VM-IDs

| Name | GUID | Beschreibung |
|:-----|:-----|:-----|
| HV_GUID_ZERO | 00000000-0000-0000-0000-000000000000 | Listener müssen eine Bindung mit dieser VM-ID herstellen, um Verbindungen von allen Partitionen zu akzeptieren. |
| HV_GUID_WILDCARD | 00000000-0000-0000-0000-000000000000 | Listener müssen eine Bindung mit dieser VM-ID herstellen, um Verbindungen von allen Partitionen zu akzeptieren. |
| HV_GUID_BROADCAST | FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF | |
| HV_GUID_CHILDREN | 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd | Platzhalteradresse für untergeordnete Elemente. Listener müssen eine Bindung mit dieser VM-ID herstellen, um Verbindungen von allen untergeordneten Partitionen zu akzeptieren. |
| HV_GUID_LOOPBACK | e0e16197-dd56-4a10-9195-5ee7a155a838 | Loopbackadresse. Bei Verwenden dieser VM-ID wird eine Verbindung mit derselben Partition wie bei Verwenden des Connectors hergestellt. |
| HV_GUID_PARENT | a42e7cda-d03f-480c-9cc2-a4de20abb878 | Übergeordnete Adresse. Bei Verwenden dieser VmId wird eine Verbindung mit der übergeordneten Partition des Connectors hergestellt.* |


\* `HV_GUID_PARENT` das übergeordnete Element eines virtuellen Computers ist sein Host.  Das übergeordnete Element eines Containers ist der Host des Containers.
Beim Herstellen einer Verbindung mithilfe eines Containers, in dem ein virtueller Computer ausgeführt wird, erfolgt eine Verbindung mit der VM, die den Container hostet.
Beim Überwachen auf diese VM-ID werden Verbindungen von folgenden Quellen akzeptiert: (in Containern): Containerhost.
(Innerhalb der VM: Containerhost/kein Container): VM-Host.
(Außerhalb der VM: Containerhost/kein Container): Nicht unterstützt.

## <a name="supported-socket-commands"></a>Unterstützte Socketbefehle

Socket() Bind() Connect() Send() Listen() Accept()

## <a name="useful-links"></a>Nützliche Links
[Vervollständigen der Winsock-API](https://docs.microsoft.com/windows/desktop/WinSock/winsock-functions)

[Hyper-V-Integration Services Referenz](../reference/integration-services.md)
