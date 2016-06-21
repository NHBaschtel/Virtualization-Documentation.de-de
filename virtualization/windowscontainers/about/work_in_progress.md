---
title: Windows-Container – laufende Arbeiten
description: Windows-Container – laufende Arbeiten
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5d9f1cb4-ffb3-433d-8096-b085113a9d7b
---

# Laufende Arbeiten

Falls Ihr Problem hier nicht behandelt wird oder Sie Fragen haben, stellen Sie diese im [Forum](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).

-----------------------

## Allgemeine Funktionalität

### Übereinstimmung der Buildnummer von Container und Host
Ein Windows-Container erfordert ein Betriebssystemimage, das dem Containerhost in Bezug auf Build- und Patchebene entspricht. Eine Nichtübereinstimmung führt möglicherweise zu Instabilität und/oder einem unvorhersehbaren Verhalten von Container und/oder Host.

Wenn Sie Updates für das Betriebssystem des Windows-Containerhosts installieren, müssen Sie das Basisbetriebssystem-Image des Containers mit den entsprechenden Updates aktualisieren.
<!-- Can we give examples of behavior or errors?  Makes it more searchable -->

**Abhilfe:**   
Herunterladen und Installieren eines Container-Basisabbilds Abgleich der Betriebssystemebene und Patchebene des Container-Hosts.

### Standardverhalten der Firewall
In einem Containerhost und in der Containerumgebung gibt es nur die Firewall des Containerhosts. Alle Firewallregeln, die im Containerhost konfiguriert sind, gelten für alle dessen Container.

### Windows-Container starten langsam
Wenn der Start Ihres Containers länger als 30 Sekunden dauert, erfolgen ggf. doppelte Virenscans.

Viele Antischadsoftware-Lösungen, z. B. Windows Defender, scannen vielleicht unnötigerweise Dateien in Containerimages einschließlich aller Betriebssystem-Binärdateien und Dateien im Containerbetriebssystem-Image.  Dies tritt auf, wenn ein neuer Container erstellt wird, und aus der Sicht der Antischadsoftware alle „Dateien des Containers“ wie neue Dateien aussehen, die zuvor noch nicht gescannt wurden.  Also wenn Prozesse innerhalb des Containers versuchen, diese Dateien lesen Scannen Antimalware-Komponenten zuerst diese vor dem Zugriff auf die Dateien.  In Wirklichkeit wurden diese Dateien bereits gescannt, wenn das Container-Bild importiert oder an den Server abgerufen wurde. Ab Windows Server Technical Preview 5 ist die entsprechende Infrastruktur verfügbar, sodass Antischadsoftware-Lösungen wie Windows Defender mit diesen Situationen zurechtkommen und entsprechend agieren, um Mehrfachscans zu vermeiden. Antischadsoftware-Lösungen können die Lösung mit der [hier](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers) beschriebenen Anleitung aktualisieren und Mehrfachscans vermeiden. 

### Starten/Beenden misslingt gelegentlich, wenn der Arbeitsspeicher auf weniger als 48 MB beschränkt ist
Bei Windows-Containern kommt es zu zufälligen Konsistenzfehlern, wenn der Arbeitsspeicher auf weniger als 48 MB beschränkt ist.

Das Ausführen des folgenden PowerShell-Befehls und Wiederholen der Aktionen zum Starten und Beenden verursacht Fehler beim Starten und Beenden.

```PowerShell
new-container "Test" -containerimagename "WindowsServerCore" -MaximumBytes 32MB
start-container test
stop-container test
```

**Abhilfe:**  
Ändern Sie den Wert des Arbeitsspeichers in 48 MB. 


### „Start-Container“ misslingt, wenn auf einer VM mit vier Kernen die Anzahl der Prozessoren auf 1 oder 2 festgelegt ist

Windows-Container kann mit folgendem Fehler nicht gestartet werden:  
`failed to start: This operation returned because the timeout period expired. (0x800705B4).`

Dies tritt auf, wenn auf einer VM mit vier Kernen die Anzahl der Prozessoren auf 1 oder 2 festgelegt ist.

``` PowerShell
new-container "Test2" -containerimagename "WindowsServerCore"
Set-ContainerProcessor -ContainerName test2 -Maximum 2
Start-Container test2

Start-Container : 'test2' failed to start.
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4).
'test2' failed to start. (Container ID 133E9DBB-CA23-4473-B49C-441C60ADCE44)
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4). (Container ID
133E9DBB-CA23-4473-B49C-441C60ADCE44)
At line:1 char:1
+ Start-Container test2
+ ~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationTimeout: (:) [Start-Container], VirtualizationException
    + FullyQualifiedErrorId : OperationTimeout,Microsoft.Containers.PowerShell.Cmdlets.StartContainer
PS C:\> Set-ContainerProcessor -ContainerName test2 -Maximum 3
PS C:\> Start-Container test2
```

**Abhilfe:**  
Erhöhen Sie die Anzahl der für den Container verfügbaren Prozessoren, geben Sie nicht explizit die Anzahl der für den Container verfügbaren Prozessoren an, oder verringern Sie die Anzahl der Prozessoren, die der VM zur Verfügung stehen.

--------------------------

## Netzwerk

### Netzwerkdepot-Isolation und Implikationen
Jeder Container verwendet ein Netzwerkdepot zur Isolation. Alle Containernetzwerkadapter (Endpunkte), die einem bestimmten Container zugeordnet sind, befinden sich im gleichen Netzwerkdepot. Abhängig vom verwendeten Netzwerkmodus (Treiber) können Sie vielleicht nicht mit gleicher IP-Adresse oder gleichem Port auf zwei verschiedene Containerendpunkte zugreifen. Darüber hinaus sind Windows-Firewallregeln nicht depot- oder containerbezogen, sodass alle angewandten Firewallregeln für alle Container des Containerhosts gelten – unabhängig von einem bestimmten Endpunkt.

*** Transparentes Netzwerk ***


***NAT-Netzwerk*** Sie können mehrere Endpunkte, die einem einzelnen Container zugewiesen sind, mit NAT-Regeln zur Portweiterleitung verfügbar machen, die jeweils pro Endpunkt angewendet werden. Diese Weiterleitungsregeln müssen verschiedene externe Ports (auf dem Containerhost) verwenden, wenn sie dem gleichen internen Port (im Container) zugeordnet sind.  Wie bereits erwähnt, haben die angewandten Firewallregeln jedoch auf dem Containerhost globale Gültigkeit.



### Statische NAT-Zuordnungen können zu Konflikten mit Portzuordnungen über Docker führen.
Ab Windows Server Technical Preview 5 sind Regeln zur NAT-Erstellung und Portzuordnung in *ContainerNetwork*-Cmdlets und Docker-Befehle integriert. Der Windows Host Network Service (HNS)verwaltet die NAT in Ihrem Auftrag. Allerdings könnte ein externer Client erfolgreich versuchen, mithilfe der vom HNS erstellten NAT eine doppelte Portzuordnung zu erstellen.


Es folgt ein Beispiel für einen Konflikt mit einer statischen Zuordnung auf Port 80 und dem von Docker in diesem Fall gemeldeten Fehler.
```
C:\Users\Administrator>docker run -it -p 80:80 windowsservercore cmd
docker: Error response from daemon: failed to create endpoint berserk_bassi on network nat: hnsCall failed in Win32: The remote procedure call failed. (0x6be).
```


***Risikominderung*** Im Allgemeinen ist es sehr unwahrscheinlich, dass dies geschieht, da NAT von HNS verwaltet wird. Alle Regeln für die Weiterleitung und Zuordnung von Ports sollten mit `docker run -p <external>:<internal>` oder Add-ContainerNetworkAdapterStaticMapping erstellt werden. Wenn die Zuordnungen jedoch nicht automatisch vom HNS bereinigt werden, kann dieser Fehler durch das Entfernen der Portzuordnung mit PowerShell behoben werden. Dadurch wird der im oben stehenden Beispiel verursachte Port 80-Konflikt beseitigt.
```powershell
Get-NetNatStaticMapping | ? ExternalPort -eq 80 | Remove-NetNatStaticMapping
```


### Windows-Container werden IP-Adressen nicht abrufen.
Wenn Sie über DHCP-VM-Switches eine Verbindung mit den Windows-Containern herstellen, ist es möglich, dass der Containerhost eine IP-Adresse erhält, die Container hingegen nicht.

Die Container erhalten eine 169.254.***.***- APIPA-IP-Adresse.

**Problemumgehung:** Dies ist ein Nebeneffekt der gemeinsamen Kernelnutzung.  Alle Container verfügen effektiv über dieselbe MAC-Adresse.

Aktivieren Sie das MAC-Adressspoofing auf dem physischen Host, der die Containerhost-VM hostet.

Dies kann mithilfe von PowerShell erreicht werden.
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```
--------------------------

## Anwendungskompatibilität

Es gibt zahlreiche Fragen dazu, welche Anwendungen in Windows-Containern funktionieren, weshalb wir uns entschlossen haben, Informationen zur Anwendungskompatibilität in einen [eigenen Artikel](../reference/app_compat.md) auszugliedern.

Nachstehend sind jedoch einige der häufigsten Probleme aufgeführt.

### Ereignisprotokolle sind innerhalb des Containers nicht verfügbar

PowerShell-Befehle wie `get-eventlog Application` und APIs, die das Ereignisprotokoll abfragen, geben etwa folgende Fehlermeldung zurück:
```
get-eventlog : Cannot open log Application on machine .. Windows has not provided an error code.
At line:1 char:1
```

Um dieses Problem zu umgehen, können Sie diesen Schritt einer Dockerfile-Datei hinzufügen. Für mit diesem Schritt erstellte Images ist die Ereignisprotokollierung aktiviert.
```
RUN powershell.exe -command Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\WMI\Autologger\EventLog-Application Start 1
```


### Unerwarteter Fehler in einem Aufruf der API-Methode der „localdb“-Instanz
Unerwarteter Fehler in einem Aufruf der API-Methode der „localdb“-Instanz

### RTerm funktioniert nicht
RTerm wird installiert, aber wird nicht in einem Windows Server-Container gestartet.

Fehler:  
```
'C:\Program' is not recognized as an internal or external command,
operable program or batch file.
```


### Container: Visual C++ Runtime x64/x86 2015 lässt sich nicht installieren.

Beobachtetes Verhalten: In einem Container:
```
C:\build\vcredist_2015_x64.exe /q /norestart
C:\build>echo %errorlevel%
0
C:\build>wmic product get
No Instance(s) Available.
```

Dies ist ein Interoperabilitätsproblem mit dem Deduplizierungsfilter. Die Deduplizierung überprüft das Umbenennungsziel, um zu prüfen, ob eine Datei dedupliziert ist. Die Erstellung misslingt mit `STATUS_IO_REPARSE_TAG_NOT_HANDLED`, da der Filter „Windows Server-Container“ sich über der Deduplizierung befindet.


Im Artikel zur [Anwendungskompatibilität](../reference/app_compat.md) finden Sie weitere Informationen dazu, für welche Anwendungen Container verwendet werden können.

--------------------------


## Docker-Verwaltung

### Nicht alle Docker-Befehle funktionieren
* „docker exec“ funktioniert in Hyper-V-Containern nicht.

Wenn Fehler bei Vorgängen auftreten, die nicht in dieser Liste enthalten sind (oder bei einem Befehl ein anderer Fehler als erwartet auftritt), lassen Sie es uns in den [Foren](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers) wissen.

### Das Einfügen von Befehlen in eine interaktive Docker-Sitzung ist auf 50 Zeichen begrenzt
Das Einfügen von Befehlen in eine interaktive Docker-Sitzung ist auf 50 Zeichen begrenzt.  
Wenn Sie eine Befehlszeile in eine interaktive Docker-Sitzung kopieren, gilt ein Limit von 50 Zeichen. Die eingefügte Zeichenfolge wird einfach abgeschnitten.

Dies ist nicht beabsichtigt. Wir arbeiten an der Aufhebung der Einschränkung.

### „net use“-Fehler
„net use“ gibt den Systemfehler 1223 zurück, anstatt den Benutzernamen oder das Kennwort anzufordern.

**Abhilfe:**  
Geben Sie bei Ausführen von „net use“ Benutzername und Kennwort an.

``` PowerShell
net use S: \\your\sources\here /User:shareuser [yourpassword]
``` 


--------------------------



## Remotedesktop 

Es ist in einer RDP-Sitzung in TP5 nicht möglich, Windows-Container zu verwalten bzw. mit ihnen zu interagieren.

--------------------------

### Das Beenden eines Containers in einem Nano Server-Containerhost ist nicht über „exit“ möglich
Wenn Sie versuchen, einen Container zu beenden, der sich in einem Nano Server-Containerhost befindet, werden Sie durch den Befehl „exit“ vom Nano Server-Containerhost getrennt, und der Container wird nicht beendet.

**Problemumgehung:** Verwenden Sie stattdessen Exit-PSSession, um den Container zu beenden.

Wenn Sie bestimmte Features wünschen, teilen Sie uns dies in den [Foren](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers) mit. 


--------------------------


## Benutzer und Domänen

### Lokale Benutzer
Lokale Benutzerkonten können erstellt und für die Ausführung von Windows-Diensten und -Anwendungen in Containern verwendet werden.


### Domänenmitgliedschaft
Container können keinen Active Directory-Domänen beitreten und Dienste oder Anwendungen nicht als Domänenbenutzer, Dienstkonten oder Computerkonten ausführen. 

Container sind so konzipiert, dass sie schnell in einem bekannten konsistenten Status gestartet werden, der von der Umgebung weitestgehend unabhängig ist. Durch den Beitritt zu einer Domäne und die Anwendung von Gruppenrichtlinieneinstellungen aus der Domäne würde die zum Starten eines Containers erforderliche Zeit verlängert, die Funktion des Containers mit der Zeit verändert und die Möglichkeit zum Verschieben oder Freigeben von Bildern zwischen Entwicklern und über Bereitstellungen hinweg begrenzt.

Wir prüfen sorgfältig das Feedback dazu, wie Dienste und Anwendungen Active Directory einsetzen, sowie die Auswirkungen von deren Bereitstellung in Containern. Wenn Sie Details zu den von Ihnen bevorzugten Funktionen bereitstellen möchten, teilen Sie sie uns in den [Foren](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers) mit. 

Wir suchen aktiv nach Lösungen für die Unterstützung solcher Szenarien.


<!--HONumber=May16_HO4-->


