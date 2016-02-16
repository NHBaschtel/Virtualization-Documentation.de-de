# Laufende Arbeiten

Falls Ihr Problem hier nicht behandelt wird oder Sie Fragen haben, stellen Sie diese im [Forum](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers).

-----------------------


## Allgemeine Funktionalität

### Übereinstimmung der Buildnummer von Container und Host

Ein Windows-Container erfordert ein Betriebssystemimage, das dem Containerhost in Bezug auf Build- und Patchebene entspricht. Eine Nichtübereinstimmung führt möglicherweise zu Instabilität und/oder einem unvorhersehbaren Verhalten von Container und/oder Host.

Wenn Sie Updates für das Betriebssystem des Windows-Containerhosts installieren, müssen Sie das Basisbetriebssystem-Image des Containers mit den entsprechenden Updates aktualisieren.


**Abhilfe:**   
Laden Sie ein Containerbasis-Image herunter, das der Betriebssystemversion und Patchebene des Containerhosts entspricht.

### Alle Laufwerke außer C:/ werden in Containern angezeigt

Alle Laufwerke außer C:/, die dem Container zur Verfügung stehen, werden automatisch in neu ausgeführten Windows-Containern zugeordnet.

Derzeit besteht keine Möglichkeit, Ordner in einem Container selektiv zuzuordnen. Als Zwischenlösung werden Laufwerke automatisch zugeordnet.

*Abhilfe:*  
Wir arbeiten daran. Künftig wird es Ordnerfreigaben geben.

### Standardverhalten der Firewall

In einem Containerhost und in der Containerumgebung gibt es nur die Firewall des Containerhosts. Alle Firewallregeln, die im Containerhost konfiguriert sind, gelten für alle dessen Container.

### Windows-Container starten langsam

Wenn der Start Ihres Containers länger als 30 Sekunden dauert, erfolgen ggf. doppelte Virenscans.

Viele Antischadsoftwarelösungen, wie z. B. Windows Defender, scannen ggf. unnötigerweise Daten innerhalb von Containerimages, einschließlich aller Binärdateien des Betriebssystems und Dateien im Containerbetriebssystem-Image. Dieser Fehler tritt immer dann auf, wenn ein neuer Container erstellt wird und aus Sicht der Antischadsoftware alle „Dateien im Container“ wie neue Dateien aussehen, die noch nicht gescannt wurden. Wenn dann Prozesse innerhalb des Containers versuchen, diese Dateien zu lesen, werden sie zuerst von den Komponenten der Antischadsoftware gescannt, ehe der Zugriff auf die Dateien erlaubt wird. Tatsächlich wurden diese Dateien bereits gescannt, als das Containerimage auf den Server importiert oder übertragen wurde. In künftigen Vorschauversionen wird es neue Infrastruktur geben, sodass Antischadsoftwarelösungen wie Windows Defender mit diesen Situationen zurechtkommen und entsprechend agieren, um Mehrfachscans zu vermeiden.

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
Fehler beim Starten: Dieser Vorgang wurde wegen Zeitüberschreitung zurückgegeben. (0x800705B4).

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

### Begrenzte Netzwerkdepots

In dieser Version unterstützt wir ein Netzwerkdepot pro Container. Dies bedeutet, dass wenn Sie einen Container mit mehreren Netzwerkadaptern haben, Sie nicht auf jedem Adapter auf denselben Port zugreifen können (wenn z. B. 192.168.0.1:80 und 192.168.0.2:80 zum selben Container gehören).

*Abhilfe:*  
Wenn mehrere Endpunkte einem Container zur Verfügung gestellt werden müssen, verwenden Sie die NAT-Portzuordnung.


### Statische NAT-Zuordnungen können mit Portzuordnungen über Docker in Konflikt stehen.

Wenn Sie Container mithilfe von Windows PowerShell und durch Hinzufügen von statischen NAT-Zuordnungen erstellen, können diese Konflikte verursachen, wenn Sie sie nicht vor dem Starten eines Containers mit `docker -p &lt;Quelle&gt;:&lt;Ziel&gt;` entfernen

Nachfolgend finden Sie ein Beispiel für einen Konflikt mit einer statischen Zuordnung an Port 80
```
PS C:\IISDemo> Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress
 172.16.0.2 -InternalPort 80 -ExternalPort 80


StaticMappingID               : 1
NatName                       : ContainerNat
Protocol                      : TCP
RemoteExternalIPAddressPrefix : 0.0.0.0/0
ExternalIPAddress             : 0.0.0.0
ExternalPort                  : 80
InternalIPAddress             : 172.16.0.2
InternalPort                  : 80
InternalRoutingDomainId       : {00000000-0000-0000-0000-000000000000}
Active                        : True



PS C:\IISDemo> docker run -it -p 80:80 microsoft/iis cmd
docker: Error response from daemon: Cannot start container 30b17cbe85539f08282340cc01f2797b42517924a70c8133f9d8db83707a2c66: 
HCSShim::CreateComputeSystem - Win32 API call returned error r1=2147942452 err=You were not connected because a 
duplicate name exists on the network. If joining a domain, go to System in Control Panel to change the computer name
 and try again. If joining a workgroup, choose another workgroup name. 
 id=30b17cbe85539f08282340cc01f2797b42517924a70c8133f9d8db83707a2c66 configuration= {"SystemType":"Container",
 "Name":"30b17cbe85539f08282340cc01f2797b42517924a70c8133f9d8db83707a2c66","Owner":"docker","IsDummy":false,
 "VolumePath":"\\\\?\\Volume{4b239270-c94f-11e5-a4c6-00155d016f0a}","Devices":[{"DeviceType":"Network","Connection":
 {"NetworkName":"Virtual Switch","EnableNat":false,"Nat":{"Name":"ContainerNAT","PortBindings":[{"Protocol":"TCP",
 InternalPort":80,"ExternalPort":80}]}},"Settings":null}],"IgnoreFlushesDuringBoot":true,
 "LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\30b17cbe85539f08282340cc01f2797b42517924a70c8133f9d8db83707a2c66",
 "Layers":[{"ID":"4b91d267-ecbc-53fa-8392-62ac73812c7b","Path":"C:\\ProgramData\\docker\\windowsfilter\\39b8f98ccaf1ed6ae267fa3e98edcfe5e8e0d5414c306f6c6bb1740816e536fb"},
 {"ID":"ff42c322-58f2-5dbe-86a0-8104fcb55c2a",
"Path":"C:\\ProgramData\\docker\\windowsfilter\\6a182c7eba7e87f917f4806f53b2a7827d2ff0c8a22d200706cd279025f830f5"},
{"ID":"84ea5d62-64ed-574d-a0b6-2d19ec831a27",
"Path":"C:\\ProgramData\\Microsoft\\Windows\\Images\\CN=Microsoft_WindowsServerCore_10.0.10586.0"}],
"HostName":"30b17cbe8553","MappedDirectories":[],"SandboxPath":"","HvPartition":false}.
```


***Lösung***
Der Konflikt kann durch Entfernen der Portzuordnung mithilfe von PowerShell behoben werden. Dadurch wird der im oben stehenden Beispiel verursachte Port 80-Konflikt beseitigt.
```powershell
Get-NetNatStaticMapping | ? ExternalPort -eq 80 | Remove-NetNatStaticMapping
```


### Windows-Container erhalten keine IP-Adressen

Wenn Sie über DHCP-VM-Switches eine Verbindung mit den Windows-Containern herstellen, ist es möglich, dass der Containerhost eine IP-Adresse erhält, die Container hingegen nicht.

Die Container erhalten eine von APIPA zugewiesene IP-Adresse im Bereich „169.254.***.***“.

**Abhilfe:**
Dies ist ein Nebeneffekt der gemeinsamen Kernelnutzung. Alle Container verfügen effektiv über dieselbe MAC-Adresse.

Aktivieren Sie das Spoofing von MAC-Adressen auf dem Containerhost.

Dies kann mithilfe von PowerShell erreicht werden.
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### HTTPS und TLS werden nicht unterstützt.

Windows Server-Container und Hyper-V-Container unterstützen weder HTTPS noch TLS. Wir arbeiten daran, diese Protokolle in Zukunft verfügbar zu machen.

--------------------------


## Anwendungskompatibilität

Es gibt zahlreiche Fragen dazu, welche Anwendungen in Windows-Containern funktionieren, weshalb wir uns entschlossen haben, Informationen zur Anwendungskompatibilität in einen [eigenen Artikel](../reference/app_compat.md) auszugliedern.

Nachstehend sind jedoch einige der häufigsten Probleme aufgeführt.

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

Beobachtetes Verhalten:
In einem Container:
```
C:\build\vcredist_2015_x64.exe /q /norestart
C:\build>echo %errorlevel%
0
C:\build>wmic product get
No Instance(s) Available.
```

Dies ist ein Interoperabilitätsproblem mit dem Deduplizierungsfilter. Die Deduplizierung überprüft das Umbenennungsziel, um zu prüfen, ob eine Datei dedupliziert ist. Die Erstellung misslingt mit `STATUS_IO_REPARSE_TAG_NOT_HANDLED`, da der Filter „Windows Server-Container“ sich über der Deduplizierung befindet.


Im [Artikel zur Anwendungskompatibilität](../reference/app_compat.md) finden Sie weitere Informationen dazu, für welche Anwendungen Container verwendet werden können.

--------------------------



## Docker-Verwaltung

### Docker-Clients ungeschützt

In dieser Vorabversion ist die Docker-Kommunikation öffentlich (wenn Sie wissen, wo Sie suchen sollen).

### Nicht alle Docker-Befehle funktionieren

* „docker exec“ funktioniert in Hyper-V-Containern nicht.
* Befehle im Zusammenhang mit DockerHub werden noch nicht unterstützt.

Wenn Fehler bei Vorgängen auftreten, die nicht in dieser Liste enthalten sind (oder ein Befehl anders als erwartet misslingt), lassen Sie es uns in den [Foren](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers) wissen.

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

Es ist in einer RDP-Sitzung in TP4 nicht möglich, Windows-Container zu verwalten bzw. mit ihnen zu interagieren.

--------------------------


## PowerShell-Verwaltung

### Nicht alle „*-PSSession“-Befehle haben ein „containerid“-Argument

Dies ist korrekt. Wir planen die vollständige Unterstützung von „cimsession“ in der Zukunft.

### Das Beenden eines Containers in einem Nano Server-Containerhost ist nicht über „exit“ möglich

Wenn Sie versuchen, einen Container zu beenden, der sich in einem Nano Server-Containerhost befindet, werden Sie durch den Befehl „exit“ vom Nano Server-Containerhost getrennt, und der Container wird nicht beendet.

**Abhilfe:**
Verwenden Sie stattdessen „Exit-PSSession“, um den Container zu beenden.

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




<!--HONumber=Feb16_HO1-->
