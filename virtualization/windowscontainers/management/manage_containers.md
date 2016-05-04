



# Verwaltung von Windows Server-Containern

**Dieser Inhalt ist vorläufig und kann geändert werden.**

Zum Lebenszyklus von Containern gehören Aktionen wie das Starten, Beenden und Entfernen von Containern. Bei diesen Aktionen müssen Sie möglicherweise auch eine Liste mit Containerimages abrufen, Netzwerkverbindungen von Containern verwalten und Containerressourcen beschränken. Dieses Dokument enthält ausführliche Informationen zum Ausführen grundlegender Verwaltungsaufgaben für Container mithilfe von PowerShell.

Informationen zur Verwaltung von Windows-Containern mit Docker finden Sie im Docker-Dokument zum [Arbeiten mit Containern](https://docs.docker.com/userguide/usingdocker/).

## PowerShell

### Erstellen eines Containers

Wenn Sie einen neuen Container erstellen möchten, benötigen Sie den Namen eines Containerimages, das als Grundlage des Containers fungieren soll. Führen Sie hierzu den Befehl `Get-ContainerImage` aus.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version         IsOSImage
----              ---------    -------         ---------
NanoServer        CN=Microsoft 10.0.10584.1000 True
WindowsServerCore CN=Microsoft 10.0.10584.1000 True
```

Mit dem Befehl `New-Container` können Sie einen neuen Container erstellen. Dem Container kann auch mithilfe des Parameters `- ContainerComputerName` ein NetBIOS-Name zugewiesen werden.

```powershell
PS C:\> New-Container -ContainerImageName WindowsServerCore -Name demo -ContainerComputerName demo

Name State Uptime   ParentImageName
---- ----- ------   ---------------
demo  Off   00:00:00 WindowsServerCore
```

Nachdem der Container erstellt wurde, fügen Sie dem Container einen Netzwerkadapter hinzu.

```powershell
PS C:\> Add-ContainerNetworkAdapter -ContainerName demo
```

Um den Netzwerkadapter des Containers mit einem virtuellen Switch zu verbinden, wird der Switchname benötigt. Mit `Get-VMSwitch` können Sie eine Liste virtueller Switches zurückgeben.

```powershell
PS C:\> Get-VMSwitch

Name SwitchType NetAdapterInterfaceDescription
---- ---------- ------------------------------
DHCP External   Microsoft Hyper-V Network Adapter
NAT  NAT
```

Verbinden Sie den Netzwerkadapter über `Connect-ContainerNetworkAdapter` mit dem virtuellen Switch. **HINWEIS**: Dieser Schritt kann auch bei der Erstellung des Containers mithilfe des Parameters „–SwitchName“ ausgeführt werden.

```powershell
PS C:\> Connect-ContainerNetworkAdapter -ContainerName demo -SwitchName NAT
```

### Starten eines Containers

Zum Starten des Containers wird ein PowerShell-Objekt aufgezählt, das den Container darstellt. Dazu kann die Ausgabe von `Get-Container` einer PowerShell-Variablen hinzugefügt werden.

```powershell
PS C:\> $container = Get-Container -Name demo
```

Diese Daten können dann mit dem Befehl `Start-Container` verwendet werden, um den Container zu starten.

```powershell
PS C:\> Start-Container $container
```

Das folgende Skript startet alle Container auf dem Host.

```powershell
PS C:\> Get-Container | Start-Container
```

### Verbinden mit einem Container

PowerShell Direct kann verwendet werden, um eine Verbindung mit einem Container herzustellen. Dies ist möglicherweise hilfreich, wenn Sie eine Aufgabe manuell ausführen müssen, wie z. B. das Installieren von Software, Starten eines Prozesses oder Behandeln eines Problems mit einem Container. Da PowerShell Direct verwendet wird, kann unabhängig von der Netzwerkkonfiguration eine PowerShell-Sitzung mit dem Container hergestellt werden. Weitere Informationen zu PowerShell Direct finden Sie im [Blog zu PowerShell Direct](http://blogs.technet.com/b/virtualization/archive/2015/05/14/powershell-direct-running-powershell-inside-a-virtual-machine-from-the-hyper-v-host.aspx).

Um eine interaktive Sitzung mit dem Container herzustellen, verwenden Sie den Befehl `Enter-PSSession`.

 ```powershell
PS C:\> Enter-PSSession -ContainerName demo -RunAsAdministrator
 ```

Nachdem die PowerShell-Remotesitzung erstellt wurde, ändert sich die PowerShell-Eingabeaufforderung entsprechend dem Containernamen.

```powershell
[demo]: PS C:\>
```

Befehle können auch auf einen Container angewendet werden, ohne dass eine beständige PowerShell-Sitzung eingerichtet wird. Verwenden Sie hierzu `Invoke-Command`.

Das folgende Beispiel erstellt einen Ordner namens „Application“ im Container.

```powershell

PS C:\> Invoke-Command -ContainerName demo -ScriptBlock {New-Item -ItemType Directory -Path c:\application }

Directory: C:\
Mode                LastWriteTime         Length Name                                                 PSComputerName
----                -------------         ------ ----                                                 --------------
d-----       10/28/2015   3:31 PM                application                                          demo
```

### Beenden eines Containers

Zum Beenden des Containers wird ein PowerShell-Objekt benötigt, das den Container darstellt. Dazu kann die Ausgabe von `Get-Container` einer PowerShell-Variablen hinzugefügt werden.

```powershell
PS C:\> $container = Get-Container -Name demo
```

Diese Daten können dann mit dem Befehl `Stop-Container` verwendet werden, um den Container zu beenden.

```powershell
PS C:\> Stop-Container $container
```

Das folgende Skript beendet alle Container auf dem Host.

```powershell
PS C:\> Get-Container | Stop-Container
```

### Entfernen eines Containers

Wenn ein Container nicht mehr benötigt wird, kann er entfernt werden. Um einen Container zu entfernen, muss er den Status „Beendet“ haben. Außerdem muss ein PowerShell-Objekt erstellt werden, das den Container darstellt.

```powershell
PS C:\> $container = Get-Container -Name demo
```

Verwenden Sie zum Entfernen des Containers den Befehl `Remove-Container`.

```powershell
PS C:\> Remove-Container $container -Force
```

Das folgende Skript entfernt alle Container vom Host.

```powershell
PS C:\> Get-Container | Remove-Container -Force
```

## Docker

### Erstellen eines Containers

Verwenden Sie den Befehl `docker run`, um einen Container mit Docker zu erstellen.

```powershell
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Weitere Informationen zum Befehl „Docker run“ finden Sie in der [Referenz zu Docker run}( https://docs.docker.com/engine/reference/run/).

### Beenden eines Containers

Verwenden Sie den Befehl `docker stop`, um einen Container mit Docker zu beenden.

```powershell
PS C:\> docker stop tender_panini

tender_panini
```

Bei diesem Beispiel werden alle ausgeführten Container mit Docker beendet.

```powershell
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### Entfernen eines Containers

Verwenden Sie zum Entfernen eines Containers mit Docker den Befehl `docker rm`.

```powershell
PS C:\> docker rm prickly_pike

prickly_pike
```

So entfernen Sie alle Container mit Docker

```powershell
PS C:\> docker rm $(docker ps -a -q)

dc3e282c064d
2230b0433370
```

Weitere Informationen zum Befehl „Docker rm“ finden Sie in der [Referenz zu Docker rm](https://docs.docker.com/engine/reference/commandline/rm/).






<!--HONumber=Feb16_HO4-->


