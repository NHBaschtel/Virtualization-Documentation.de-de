---
title: Erstellen eigener Integrationsdienste
description: Windows 10-Integrationsdienste.
keywords: "Windows 10, Hyper-V, HVSocket, AF_HYPERV"
author: scooley
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ef8f18c-3d76-4c06-87e4-11d8d4e31aea
translationtype: Human Translation
ms.sourcegitcommit: b6b63318ed71931c2b49039e57685414f869a945
ms.openlocfilehash: 19e8cf269b0bef127fb06d2c99391107cd8683b1
ms.lasthandoff: 02/16/2017

---

# Erstellen eigener Integrationsdienste

Ab Windows 10 Anniversary Update können Sie selbst Anwendungen erstellen, die zwischen dem Hyper-V-Host und dessen virtuellen Computern kommunizieren, und zwar mithilfe von Hyper-V-Sockets. Das sind Windows-Sockets mit einer neuen Adressfamilie und speziellen Endpunkten für die Auswahl von virtuellen Computern.  Die gesamte Kommunikation über Hyper-V-Sockets erfolgt ohne Networking, und alle Daten verbleiben auf dem gleichen physischen Speicher.   Anwendungen, die Hyper-V-Sockets verwenden, ähneln den Hyper-V-Integrationsdiensten.

Dieses Dokument erläutert die Erstellung eines einfachen Programms für Hyper-V-Sockets.

**Unterstützte Host-Betriebssysteme**
* Unter Windows 10 unterstützt
* Windows Server 2016
* Künftige Versionen (Server 2016 +)

**Unterstützte Gastbetriebssysteme**
* Windows 10
* Windows Server Technical Preview 4 und höher
* Künftige Versionen (Server 2016 +)
* Linux-Gastcomputer mit Linux Integration Services (siehe [Unterstützte virtuelle Linux- und FreeBSD-Computer für Hyper-V auf Windows](https://technet.microsoft.com/library/dn531030(ws.12).aspx))

**Stärken und Schwächen**  
* Unterstützt den Kernelmodus oder Benutzermodusaktionen  
* Nur Datenstrom      
* Kein Blockspeicher (für Sicherungen/Video nicht optimal) 

--------------

## Erste Schritte

Anforderungen:
* C/C++-Compiler.  Wenn Sie keinen besitzen, fragen Sie in der [Visual Studio Community](https://aka.ms/vs) nach.
* [Windows 10 SDK](https://developer.microsoft.com/windows/downloads/windows-10-sdk) – vorinstalliert in Visual Studio 2015 mit Update 3 oder höher.
* Ein Computer, auf dem eines der o. g. Host-Betriebssysteme ausgeführt wird, und mindestens ein virtueller Computer (zum Testen der Anwendung).

> **Hinweis:** Die API für Hyper-V-Sockets war etwas später in Windows 10 öffentlich verfügbar.  Anwendungen, die HVSocket verwenden, sind auf jedem Host und Gast unter Windows 10 ausführbar, können jedoch nur einem Windows SDK ab Build 14290 entwickelt werden.  

## Registrieren einer neuen Anwendung
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


**Registrierungsschlüssel und Informationen:**  
``` 
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```  
An diesem Registrierungszweig sehen Sie mehrere GUIDs.  Diese gehören zu unseren Standarddiensten.

Die Informationen in der Registrierung nach Dienst:
* `Service GUID`   
    * `ElementName (REG_SZ)` : Dies ist der Anzeigename des Diensts.

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

> **Tipp:** Mit folgender Anweisung können Sie eine GUID in PowerShell generieren und in die Zwischenablage kopieren:  
``` PowerShell
(New-Guid).Guid | clip.exe
```

## Erstellen eines Hyper-V-Sockets

Im einfachsten Fall erfordert die Definition eines Sockets eine Adressfamilie, einen Verbindungstyp und ein Protokoll.

Hier sehen Sie eine einfache [Socketdefinition](
https://msdn.microsoft.com/en-us/library/windows/desktop/ms740506(v=vs.85).aspx
).

``` C
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);
``` 

Für ein Hyper-V-Socket:
* Adressfamilie: `AF_HYPERV`
* Typ: `SOCK_STREAM`
* Protokoll: `HV_PROTOCOL_RAW`


Hier sehen Sie eine Beispieldeklaration/-instanziierung:  
``` C
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);
```


## Herstellen einer Bindung an ein Hyper-V-Socket

Über eine Bindung wird ein Socket mit Verbindungsinformationen verknüpft.

Nachstehend finden Sie die Funktionsdefinition. Weitere Informationen zu Bindungen erhalten Sie [hier](https://msdn.microsoft.com/en-us/library/windows/desktop/ms737550.aspx).

``` C
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);
```

Im Gegensatz zur Socketadresse (sockaddr) für eine standardmäßige IP-Adressfamilie (`AF_INET`), die aus der IP-Adresse des Hostcomputers und einer Portnummer auf diesem Host besteht, setzt sich die Socketadresse für `AF_HYPERV` aus der ID des virtuellen Computers und der zuvor definierten Anwendungs-ID zusammen, um eine Verbindung herzustellen. 

Da Hyper-V-Sockets nicht von einem Netzwerkstapel, TCP/IP, DNS usw. abhängen, benötigt der Socket-Endpunkt ein Format ohne IP-Adresse und Hostnamen, mit dem die Verbindung dennoch eindeutig beschrieben wird.

Hier die Definition der Socketadresse eines Hyper-V-Sockets:

``` C
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};
```

Anstelle von IP-Adresse oder Hostname arbeiten AF_HYPERV-Endpunkte hauptsächlich mit zwei GUIDs:  
* VM-ID – ist dies die eindeutige ID, die pro virtueller Maschine zugewiesen.  Eine VM-ID kann mit dem folgenden PowerShell-Ausschnitt gefunden werden.  
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* Dienst-ID: [Zuvor beschriebene](#RegisterANewApplication) GUID, mit der die Anwendung in der Registrierung des Hyper-V-Hosts registriert wird.

Es stehen auch verschiedene Platzhalter für VM-IDs zur Verfügung, wenn keine Verbindung mit einem spezifischen virtuellen Computer besteht.
 
### Platzhalter für VM-IDs

| Name | GUID | Beschreibung |
|:-----|:-----|:-----|
| HV_GUID_ZERO | 00000000-0000-0000-0000-000000000000 | Listener müssen eine Bindung mit dieser VM-ID herstellen, um Verbindungen von allen Partitionen zu akzeptieren. |
| HV_GUID_WILDCARD | 00000000-0000-0000-0000-000000000000 | Listener müssen eine Bindung mit dieser VM-ID herstellen, um Verbindungen von allen Partitionen zu akzeptieren. |
| HV_GUID_BROADCAST | FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF | |  
| HV_GUID_CHILDREN | 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd | Platzhalteradresse für untergeordnete Elemente. Listener müssen eine Bindung mit dieser VM-ID herstellen, um Verbindungen von allen untergeordneten Partitionen zu akzeptieren. |
| HV_GUID_LOOPBACK | e0e16197-dd56-4a10-9195-5ee7a155a838 | Loopbackadresse. Bei Verwenden dieser VM-ID wird eine Verbindung mit derselben Partition wie bei Verwenden des Connectors hergestellt. |
| HV_GUID_PARENT | a42e7cda-d03f-480c-9cc2-a4de20abb878 | Übergeordnete Adresse. Bei Verwenden dieser VmId wird eine Verbindung mit der übergeordneten Partition des Connectors hergestellt.* |


\* `HV_GUID_PARENT`  
Das übergeordnete Element eines virtuellen Computers ist sein Host.  Das übergeordnete Element eines Containers ist der Host des Containers.  
Beim Herstellen einer Verbindung mithilfe eines Containers, in dem ein virtueller Computer ausgeführt wird, erfolgt eine Verbindung mit der VM, die den Container hostet.  
Beim Überwachen auf diese VM-ID werden Verbindungen von folgenden Quellen akzeptiert:  
(Innerhalb von Containern): Containerhost.  
(Innerhalb der VM: Containerhost/kein Container): VM-Host.  
(Außerhalb der VM: Containerhost/kein Container): Nicht unterstützt.

## Unterstützte Socketbefehle

Socket()  
Bind()  
Connect()  
Send()  
Listen()  
Accept()  

## Nützliche Links
[Vollständige WinSock-API](https://msdn.microsoft.com/en-us/library/windows/desktop/ms741394.aspx)

[Referenz für Hyper-V-Integrationsdienste](../reference/integration-services.md)
