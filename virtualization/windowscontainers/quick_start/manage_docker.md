



# Schnellstartanleitung: Windows-Container und Docker

Mithilfe von Windows-Containern können viele isolierte Anwendungen schnell auf einem einzelnen Computersystem bereitgestellt werden. In dieser Übung wird das Erstellen und Verwalten von Windows-Containern mit Docker demonstriert. Im Anschluss sollten Sie grundlegend damit vertraut sein, wie Docker und Windows-Container integriert sind, nachdem Sie praktische Erfahrung mit dieser Technologie gesammelt haben.

In dieser exemplarischen Vorgehensweise werden sowohl Windows Server- als auch Hyper-V-Container detailliert behandelt. Jeder Containertyp hat eigene Grundanforderungen. In der Dokumentation zu Windows-Containern wird ein Verfahren für die schnelle Bereitstellung eines Containerhosts beschrieben. Dies ist die einfachste Möglichkeit, schnell mit Windows-Containern zu starten. Wenn Sie noch keinen Containerhost haben, lesen Sie die [Schnellstartanleitung zur Bereitstellung von Containerhosts](./container_setup.md).

Die folgenden Elemente werden für jede Übung benötigt.

**Windows Server-Container:**

- Ein Windows-Containerhost unter Windows Server 2016 (Vollversion oder Core), entweder lokal oder in Azure.

**Hyper-V-Container:**

- Ein Windows-Containerhost mit aktivierter geschachtelter Virtualisierung.
- Das Windows Server 2016-Installationsmedium: [Herunterladen](https://aka.ms/tp4/serveriso).

> Microsoft Azure unterstützt keine Hyper-V-Container. Für die Übungen mit Hyper-V-Containern benötigen Sie einen lokalen Containerhost.

## Windows Server-Container

Windows Server-Container bieten eine isolierte, portierbare Betriebsumgebung mit Ressourcenkontrolle für das Ausführen von Anwendungen und Hosten von Prozessen. Windows Server-Container bieten zwischen Containern und Host eine Isolation, indem Prozesse und Namespaces voneinander getrennt werden.

### Erstellen eines Containers

Rufen Sie vor dem Erstellen eines Containers den Befehl `docker images` auf, um die auf dem Host installierten Containerimages aufzulisten.

```powershell
PS C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
nanoserver          10.0.10586.0        8572198a60f1        2 weeks ago         0 B
nanoserver          latest              8572198a60f1        2 weeks ago         0 B
```

In diesem Beispiel erstellen Sie einen Container mit dem Windows Server Core-Image. Dies erfolgt mit dem Befehl `docker run`. Weitere Informationen zu `docker run` finden Sie in der [Referenz zu „Docker Run“ auf docker.com](https://docs.docker.com/engine/reference/run/).

Dieses Beispiel erstellt einen Container namens `iisbase` und startet eine interaktive Sitzung mit dem Container.

```powershell
C:\> docker run --name iisbase -it windowsservercore cmd
```

Nachdem der Container erstellt wurde, arbeiten Sie in einer Shellsitzung innerhalb des Containers.


### Erstellen des IIS-Images

IIS wird installiert. Anschließend wird ein Image anhand des Containers erstellt. Führen Sie folgende Schritte aus, um IIS zu installieren.

```powershell
C:\> powershell.exe Install-WindowsFeature web-server
```

Beenden Sie anschließend die interaktive Shellsitzung.

```powershell
C:\> exit
```

Abschließend wird für den Container mithilfe von `docker commit` ein Commit in ein neues Containerimage ausgeführt. Dieses Beispiel erstellt ein neues Containerimage mit dem Namen `windowsservercoreiis`.

```powershell
C:\> docker commit iisbase windowsservercoreiis
4193c9f34e320c4e2c52ec52550df225b2243927ed21f014fbfff3f29474b090
```

Die neuen IIS-Images können mithilfe des Befehls `docker images` angezeigt werden.

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
windowsservercoreiis   latest              4193c9f34e32        4 minutes ago       170.8 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore      latest              6801d964fda5        2 weeks ago         0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago         0 B
nanoserver             latest              8572198a60f1        2 weeks ago         0 B
```

### Konfigurieren des Netzwerks

Vor dem Erstellen eines Containers mit Docker muss eine Regel für die Windows-Firewall erstellt werden, die eine Netzwerkverbindung mit dem Container zulässt. Führen Sie Folgendes aus, um eine Regel für Port 80 zu erstellen.

```powershell
powershell.exe "if(!(Get-NetFirewallRule | where {$_.Name -eq 'TCP80'})) { New-NetFirewallRule -Name 'TCP80' -DisplayName 'HTTP on TCP/80' -Protocol tcp -LocalPort 80 -Action Allow -Enabled True }" 
```

Sie sollten sich auch die IP-Adresse des Containerhosts notieren, da sie in der gesamten Übung verwendet wird.

### Erstellen des IIS-Containers

Sie haben nun ein Container-Image, das IIS enthält und verwendet werden kann, um IIS-fähige Betriebsumgebungen bereitzustellen.

Verwenden Sie zum Erstellen eines Containers aus dem neuen Image den Befehl `docker run`, wobei Sie diesmal den Namen des IIS-Images angeben. Beachten Sie, dass in diesem Beispiel der Parameter `-p 80:80` angegeben ist. Da der Container mit einem virtuellen Switch verbunden ist, der IP-Adressen über NAT (Netzwerkadressübersetzung) bereitstellt, muss ein Port auf dem Containerhost einem Port in der NAT-IP-Adresse des Containers zugeordnet werden. Weitere Informationen zum Parameter `-p` finden Sie in der [Referenz zu „Docker Run“ auf docker.com](https://docs.docker.com/engine/reference/run/).

```powershell
C:\> docker run --name iisdemo -it -p 80:80 windowsservercoreiis cmd
```

Nachdem der Container erstellt wurde, öffnen Sie einen Browser und navigieren zur IP-Adresse des Containerhosts. Da Port 80 auf dem Host Port 80 im Container zugeordnet ist, sollte der IIS-Begrüßungsbildschirm angezeigt werden.

![](media/iis1.png)

### Erstellen einer Anwendung

Führen Sie zum Entfernen des IIS-Begrüßungsbildschirms den folgenden Befehl aus.

```powershell
C:\> del C:\inetpub\wwwroot\iisstart.htm
```

Führen Sie den folgenden Befehl aus, um die IIS-Standardwebsite durch eine neue statische Website zu ersetzen.

```powershell
C:\> echo "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

Navigieren Sie erneut zur IP-Adresse des Containerhosts. Jetzt sollte die „Hello World“-Anwendung angezeigt werden. Hinweis: Möglicherweise müssen Sie alle vorhandenen Browserverbindungen schließen oder den Browsercache leeren, um die aktualisierte Anwendung anzuzeigen.

![](media/HWWINServer.png)

Beenden Sie die interaktive Sitzung mit dem Container.

```powershell
C:\> exit
```

Entfernen des Containers

```powershell
C:\> docker rm iisdemo
```
Entfernen Sie das IIS-Image.

```powershell
C:\> docker rmi windowsservercoreiis
```

## Dockerfile

In der letzten Übung wurde ein Container manuell erstellt, geändert und dann in einem neuen Container-Image erfasst. Docker bietet eine Methode für die Automatisierung dieses Prozesses, die als Dockerfile bezeichnet wird. Diese Übung liefert dieselben Ergebnisse wie die letzte, doch diesmal ist der Prozess vollständig automatisiert.

### Erstellen des IIS-Images

Erstellen Sie auf dem Containerhost das Verzeichnis `c:\build` und in diesem Verzeichnis eine Datei namens `dockerfile`.

```powershell
C:\> powershell new-item c:\build\dockerfile -Force
```

Öffnen Sie die Dockerfile in Editor.

```powershell
C:\> notepad c:\build\dockerfile
```

Kopieren Sie den folgenden Text in die Dockerfile, und speichern Sie sie. Diese Befehle weisen Docker an, ein neues Image mit `windowsservercore` als Basis zu erstellen und die mit `RUN` angegebenen Änderungen einzuschließen. Weitere Informationen zu „Dockerfiles“ finden Sie in der [Referenz zu Dockerfiles auf docker.com](http://docs.docker.com/engine/reference/builder/).

```powershell
FROM windowsservercore
RUN dism /online /enable-feature /all /featurename:iis-webserver /NoRestart
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

Mit diesem Befehl wird der automatische Buildprozess gestartet. Der Parameter `-t` weist den Prozess an, das neue Image `iis` zu nennen.

```powershell
C:\> docker build -t iis c:\Build
```

Abschließend können Sie mithilfe des Befehls `docker images` prüfen, ob das Image erstellt wurde.

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
iis                 latest              abb93867b6f4        26 seconds ago      209 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
nanoserver          10.0.10586.0        8572198a60f1        2 weeks ago         0 B
nanoserver          latest              8572198a60f1        2 weeks ago         0 B
```

### Bereitstellen des IIS-Containers

Stellen Sie wie in der letzten Übung nun den Container bereit, und weisen Sie Port 80 auf dem Host Port 80 des Containers zu.

```powershell
C:\> docker run --name iisdemo -it -p 80:80 iis cmd
```

Nachdem der Container erstellt wurde, navigieren Sie zur IP-Adresse des Containerhosts. Die „Hello World“-Anwendung sollte angezeigt werden.

![](media/dockerfile2.png)

Beenden Sie die interaktive Sitzung mit dem Container.

```powershell
C:\> exit
```

Entfernen des Containers

```powershell
C:\> docker rm iisdemo
```
Entfernen Sie das IIS-Image.

```powershell
C:\> docker rmi iis
```

## Hyper-V-Container

Hyper-V-Container bieten im Vergleich mit Windows Server-Containern eine zusätzliche Ebene der Isolation. Jeder Hyper-V-Container wird in einem überaus optimierten virtuellen Computer erstellt. Während sich ein Windows Server-Container einen Kernel mit dem Containerhost teilt, ist ein Hyper-V-Container vollständig isoliert. Hyper-V-Container werden wie Windows Server-Container erstellt und verwaltet. Weitere Informationen zu Hyper-V-Containern finden Sie unter [Verwalten von Hyper-V-Containern](../management/hyperv_container.md).

> Microsoft Azure unterstützt keine Hyper-V-Container. Für die Hyper-V-Übungen benötigen Sie einen lokalen Containerhost.

### Erstellen eines Containers

Da im Container ein Nano Server-Betriebssystemimage ausgeführt wird, sind für die Installation von IIS die IIS-Pakete für Nano Server erforderlich. Diese finden Sie auf dem Windows Server 2016 TP4-Installationsmedium im Verzeichnis `NanoServer\Packages`.

In diesem Beispiel wird dem ausgeführten Container über den Parameter `-v` von `docker run` ein Verzeichnis vom Containerhost zur Verfügung gestellt. Bevor Sie dies tun, muss das Quellverzeichnis konfiguriert werden.

Erstellen Sie ein Verzeichnis auf dem Containerhost, das für den Container freigegeben wird. Wenn Sie bereits die exemplarische Vorgehensweise für PowerShell abgeschlossen haben, sind dieses Verzeichnis und die benötigten Dateien ggf. bereits vorhanden.

```powershell
C:\> powershell New-Item -Type Directory c:\share\en-us
```

Kopieren Sie `Microsoft-NanoServer-IIS-Package.cab` aus `NanoServer\Packages` nach `c:\share` auf dem Containerhost.

Kopieren Sie `NanoServer\Packages\en-us\Microsoft-NanoServer-IIS-Package.cab` nach `c:\share\en-us` auf dem Containerhost.

Erstellen Sie im Ordner „c:\share“ eine Datei namens „Unattend.xml“, und kopieren Sie diesen Text in die Datei „Unattend.xml“.

```powershell
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <servicing>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" />
            <source location="c:\iisinstall\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="en-US" />
            <source location="c:\iisinstall\en-us\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
    </servicing>
</unattend>
```

Abschließend sollte das Verzeichnis `c:\share` auf dem Containerhost wie folgt konfiguriert sein.

```
c:\share
|-- en-us
|    |-- Microsoft-NanoServer-IIS-Package.cab
|
|-- Microsoft-NanoServer-IIS-Package.cab
|-- unattend.xml
```

Geben Sie zum Erstellen eines Hyper-V-Containers mit Docker den Parameter `–isolation=hyperv` an. Dieses Beispiel stellt das Verzeichnis `c:\share` auf dem Host für das Verzeichnis `c:\iisinstall` im Container bereit und erstellt dann eine interaktive Shellsitzung mit dem Container.

```powershell
C:\> docker run --name iisnanobase -it -v c:\share:c:\iisinstall --isolation=hyperv nanoserver cmd
```

### Erstellen des IIS-Images

Innerhalb der Containershellsitzung kann IIS mithilfe von `dism` installiert werden. Führen Sie zum Installieren von IIS im Container den folgenden Befehl aus.

```powershell
C:\> dism /online /apply-unattend:c:\iisinstall\unattend.xml
```

Wenn die IIS-Installation abgeschlossen ist, starten Sie IIS manuell mit dem folgenden Befehl.

```powershell
C:\> Net start w3svc
```

Beenden Sie die Containersitzung.

```powershell
C:\> exit
```

### Erstellen des IIS-Containers

Für den geänderten Nano Server-Container kann nun ein Commit in einem neuen Container-Image ausgeführt werden. Verwenden Sie hierzu den Befehl `docker commit`.

```powershell
C:\> docker commit iisnanobase nanoserveriis
```

Die Ergebnisse werden angezeigt, wenn eine Liste der Container-Images zurückgegeben wird.

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
nanoserveriis       latest              444435a4e30f        About a minute ago   69.14 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago          0 B
windowsservercore   latest              6801d964fda5        2 weeks ago          0 B
nanoserver          10.0.10586.0        8572198a60f1        2 weeks ago          0 B
nanoserver          latest              8572198a60f1        2 weeks ago          0 B
```

### Erstellen einer Anwendung

Das IIS-Image für Nano Server kann nun in einem neuen Container bereitgestellt werden.

```powershell
C:\> docker run -it -p 80:80 --isolation=hyperv nanoserveriis cmd
```

Führen Sie zum Entfernen des IIS-Begrüßungsbildschirms den folgenden Befehl aus.

```powershell
C:\> del C:\inetpub\wwwroot\iisstart.htm
```

Führen Sie den folgenden Befehl aus, um die IIS-Standardwebsite durch eine neue statische Website zu ersetzen.

```powershell
C:\> echo "Hello World From a Hyper-V Container" > C:\inetpub\wwwroot\index.html
```

Navigieren Sie zur IP-Adresse des Containerhosts. Jetzt sollte die „Hello World“-Anwendung angezeigt werden. Hinweis: Möglicherweise müssen Sie alle vorhandenen Browserverbindungen schließen oder den Browsercache leeren, um die aktualisierte Anwendung anzuzeigen.

![](media/HWWINServer.png)

Beenden Sie die interaktive Sitzung mit dem Container.

```powershell
C:\> exit
```








<!--HONumber=Feb16_HO4-->


