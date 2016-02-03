# Verwaltung von Containerressourcen

**Dieser Inhalt ist vorläufig und kann geändert werden.**

Für Windows-Container kann konfiguriert werden, wie viele CPU-, Datenträger-E/A-, Netzwerk- und Arbeitsspeicherressourcen ihnen zur Verfügung stehen. Das Einschränken von Containerressourcen ermöglicht eine effiziente Nutzung von Hostressourcen und verhindert eine übermäßige Nutzung. Dieses Dokument bietet ausführliche Informationen zum Verwalten von Containerressourcen mit PowerShell und Docker.

## Verwalten von Ressourcen mit PowerShell

### Arbeitsspeicher

Arbeitsspeicherlimits für Container können festgelegt werden, wenn bei Ausführung des Befehls `New-Container` zum Erstellen eines Containers der Parameter `-MaximumMemoryBytes` angegeben wird. In diesem Beispiel wird der maximale Arbeitsspeicher auf 256 MB festgelegt.

```powershell
PS C:\> New-Container –Name TestContainer –MaximumMemoryBytes 256MB -ContainerimageName WindowsServerCore
```
Mit dem Cmdlet `Set-ContainerMemory` können Sie auch das Arbeitsspeicherlimit eines vorhandenen Containers festlegen.

```powershell
PS C:\> Set-ContainerMemory -ContainerName TestContainer -MaximumBytes 256mb
```

### Netzwerkbandbreite

Für einen vorhandenen Container können Limits für die Netzwerkbandbreite festgelegt werden. Stellen Sie zunächst mit dem Befehl `Get-ContainerNetworkAdapter` sicher, dass der Container über einen Netzwerkadapter verfügt. Wenn kein Netzwerkadapter vorhanden ist, erstellen Sie einen mit dem Befehl `Add-ContainerNetworkAdapter`. Führen Sie abschließend den Befehl `Set-ContainerNetworkAdapter` aus, um die maximale ausgehende Netzwerkbandbreite des Containers zu begrenzen.

Im nachstehenden Beispiel ist die maximale Bandbreite auf 100 Mbit/s festgelegt.

```powershell
PS C:\> Set-ContainerNetworkAdapter –ContainerName TestContainer –MaximumBandwidth 100000000
```

### CPU

Sie können die CPU-Kapazität eines Containers begrenzen, indem Sie entweder einen maximalen Prozentsatz der CPU-Kapazität oder eine relative Gewichtung für den Container festlegen. Wenngleich eine Kombination dieser beiden CPU-Verwaltungsmodi möglich ist, wird sie nicht empfohlen. Standardmäßig können alle Container den Prozessor vollständig nutzen (maximal zu 100 %) und haben eine relative Gewichtung von 100.

Im folgenden Beispiel wird die relative Gewichtung des Containers auf 1000 festgelegt. Die Standardgewichtung eines Containers ist 100, weshalb dieser Container eine um das Zehnfache höhere Priorität als ein Container hat, der auf den Standardwert festgelegt ist. Der Maximalwert ist 10.000.

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 –RelativeWeight 10000
```

Sie können auch ein festes Limit für die CPU-Kapazität für einen Container mithilfe eines Prozentsatzes der CPU-Zeit festlegen. Standardmäßig kann ein Container die CPU zu 100 % nutzen. Im folgenden Beispiel wird der maximale Prozentsatz einer CPU, die ein Container nutzen kann, auf 30 % festgelegt. Durch Angeben des Kennzeichens „–Maximum“ wird „RelativeWeight“ automatisch auf 100 festgelegt.

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 -Maximum 30
```

### Speicher-E/A

Sie können begrenzen, wie viel E/A-Kapazität ein Container in Form von Bandbreite (Bytes pro Sekunde) oder mit 8 KB normalisierten IOPS nutzen darf. Diese beiden Parameter können zusammen festgelegt werden. Eine Drosselung tritt auf, wenn das erste Limit erreicht wird.

```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumBandwidth 1000000
```
```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumIOPS 32
```

## Verwalten von Ressourcen mit Docker

Wir bieten die Möglichkeit, eine Teilmenge der Containerressourcen mit Docker zu verwalten. Insbesondere ermöglichen wir Benutzern die Angabe, wie viel der CPU-Kapazität für Container freigegeben wird.

### CPU

CPU-Freigaben unter Containern können zur Laufzeit mithilfe des Kennzeichens „--cpu-shares“ verwaltet werden. Standardmäßig steht allen Container ein gleich großer Anteil der CPU-Zeit zur Verfügung. Um die relative CPU-Freigabe für Container zu ändern, geben Sie für das Kennzeichen „--cpu-shares“ einen Wert von 1 bis 10.000 an. Standardmäßig erhalten alle Container eine Gewichtung von 5000. Weitere Informationen zur CPU-Freigabeeinschränkung finden Sie in der [Referenz zu „Docker Run“](https://docs.docker.com/engine/reference/run/#cpu-share-constraint).

```powershell 
C:\> docker run –it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## Bekannte Probleme

- Für Hyper-V-Container wird die Steuerung von CPU- und E/A-Ressourcen derzeit nicht unterstützt.
- Die Steuerung von E/A-Ressourcen wird derzeit nicht für freigegebene Ordner in einem Container unterstützt.





<!--HONumber=Jan16_HO1-->
