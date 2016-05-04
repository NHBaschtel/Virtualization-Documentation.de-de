---
author: neilpeterson
---

# Containerimages

**Dieser Inhalt ist vorläufig und kann geändert werden.**

Containerimages dienen zum Bereitstellen von Containern. Diese Images können ein Betriebssystem, Anwendungen und sämtliche Anwendungsabhängigkeiten enthalten. Sie können z. B. ein Containerimage entwickeln, das mit Nano Server, IIS und einer in IIS ausgeführten Anwendung vorkonfiguriert wurde. Dieses Containerimage kann anschließend für die spätere Verwendung in einer Containerregistrierung gespeichert, auf einem beliebigen Windows-Containerhost (lokal, Cloud oder Containerdienst) bereitgestellt und außerdem als Basis für ein neues Containerimage verwendet werden.

Es gibt zwei Arten von Containerimages:

- **Basisbetriebssystemimages**: Werden von Microsoft bereitgestellt und enthalten die wesentlichen Betriebssystemkomponenten.
- **Containerimages**: Ein benutzerdefiniertes Containerimage, das vom Basisbetriebssystemimage abgeleitet wird.

## Basisbetriebssystemimages

### Installieren eines Images

Containerbetriebssystemimages können für die PowerShell- und Docker-Verwaltung mithilfe des PowerShell-Moduls „ContainerProvider“ bestimmt und installiert werden. Sie müssen dieses Modul erst installieren, um es verwenden zu können. Installieren Sie das Modul mit dem folgenden Befehl.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

Nach der Installation kann mit `Find-ContainerImage` eine Liste der Basisbetriebssystemimages zurückgegeben werden.

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

Zum Herunterladen und Installieren des Basisbetriebssystem-Images für Nano Server führen Sie die folgenden Schritte aus. Der Parameter `–version` ist optional. Wenn keine Version für das Basisbetriebssystem-Image angegeben wird, wird die neueste Version installiert.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

Mit diesem Befehl wird auch das Basisbetriebssystemimage für Windows Server Core heruntergeladen und installiert. Der Parameter `–version` ist optional. Wenn keine Version für das Basisbetriebssystem-Image angegeben wird, wird die neueste Version installiert.

> **Problem**: Die Cmdlets „Save-ContainerImage“ und „Install-ContainerImage“ funktionieren in einer PowerShell-Remotesitzung möglicherweise nicht mit einem WindowsServerCore-Containerimage. **Lösung:** Melden Sie sich über Remotedesktop am Computer an, und verwenden Sie das Cmdlet „Save-ContainerImage“ direkt.

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

Überprüfen Sie mit dem Befehl `Get-ContainerImage`, ob die Images Installiert wurden.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

> Mit **Install-ContainerImage** wird ein Basisbetriebssystemimage zur Verwendung in Containern installiert, die von PowerShell oder Docker verwaltet werden. Wenn das Basisbetriebssystemimage heruntergeladen wurde, aber beim Ausführen von `docker images` nicht angezeigt wird, starten Sie den Docker-Dienst mithilfe des Systemsteuerungs-Applets für Dienste oder den Befehlen „sc docker stop“ und „sc docker start“ neu.

### Offineinstallation

Basisbetriebssystemimages können auch ohne Internetzugang installiert werden. Dazu werden die Images auf einen Computer mit Internetverbindung heruntergeladen, auf das Zielsystem kopiert und dann mit dem Befehl `Install-ContainerOSImages` importiert.

Bereiten Sie das System vor dem Herunterladen des Basisbetriebssystemimages durch Ausführen des folgenden Befehls mit dem Containerimageanbieter vor.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

So geben Sie eine Liste der Images aus dem PowerShell OneGet-Paket-Manager zurück:

```powershell
PS C:\> Find-ContainerImage

Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.10586.0            Container OS Image of Windows Server 2016 Techn...
```

Um ein Image herunterzuladen, verwenden Sie den Befehl `Save-ContainerImage`.

```powershell
PS C:\> Save-ContainerImage -Name NanoServer -Destination c:\container-image\NanoServer.wim
```

Das heruntergeladene Containerimage kann jetzt auf einen anderen Containerhost kopiert und mit dem Befehl `Install-ContainerOSImage` installiert werden.

```powershell
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### Kennzeichnen von Images

Wenn auf ein Containerimage anhand des Namens verwiesen wird, sucht das Docker-Modul nach der neuesten Version des Images. Wenn die neueste Version nicht bestimmt werden kann, wird der folgende Fehler ausgelöst.

```powershell
PS C:\> docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

Nach der Installation von Windows Server Core- oder Nano Server-Basisbetriebssystemimages müssen diese mit „Latest“ als neueste Versionen gekennzeichnet werden. Verwenden Sie hierzu den Befehl `docker tag`.

Weitere Informationen zu `docker tag` finden Sie auf „docker.com“ unter [Tag, push, and pull your image](https://docs.docker.com/mac/step_six/).

```powershell
PS C:\> docker tag <image id> windowsservercore:latest
```

Nach dem Kennzeichnen werden in der Ausgabe von `docker images` zwei Versionen desselben Images angezeigt: eines mit dem Tag der Imageversion und ein zweites mit dem Tag „Latest“. Jetzt kann mit dem Namen auf das Image verwiesen werden.

```powershell
PS C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14289.1000     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14289.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### Deinstallieren des Betriebssystemimages

Basisbetriebssystemimages können mit dem Befehl `Uninstall-ContainerOSImage` deinstalliert werden. Im folgende Beispiel wird das NanoServer-Basisbetriebssystemimage deinstalliert.

```powershell
Get-ContainerImage -Name NanoServer | Uninstall-ContainerOSImage
```

## Containerimages, PowerShell

### Auflisten von Images

Führen Sie `Get-ContainerImage` aus, um eine Liste der Images auf dem Containerhost zurückzugeben. Die Eigenschaft `IsOSImage` dient zum Unterscheiden der Typen von Containerimages.

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### Erstellen eines neuen Images

Sie können ein neues Containerimage aus einem vorhandenen Container erstellen. Verwenden Sie hierzu den Befehl `New-ContainerImage`.

```powershell
PS C:\> New-ContainerImage -Container $container -Publisher Demo -Name DemoImage -Version 1.0
```

### Entfernen eines Images

Containerimages können nicht entfernt werden, wenn ein Container, selbst mit dem Status „Beendet“, eine Abhängigkeit vom Image aufweist.

Entfernen Sie ein einzelnes Image mit PowerShell.

```powershell
PS C:\> Get-ContainerImage -Name newimage | Remove-ContainerImage -Force
```

### Image-Abhängigkeit

Wenn ein neues Image erstellt wird, wird es vom Image abhängig, anhand dessen es erstellt wurde. Diese Abhängigkeit kann mit dem Befehl `Get-ContainerImage` angezeigt werden. Wenn ein übergeordnetes Image nicht aufgeführt ist, heißt dies, dass das Image eine Basisbetriebssystem-Image ist.

```powershell
PS C:\> Get-ContainerImage | select Name, ParentImage

Name              ParentImage
----              -----------
NanoServerIIS     ContainerImage (Name = 'NanoServer') [Publisher = 'CN=Microsoft', Version = '10.0.10586.0']
NanoServer
WindowsServerCore
```

### Verschieben des Imagerepositorys

Wenn Sie ein neues Containerimage mit dem Befehl `New-ContainerImage` erstellen, wird dieses Image im Standardverzeichnis „C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store“ gespeichert. Dieses Repository kann mit dem Befehl `Move-ContainerImageRepository` verschoben werden. Mit dem folgenden Befehl würde beispielsweise ein neues Containerimagerepository unter „C:\container-images“ erstellt.

```powershell
Move-ContainerImageRepository -Path c:\container-images
```
> Der mit dem Befehl `Move-ContainerImageRepository` verwendete Pfad braucht beim Ausführen des Befehls noch nicht vorhanden zu sein.

## Containerimages, Docker

### Auflisten von Images

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### Erstellen eines neuen Images

Sie können ein neues Containerimage aus einem vorhandenen Container erstellen. Verwenden Sie hierzu den Befehl `docker commit`. Dieses Beispiel erstellt ein neues Containerimage mit dem Namen „windowsservercoreiis“.

```powershell
C:\> docker commit 475059caef8f windowsservercoreiis

ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Entfernen eines Images

Containerimages können nicht entfernt werden, wenn ein Container, selbst mit dem Status „Beendet“, eine Abhängigkeit vom Image aufweist.

Wenn Sie ein Image mit Docker entfernen, kann auf die Images anhand des Imagenamens oder der ID verwiesen werden.

```powershell
C:\> docker rmi windowsservercoreiis

Untagged: windowsservercoreiis:latest
Deleted: ca40b33453f803bb2a5737d4d5dd2f887d2b2ad06b55ca681a96de8432b5999d
```

### Image-Abhängigkeit

Zum Anzeigen von Image-Abhängigkeiten mit Docker dient der Befehl `docker history`.

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

Die Docker Hub-Registrierung enthält vordefinierte Images, die auf einen Containerhost heruntergeladen werden können. Sobald diese Images heruntergeladen wurden, können sie als Basis für Windows-Containeranwendungen verwendet werden.

Über den Befehl `docker search` können Sie eine Liste der auf Docker Hub verfügbaren Images anzeigen. Hinweis: Das Basisbetriebssystemimage für Windows Server Core oder Nano Server muss installiert werden, ehe diese abhängigen Images von Docker Hub bezogen werden.

> Die Images, die mit „nano-“ beginnen, sind vom Nano Server-Basisbetriebssystemimage abhängig.

```powershell
C:\> docker search *

NAME                    DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/aspnet        ASP.NET 5 framework installed in a Windows...   1         [OK]       [OK]
microsoft/django        Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35      .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/golang        Go Programming Language installed in a Win...   1                    [OK]
microsoft/httpd         Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis           Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/mongodb       MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/mysql         MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/nginx         Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/node          Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/php           PHP running on Internet Information Servic...   1                    [OK]
microsoft/python        Python installed in a Windows Server Core ...   1                    [OK]
microsoft/rails         Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/redis         Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/ruby          Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sqlite        SQLite installed in a Windows Server Core ...   1                    [OK]
microsoft/nano-golang   Go Programming Language installed in a Nan...   1                    [OK]
microsoft/nano-httpd    Apache httpd installed in a Nano Server ba...   1                    [OK]
microsoft/nano-iis      Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/nano-mysql    MySQL installed in a Nano Server based con...   1                    [OK]
microsoft/nano-nginx    Nginx installed in a Nano Server based con...   1                    [OK]
microsoft/nano-node     Node installed in a Nano Server based cont...   1                    [OK]
microsoft/nano-python   Python installed in a Nano Server based co...   1                    [OK]
microsoft/nano-rails    Ruby on Rails installed in a Nano Server b...   1                    [OK]
microsoft/nano-redis    Redis installed in a Nano Server based con...   1                    [OK]
microsoft/nano-ruby     Ruby installed in a Nano Server based cont...   1                    [OK]
```

Über den Befehl `docker pull` können Sie ein Image von Docker Hub herunterladen.

```powershell
C:\> docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

Bei Ausführung von `docker images` wird das Image jetzt angezeigt.

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```






<!--HONumber=Mar16_HO3-->


