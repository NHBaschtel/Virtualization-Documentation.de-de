# Containerimages

**Dieser Inhalt ist vorläufig und kann geändert werden.**

Containerimages dienen zum Bereitstellen von Containern. Diese Images können ein Betriebssystem, Anwendungen und sämtliche Anwendungsabhängigkeiten enthalten. Sie können z. B. ein Containerimage entwickeln, das mit Nano Server, IIS und einer in IIS ausgeführten Anwendung vorkonfiguriert wurde. Dieses Containerimage kann anschließend für die spätere Verwendung in einer Containerregistrierung gespeichert, auf einem beliebigen Windows-Containerhost (lokal, Cloud oder Containerdienst) bereitgestellt und außerdem als Basis für ein neues Containerimage verwendet werden.

Es gibt zwei Arten von Containerimages:

- Basisbetriebssystem-Images, die von Microsoft bereitgestellt werden und die wesentlichen Betriebssystemkomponenten enthalten.
- Containerimages, die anhand eines Basisbetriebssystem-Images erstellt wurden.

## PowerShell

### Auflisten von Images

Führen Sie `get-containerImage` aus, um eine Liste der Images auf dem Containerhost zurückzugegeben. Die Eigenschaft `IsOSImage` dient zum Unterscheiden der Typen von Containerimages.

```powershell
PS C:\> Get-ContainerImage

Name                    Publisher       Version         IsOSImage
----                    ---------       -------         ---------
NanoServer              CN=Microsoft    10.0.10586.0    True
WindowsServerCore       CN=Microsoft    10.0.10586.0    True
WindowsServerCoreIIS    CN=Demo         1.0.0.0         False
```

### Installieren von Basisbetriebssystem-Images

Containerbetriebssystem-Images können mithilfe des PowerShell-Moduls „ContainerProvider“ bestimmt und installiert werden. Sie müssen dieses Modul erst installieren, um es verwenden zu können. Installieren Sie das Modul über die folgenden Befehle.

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

Zum Herunterladen und Installieren des Basisbetriebssystem-Images für Nano Server führen Sie die folgenden Schritte aus.

```powershell
PS C:\> Install-ContainerImage -Name NanoServer -Version 10.0.10586.0

Downloaded in 0 hours, 0 minutes, 10 seconds.
```

Mit diesem Befehl wird auch das Basisbetriebssystem-Image für Windows Server Core heruntergeladen und installiert.

> **Problem:** Die Cmdlets „Save-ContainerImage“ und „Install-ContainerImage“ funktionieren in einer PowerShell-Remotesitzung nicht mit einem „WindowsServerCore“-Containerimage. **Lösung:** Melden Sie sich über Remotedesktop am Computer an, und verwenden Sie das Cmdlet „Save-ContainerImage“ direkt.

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
Weitere Informationen zur Verwaltung von Containerimages finden Sie unter [Windows-Containerimages](../management/manage_images.md).

### Erstellen eines neuen Images

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

## Docker

### Auflisten von Images

```powershell
C:\> docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.10586.0        6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.10586.0        8572198a60f1        2 weeks ago          0 B
```

### Erstellen eines neuen Images

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

### Docker Hub

Die Docker Hub-Registrierung enthält vordefinierte Images, die auf einen Containerhost heruntergeladen werden können. Sobald diese Images heruntergeladen wurden, können sie als Basis für Windows-Containeranwendungen verwendet werden.

Über den Befehl `docker search` können Sie eine Liste der auf Docker Hub verfügbaren Images anzeigen. Hinweis: Das Basisbetriebssystem-Image für Windows Server Core muss installiert sein, ehe von Windows Server Core abhängige Images von Docker Hub bezogen werden.

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
```

Über den Befehl `docker pull` können Sie ein Image von Docker Hub herunterladen.

```powershell
C:\> docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

Bei Ausführung von `docker images` wird das Image angezeigt.

```powershell
C:\> docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.10586.0        6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```

### Image-Abhängigkeit

Zum Anzeigen von Image-Abhängigkeiten mit Docker dient der Befehl `docker history`.

```powershell
C:\> docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```



