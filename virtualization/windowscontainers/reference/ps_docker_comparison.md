# Vergleichen der Verwaltung von Windows-Containern mit PowerShell und Docker

Es gibt viele Methoden zum Verwalten von Windows-Containern sowohl mit integrierten Windows-Tools (in dieser Vorschau PowerShell) als auch Open-Source-Verwaltungstools wie Docker.  
Es gibt auch zwei Leitfäden, in denen diese Vorgänge toolbezogen beschrieben werden:
* [Verwalten von Windows-Containern mit Docker](../quick_start/manage_docker.md)
* [Verwalten von Windows-Containern mit PowerShell](../quick_start/manage_powershell.md)

Diese Seite bietet eine detailliertere Referenz mit einem Vergleich der Docker-Tools und PowerShell-Verwaltungstools.

## PowerShell für Container im Vergleich zu Hyper-V-VMs

Sie können Windows-Container mithilfe von PowerShell-Cmdlets erstellen, ausführen und damit interagieren. Alles, was Sie brauchen, ist standardmäßig verfügbar.

Wenn Sie mit PowerShell für Hyper-V gearbeitet haben, sollte Ihnen die Art der Cmdlets bereits bekannt sein. Ein Großteil des Workflows entspricht dem Verwalten eines virtuellen Computers mit dem Hyper-V-Modul. Anstelle von `New-VM`, `Get-VM`, `Start-VM` und `Stop-VM` verwenden Sie `New-Container`, `Get-Container`, `Start-Container` und `Stop-Container`. Es gibt einige containerspezifische Cmdlets und Parameter, aber der allgemeinen Lebenszyklus samt Verwaltung eines Windows-Containers entsprecht in etwa dem einer Hyper-V-VM.

## Wie lässt sich die PowerShell-Verwaltung mit Docker vergleichen?

Die PowerShell-Cmdlets für Container machen eine API verfügbar, die nicht mit der von Docker identisch ist. Im Allgemeinen sind die Cmdlets differenzierter. Für einige Docker-Befehle gibt es eindeutige Parallelen in PowerShell:

| Docker-Befehl| PowerShell-Cmdlet|
|----|----|
| `docker ps -a`| `Get-Container`|
| `docker images`| `Get-ContainerImage`|
| `docker rm`| `Remove-Container`|
| `docker rmi`| `Remove-ContainerImage`|
| `docker create`| `New-Container`|
| `docker commit <Container-ID>`| `New-ContainerImage -Container <Container>`|
| `docker load <tarball>`| `Import-ContainerImage <AppX-Paket>`|
| `docker save`| `Export-ContainerImage`|
| `docker start`| `Start-Container`|
| `docker stop`| `Stop-Container`|

Die PowerShell-Cmdlets sind keine exakten perfekten Gegenstücke. Es gibt ziemlich viele Befehle, für die wir keine PowerShell-Entsprechung bieten* (insbesondere `docker build` und `docker cp`). Es wird Ihnen jedoch möglicherweise auffallen, dass es keinen einzeiligen Ersatz für `docker run` gibt.

\* Änderungen vorbehalten.

### Aber ich brauche „docker run“! Was passiert?

Wir führen hier verschiedene Schritte aus, um Benutzern, die sich mit PowerShell bereits auskennen, ein etwas vertrauteres Interaktionsmodell zu bieten. Wenn Sie allerdings an das Arbeiten mit Docker gewöhnt sind, ist ein gewisses Umdenken erforderlich.

1.  Der Lebenszyklus eines Containers im PowerShell-Modell ist etwas anders. Im PowerShell-Modul für Container machen wir die differenzierteren Vorgänge `New-Container` (wodurch ein neuer Container erstellt wird, der beendet wird) und `Start-Container` verfügbar.

    Zwischen dem Erstellen und Starten des Containers können Sie auch die Einstellungen des Containers konfigurieren. Für TP3 ist die einzige andere Konfiguration, die wir verfügbar machen möchten, die Fähigkeit zum Festlegen der Netzwerkverbindung des Containers mithilfe der „ContainerNetworkAdapter“-Cmdlets (Add/Remove/Connect/Disconnect/Get/Set)

2.  Sie können derzeit keinen Befehl für die Ausführung im Container beim Start übergeben. Sie können allerdings weiter eine interaktive PowerShell-Sitzung mit einem ausgeführten Container einrichten. Verwenden Sie dazu `Enter-PSSession -ContainerId <ID eines ausgeführten Containers)`. Sie können auch einen Befehl in einem ausgeführten Container ausführen. Verwenden Sie dazu `Invoke-Command -ContainerId <Container-ID> -ScriptBlock {im Container auszuführender Code}` oder `Invoke-Command -ContainerId <Container-ID> -FilePath <Pfad zum Skript>`.  
    Beide Befehle lassen das optionale Kennzeichen `-RunAsAdministrator` für Aktionen mit höheren Berechtigungen zu.


## Einschränkungen und bekannte Probleme

1.  Bislang haben die Container-Cmdlets keine Kenntnis von Containern oder Images, die mit Docker erstellt wurden. Umgekehrt weiß Docker nichts über Container und Images, die mit PowerShell erstellt wurden. Wenn Sie diese in Docker erstellt haben, verwalten Sie sie mit Docker. Wenn Sie sie über PowerShell erstellt haben, verwalten Sie sie über PowerShell.

2.  Wir haben noch einiges zu tun, um die Endbenutzerumgebung zu verbessern: bessere Fehlermeldungen, bessere Fortschrittsberichte, ungültige Ereigniszeichenfolgen usw. Wenn Sie in eine Situation geraten, in der Sie sich wünschten, Sie erhielten mehr oder bessere Informationen, senden Sie uns bitte Vorschläge in den Foren.

## Eine kurze exemplarische Vorgehensweise

Nun wollen wir verschiedene allgemeine Workflows durchlaufen.

Dafür wird vorausgesetzt, dass Sie ein Betriebssystemcontainer-Image mit dem Namen „ServerDatacenterCore“ installiert und einen virtuellen Switch mit dem Namen „Virtual Switch“ (mithilfe von „New-VMSwitch“) erstellt haben.

``` PowerShell
### 1. Enumerating images
# At this point, you can enumerate the images on the system:
Get-ContainerImage

# Get-ContainerImage also accepts filters.
# For example, this will return all container images whose Name starts with S (case-insensitive):
Get-ContainerImage -Name S*

# You can save the results of this to a variable.
# (If you're not familiar with PowerShell, the "$" denotes a variable.)
$baseImage = Get-ContainerImage -Name ServerDatacenterCore
$baseImage

### 2. Creating and enumerating containers
# Now, we can create a new container using this image:
New-Container -Name "MyContainer" -ContainerImage $baseImage -SwitchName "Virtual Switch"

# Now we can enumerate all containers.
Get-Container

# Similarly, we can save this container to a variable:
$container1 = Get-Container -Name "MyContainer"

### 3. Starting containers, interacting with running containers, and stopping containers
# Now let's go ahead and start the container.
Start-Container -Name "MyContainer"

# (We could've also started this container using "Start-Container -Container $container1".)

# With the container now running, let's go ahead and enter an interactive PowerShell session:
Enter-PSSession -ContainerId $container1.Id

# This should eventually bring up a PowerShell prompt from inside the container.
# You can try all the things that you did in the interactive cmd prompt given by "docker run -it".
# For now, just to prove we've been here, we can create a new file:
cd \
mkdir Test
cd Test
echo "hello world" > hello.txt
exit

# Now we should be back in the outside world. Even though we've exited the PowerShell session,
# the container itself is still running, as you can see by printing out the container again:
$container1

# Before we can commit this container to a new image, we need to stop the container.
# Let's do that now.
Stop-Container -Container $container1

### 4. Creating a new container image
# And now let's commit it to a new image.
$image1 = New-ContainerImage -Container $container1 -Publisher Test -Name Image1 -Version 1.0

# Enumerate all the images again, for sanity's sake:
Get-ContainerImage

# Rinse and repeat! Make another container based on the new image.
$container2 = New-Container -Name "MySecondContainer" -ContainerImage $image1 -SwitchName "Virtual Switch"

# (If you like, you can start the second container and verify that the new file
# "\Test\hello.txt" is there as expected.)

### 5. Removing a container
# The first container we created is now stopped. Let's get rid of it:
Remove-Container -Container $container1

# And confirm that it's gone:
Get-Container

### 6. Exporting, removing, and importing images
# For images that aren't the base OS image, we can export them into an .appx package file.
Export-ContainerImage -Image $image1 -Path "C:\exports"
# This should create a .appx file in the C:\exports folder.
# If you've given your image the same publisher, name, and version we used earlier,
# you'd expect the resulting .appx to be named "CN=Test_Image1_1.0.0.0.appx".

# Before we can try importing the image again, we need to remove the image.
# (If you have any running containers that depend on this image, you'll want to stop them first.)
Remove-ContainerImage -Image $image1

# Now let's import the image again:
Import-ContainerImage -Path C:\exports\CN=Test_Image1_1.0.0.0.appx

# We'd previously created a container dependent on this image. You should be able to start it:
Start-Container -Container $container2 
```

### Erstellen Ihres eigenen Beispiels

Mit dem Befehl `Get-Command -Module Containers` können Sie alle Cmdlets für Container anzeigen. Es gibt mehrere andere Cmdlets, die hier nicht beschrieben und zu denen Sie sich selbst schlau machen müssen.    
**Hinweis** Die Cmdlets `Enter-PSSession` und `Invoke-Command`, die Teil der wichtigsten PowerShell-Cmdlets sind, werden dadurch nicht zurückgegeben.

Über `Get-Help [Name des Cmdlets]` oder `[Name des Cmdlets] -?` erhalten Sie Hilfe zu sämtlichen Cmdlets. Derzeit wird die Ausgabe der Hilfe automatisch generiert und informiert Sie bloß über die Syntax der Befehle. Sobald wir uns dem Abschluss des Cmdlet-Designs nähern, fügen wir weitere Dokumentation hinzu.

Eine bessere Möglichkeit, die Syntax zu ermitteln, ist PowerShell ISE, womit Sie sich ggf. noch nicht beschäftigt haben, wenn Sie PowerShell zuvor noch nicht ausgiebig genutzt haben. Wenn Sie mit einer SKU arbeiten, die dies zulässt, versuchen Sie, die ISE zu starten. Öffnen Sie dazu den Bereich „Befehle“, und wählen Sie das Modul „Container“ aus. Dadurch wird eine grafische Darstellung der Cmdlets und ihrer Parametersätze angezeigt.

PS: Um nachzuweisen, dass es möglich ist, hier nun eine PowerShell-Funktion, die aus einigen der Cmdlets besteht, die wir bereits als Ersatz für `docker run` gesehen haben. (Hierbei handelt es sich erst einmal um eine Machbarkeitsstudie und keine aktive Entwicklungsphase.)

``` PowerShell
function Run-Container ([string]$ContainerImageName, [string]$Name="fancy_name", [switch]$Remove, [switch]$Interactive, [scriptblock]$Command) {
    $image = Get-ContainerImage -Name $ContainerImageName
    $container = New-Container -Name $Name -ContainerImage $image
    Start-Container $container

    if ($Interactive) {
         Start-Process powershell ("-NoExit", "-c", "Enter-PSSession -ContainerId $($container.Id)") -Wait
    } else {
        Invoke-Command -ContainerId $container.Id -ScriptBlock $Command
    }

    Stop-Container $container

    if ($Remove) {
        Remove-Container $container -Force
    }
} 
```

## Docker

Windows-Container können mit Docker-Befehlen verwaltet werden. Während Windows-Container mit ihren Linux-Entsprechungen vergleichbar sein sollten und über Docker gleich verwaltet werden sollten, gibt es einige Docker-Befehle, die für einen Windows-Container schlicht nicht sinnvoll sind. Andere wurden einfach noch nicht getestet (wir sind dabei).

Um die in Docker verfügbare Dokumentation nicht zu duplizieren, hier ein Link zu den Docker-Verwaltungs-APIs. Die exemplarischen Vorgehensweisen sind fantastisch.

In unserem Dokument zu den laufenden Arbeiten verfolgen wir die Dinge nach, die in Docker-APIs funktionieren bzw. nicht funktionieren.



