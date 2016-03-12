



# Container und freigegebene Ordner

**Dieser Inhalt ist vorläufig und kann geändert werden.**

Freigegebene Ordner ermöglichen, dass Daten von einem Containerhost und Container gemeinsam genutzt werden. Nach seiner Erstellung steht der freigegebene Ordner im Container zur Verfügung. Daten, die vom Host im freigegebenen Ordner abgelegt werden, sind im Container verfügbar. Daten, die vom Container im freigegebenen Ordner abgelegt werden, sind im Host verfügbar. Ein einzelnen Ordner auf dem Host kann für viele Container freigegeben werden, und diese Konfigurationsdaten können von ausgeführten Containern gemeinsam genutzt werden.

## Verwalten von Daten – PowerShell

### Erstellen eines freigegebenen Ordners

Mit dem Befehl `Add-ContainerSharedFolder` können Sie einen freigegebenen Ordner erstellen. Im folgenden Beispiel wird ein Verzeichnis im Container `c:\shared_data` erstellt, der einem Verzeichnis auf dem Host `c:\data_source` zugeordnet wird.

> Damit ein Container einem freigegebenen Ordner hinzugefügt werden kann, muss sein Status „Beendet“ sein.

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\data_source -DestinationPath c:\shared_data

ContainerName SourcePath       DestinationPath AccessMode
------------- ----------       --------------- ----------
DEMO          c:\data_source   c:\shared_data  ReadWrite
```

### Schreibgeschützte freigegebene Ordner

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\sf1 -DestinationPath c:\sf2 -AccessMode ReadOnly

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\sf1     c:\sf2          ReadOnly
```

### Auflisten freigegebener Ordner

Um eine Liste freigegebener Ordner für einen bestimmten Container anzuzeigen, verwenden Sie den Befehl `Get-ContainerSharedFolder`.

```powershell
PS C:\> Get-ContainerSharedFolder -ContainerName DEMO2

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\source  c:\source       ReadWrite
```

### Ändern eines freigegebenen Ordners

Mit dem Befehl `Set-ContainerSharedFolder` können Sie eine vorhandene Konfiguration eines freigegebenen Ordners ändern.

```powershell
PS C:\> Set-ContainerSharedFolder -ContainerName SFRO -SourcePath c:\sf1 -DestinationPath c:\sf1
```

### Entfernen eines freigegebenen Ordners

Mit dem Befehl `Remove-ContainerSharedFolder` können Sie einen freigegebenen Ordner entfernen.

> Damit ein Container entfernt werden kann, muss sein Status „Beendet“ sein.

```powershell
PS C:\> Remove-ContainerSharedFolder -ContainerName DEMO2 -SourcePath c:\source -DestinationPath c:\source
```
## Verwalten von Daten – Docker

### Bereitstellen von Volumes

Bei der Verwaltung von Windows-Containern mit Docker können Volumes mithilfe der Option `-v` bereitgestellt werden.

In dem folgenden Beispiel ist „c:\source“ der Quellordner und „c:\destination“ der Zielordner.

```powershell
PS C:\> docker run -it -v c:\source:c:\destination 1f62aaf73140 cmd
```

Weitere Informationen zur Datenverwaltung in Containern mit Docker finden Sie unter [Docker Volumes auf Docker.com](https://docs.docker.com/userguide/dockervolumes/).

## Video zur exemplarischen Vorgehensweise

<iframe src="https://channel9.msdn.com/Blogs/containers/Container-Fundamentals--Part-3-Shared-Folders/player#ccLang=de" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>



<!--HONumber=Feb16_HO3-->
