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

### Windows-Container erhalten keine IP-Adressen

Wenn Sie über DHCP-VM-Switches eine Verbindung mit den Windows-Containern herstellen, ist es möglich, dass der Containerhost eine IP-Adresse erhält, die Container hingegen nicht.

Die Container erhalten eine von APIPA zugewiesene IP-Adresse im Bereich „169.254.***.***“.

**Abhilfe:**
Dies ist ein Nebeneffekt der gemeinsamen Kernelnutzung. Alle Container verfügen eigentlich über dieselbe MAC-Adresse.

Aktivieren Sie das Spoofing von MAC-Adressen auf dem Containerhost.

Dies kann mithilfe von PowerShell erreicht werden.
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```

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


Im [Artikel zur Anwendungskompatibilität](../reference/app_compat.md) finden Sie weitere Informationen dazu, welche Anwendungen containerisiert werden können.

--------------------------



## Docker-Verwaltung

### Docker-Clients ungeschützt

In dieser Vorabversion ist die Docker-Kommunikation öffentlich (wenn Sie wissen, wo Sie suchen sollen).

### Nicht alle Docker-Befehle funktionieren

„docker exec“ funktioniert in Hyper-V-Containern nicht.

Befehle im Zusammenhang mit DockerHub werden noch nicht unterstützt.

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


## Remotedesktop

Es ist in einer RDP-Sitzung in TP4 nicht möglich, Windows-Container zu verwalten bzw. mit ihnen zu interagieren.

--------------------------


## PowerShell-Verwaltung

### Nicht alle „*-PSSession“-Befehle haben ein „containerid“-Argument

Dies ist korrekt. Wir planen die vollständige Unterstützung von „cimsession“ in der Zukunft.

Wenn Sie bestimmte Features wünschen, teilen Sie uns dies in den [Foren](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers) mit.




