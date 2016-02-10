# Erstellen eigener Integrationsdienste

Ab Windows 10 kann jeder einen Dienst, der den integrierten Hyper-V-Integrationsdiensten ähnelt, mithilfe eines neuen socketbasierten Kommunikationskanals zwischen dem Hyper-V-Host und den darauf ausgeführten virtuellen Computern erstellen. Bei Verwenden dieser Hyper-V-Sockets können Dienste unabhängig vom Netzwerkstapel ausgeführt werden, wobei alle Daten im gleichen physischen Speicher verbleiben.

Dieses Dokument bietet eine exemplarische Vorgehensweise zum Erstellen einer einfachen auf Hyper-V-Sockets basierenden Anwendung und Nutzen dieser Sockets.

[PowerShell Direct](../user_guide/vmsession.md) ist ein Beispiel einer Anwendung (in diesem Fall eines integrierten Windows-Diensts), die Hyper-V-Sockets zum Kommunizieren verwendet.

**Unterstützte Hostbetriebssysteme**
* Windows 10
* Windows Server Technical Preview 3
* Künftige Versionen (Server 2016 +)

**Unterstützte Gastbetriebssysteme**
* Windows 10
* Windows Server Technical Preview 3
* Künftige Versionen (Server 2016 +)

**Stärken und Schwächen**
* Unterstützt den Kernelmodus oder Benutzermodusaktionen
* Nur Datenstrom
* Kein Blockspeicher (für Sicherungen/Video nicht optimal)

--------------


## Erste Schritte

Derzeit sind Hyper-V-Sockets in systemeigenem Code (C/C++) verfügbar.

Zum Schreiben einer einfachen Anwendung benötigen Sie Folgendes:
* C-Compiler. Wenn Sie keinen haben, probieren Sie [Visual Studio Code](https://aka.ms/vs) aus.
* Einen Computer, auf dem Hyper-V und ein virtueller Computer ausgeführt werden.
  * Das Betriebssystem von Host- und Gastcomputer (VM) muss Windows 10, Windows Server Technical Preview 3 oder höher sein.
* Windows SDK: Hier ist ein Link zum [Win10 SDK](https://dev.windows.com/en-us/downloads/windows-10-sdk), das `hvsocket.h` enthält.

## Registrieren einer neuen Anwendung

Damit Sie Hyper-V-Sockets verwenden können, muss die Anwendung in der Registrierung des Hyper-V-Hosts registriert werden.

Durch die Registrierung des Diensts in der Registrierung wird Folgendes verfügbar:
*  WMI-Verwaltung zum Aktivieren, Deaktivieren und Auflisten verfügbarer Dienste
*  Berechtigung für die direkte Kommunikation mit virtuellen Computern

Mithilfe der folgenden PowerShell-Befehle wird eine neue Anwendung mit dem Namen „HV Socket Demo“ registriert. Diese muss als Administrator ausgeführt werden. Anweisungen dazu folgen weiter unten.

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID and add it to the services list then add the name as a value

$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ([System.Guid]::NewGuid().ToString())

$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```

*Registrierungsspeicherort und Informationen*

``` 
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```
An diesem Registrierungsspeicherort sehen Sie mehrere GUIDs. Diese gehören zu unseren Standarddiensten.

Dienstbezogene Informationen in der Registrierung:
* `Dienst-GUID`
    * `ElementName (REG_SZ)`: Dies ist der Anzeigename des Diensts.

Um einen eigenen Dienst zu registrieren, erstellen Sie einen neuen Registrierungsschlüssel mit eigenen Angaben für GUID und Anzeigename.

Der Anzeigename wird Ihrer neuen Anwendung zugeordnet. Er wird in Leistungsindikatoren sowie an anderen Stellen angezeigt, an denen eine GUID nicht geeignet ist.

Der Registrierungseintrag sieht folgendermaßen aus:
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName REG_SZ  VM Session Service
    YourGUID\
        ElementName REG_SZ  Your Service Friendly Name
```

> ** Tipp: ** Führen Sie Folgendes aus, um eine GUID in PowerShell zu generieren und in die Zwischenablage zu kopieren:
``` PowerShell
[System.Guid]::NewGuid().ToString() | clip.exe
```

## Erstellen eines Hyper-V-Sockets

Im einfachsten Fall erfordert die Definition eines Sockets eine Adressfamilie, einen Verbindungstyp und ein Protokoll.

Hier ein einfaches Beispiel: [socket definition](
https://msdn.microsoft.com/en-us/library/windows/desktop/ms740506(v=vs.85).aspx
)

``` C
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);
```

Für ein Hyper-V-Socket:
* Adressfamilie: `AF_HYPERV`
* Typ: `SOCK_STREAM`, `SOCK_DGRAM` oder `SOCK_RAW`
* Protokoll: `HV_PROTOCOL_RAW`


Hier ein Beispiel einer Deklaration/Instanziierung:
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

Da Hyper-V-Sockets nicht von einen Netzwerkstapel, TCP/IP, DNS usw. abhängen, benötigt der Socket-Endpunkt ein Format ohne IP-Adresse und Hostnamen, mit dem die Verbindung dennoch eindeutig beschrieben wird.

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
* VM-ID: Dies ist die eindeutige ID, die VM-bezogen zugewiesen wird. Eine VM-ID kann mit dem folgenden PowerShell-Codeausschnitt gefunden werden.
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* Dienst-ID: [Zuvor beschriebene](#RegisterANewApplication) GUID, mit der die Anwendung in der Registrierung des Hyper-V-Hosts registriert wird.

Es stehen auch verschiedene Platzhalter für VM-IDs zur Verfügung, wenn keine Verbindung mit einem spezifischen virtuellen Computer besteht.

### Platzhalter für VM-IDs

| Name| GUID| Beschreibung|
|:-----|:-----|:-----|
| HV_GUID_ZERO| 00000000-0000-0000-0000-000000000000| Listener müssen eine Bindung mit dieser VM-ID herstellen, um Verbindungen von allen Partitionen zu akzeptieren.|
| HV_GUID_WILDCARD| 00000000-0000-0000-0000-000000000000| Listener müssen eine Bindung mit dieser VM-ID herstellen, um Verbindungen von allen Partitionen zu akzeptieren.|
| HV_GUID_BROADCAST| FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF| |
| HV_GUID_CHILDREN| 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd| Platzhalteradresse für untergeordnete Elemente.Listener müssen eine Bindung mit dieser VM-ID herstellen, um Verbindungen von allen untergeordneten Partitionen zu akzeptieren.|
| HV_GUID_LOOPBACK| e0e16197-dd56-4a10-9195-5ee7a155a838| Loopbackadresse.Bei Verwenden dieser VM-ID wird eine Verbindung mit derselben Partition wie bei Verwenden des Connectors hergestellt.|
| HV_GUID_PARENT| a42e7cda-d03f-480c-9cc2-a4de20abb878| Übergeordnete Adresse.Bei Verwenden dieser VM-ID wird eine Verbindung mit der übergeordneten Partition des Connectors hergestellt.*|


***HV_GUID_PARENT**  
Das übergeordnete Element eines virtuellen Computers ist sein Host. Das übergeordnete Element eines Containers ist der Host des Containers.  
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

[Vollständige WinSock-API](https://msdn.microsoft.com/en-us/library/windows/desktop/ms741394.aspx)

## In Bearbeitung

Graceful disconnect
select




<!--HONumber=Feb16_HO1-->
