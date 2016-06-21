---
author: enderb-ms
redirect_url: https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-container-samples
---


# Erstellen eines .NET 3.5 Server Core-Containerimages

In dieser Anleitung wird erläutert, wie Sie einen Windows Server Core-Container erstellen, der das .NET 3.5-Framework umfasst. Bevor Sie mit dieser Übung beginnen, benötigen Sie die ISO-Datei für Windows Server 2016 oder Zugriff auf das Windows Server 2016-Installationsmedium.

## Vorbereiten der Medien

Bevor Sie ein .NET 3.5-fähiges Containerimage erstellen, muss das .NET 3.5-Paket für die Verwendung in einem Container bereitgestellt werden. Für dieses Beispiel wird die Datei `Microsoft-windows-netfx3-ondemand-package.cab` aus dem Windows Server 2016-Installationsmedium auf den Containerhost kopiert.

Erstellen Sie auf dem Containerhost ein Verzeichnis mit dem Namen `dotnet3.5\source`.

```powershell
New-Item -ItemType Directory c:\dotnet3.5\source
```

Kopieren Sie die Datei `Microsoft-windows-netfx3-ondemand-package.cab` in dieses Verzeichnis. Sie finden diese Datei im Ordner „sources\sxs“ des Windows Server 2016-Installationsmediums.

```powershell
$file = "d:\sources\sxs\Microsoft-windows-netfx3-ondemand-package.cab"
Copy-Item -Path $file -Destination c:\dotnet3.5\source
``` 
    
Wenn der Containerhost auf einem virtuellen Hyper-V-Computer ausgeführt wird und mit einem Schnellstartskript bereitgestellt wurde, können Sie auch Folgendes ausführen. Beachten Sie, dass die Ausführung auf dem Hyper-V-Host und nicht auf dem Containerhost erfolgt. 

```powershell
$vm = "<Container Host VM Name>"
$iso = "$((Get-VMHost).VirtualHardDiskPath)".TrimEnd("\") + "\WindowsServerTP4.iso"
Mount-DiskImage -ImagePath $iso
$ISOImage = Get-DiskImage -ImagePath $iso | Get-Volume
$ISODrive = "$([string]$iSOImage.DriveLetter):"
Get-VM -Name $vm | Enable-VMIntegrationService -Name "Guest Service Interface"
Copy-VMFile -Name $vm -SourcePath "$iSODrive\sources\sxs\microsoft-windows-netfx3-ondemand-package.cab" -DestinationPath "c:\dotnet3.5\source\microsoft-windows-netfx3-ondemand-package.cab" -FileSource Host -CreateFullPath
Dismount-DiskImage -ImagePath $iso
```

Sie können jetzt ein Containerimage erstellen, das in das .NET 3.5-Framework einbezogen wird. Verwenden Sie hierfür PowerShell oder Docker. Beispiele für beide sind unten aufgeführt.

## Erstellen eines Images mit PowerShell

Um ein neues Image mit PowerShell zu erstellen, wird ein Container erstellt, angepasst und dann in einem neuen Image erfasst.

Erstellen Sie anhand des Windows Server Core-Basisimages einen neuen Container.

```powershell
New-Container -Name dotnet35 -ContainerImageName windowsservercore -SwitchName "Virtual Switch"
```

Erstellen Sie einen freigegebenen Ordner für den neuen Container. Darin wird die .NET 3.5-CAB-Datei innerhalb des neuen Container zugänglich gemacht.  Beachten Sie, dass der Container bei der Ausführung des folgenden Befehls beendet werden muss.

```powershell
Add-ContainerSharedFolder -ContainerName dotnet35 -SourcePath C:\dotnet3.5\source -DestinationPath c:\sxs
```

Starten Sie den Container, und führen Sie zum Installieren von .NET 3.5 den folgenden Befehl aus.

```powershell
Start-Container dotnet35
Invoke-Command -ContainerName dotnet35 -ScriptBlock {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs} -RunAsAdministrator
```

Beenden Sie den Container nach Abschluss der Installation.

```powershell
Stop-Container dotnet35
```

Um ein Image dieses Containers zu erstellen, führen Sie Folgendes auf dem Containerhost aus.

```powershell
New-ContainerImage -ContainerName dotnet35 -Name dotnet35 -Publisher Demo -Version 1.0
```

Führen Sie `Get-ContainerImages` aus, um das neue Image anzuzeigen. Dieses Image kann jetzt verwendet werden, um einen Container mit dem vorinstallierten .NET 3.5-Framework auszuführen.

```powershell
Get-ContainerImages
```

## Erstellen eines Images mit Docker
 
Um ein neues Image mit Docker zu erstellen, wird eine Dockerfile-Datei mit Anweisungen zum Erstellen des neuen Images erstellt. Diese Dockerfile-Datei wird anschließend ausgeführt, was zu einem neuen Containerimage führt. Beachten Sie, dass die folgenden Befehle auf dem virtuellen Containerhostcomputer ausgeführt werden.

Erstellen Sie eine Dockerfile-Datei, und öffnen Sie sie in Editor.

```powershell
New-Item C:\dotnet3.5\dockerfile -Force
Notepad C:\dotnet3.5\dockerfile
```

Kopieren Sie diesen Text in die Dockerfile-Datei, und speichern Sie sie.

```powershell
FROM windowsservercore
ADD source /sxs
RUN powershell -Command "& {Add-WindowsFeature -Name NET-Framework-Core -Source c:\sxs}"
```

Führen Sie den Befehl `docker build` aus, der die Dockerfile-Datei nutzt, um das neue Containerimage zu erstellen.

```powershell
Docker build -t dotnet35 C:\dotnet3.5\
```

Führen Sie `docker images` aus, um das neue Image anzuzeigen. Dieses Image kann jetzt verwendet werden, um einen Container mit dem vorinstallierten .NET 3.5-Framework auszuführen.

```powershell
docker images
```


<!--HONumber=May16_HO3-->


