---
author: scooley
redirect_url: ./manage_docker
translationtype: Human Translation
ms.sourcegitcommit: 745b0b26e15ec1179e8b55cd1b3d2ab3e3fe34d2
ms.openlocfilehash: f01125a9e6f8a952ae6a82fb2d1df20bb5f6c900

---

# Vergleichen der Verwaltung von Windows-Containern mit PowerShell und Docker 

Es gibt viele Methoden zum Verwalten von Windows-Containern sowohl mit integrierten Windows-Tools (in dieser Vorschau PowerShell) als auch Open-Source-Verwaltungstools wie Docker.  
Führungslinien Gliedern beide einzeln erhältlich hier:
* [Verwalten von Windows-Containern mit Docker](../quick_start/manage_docker.md)
* [Verwalten von Windows-Containern mit PowerShell](../quick_start/manage_powershell.md) 

Diese Seite ist einem mehr Tiefe Referenz Vergleich von Docker-Tools und PowerShell-Verwaltungstools.

## PowerShell für Container im Vergleich zu Hyper-V-VMs
Sie können Windows-Container mithilfe von PowerShell-Cmdlets erstellen, ausführen und damit interagieren. Alles, was Sie brauchen, ist standardmäßig verfügbar.

Wenn Sie Hyper-V-PowerShell verwendet haben, sollte das Design der Cmdlets Recht vertraut sein. Ein Großteil der Workflow ist ähnlich wie eine virtuelle Maschine mit dem Hyper-V-Modul verwaltet. Anstelle von `New-VM`, `Get-VM`, `Start-VM`, `Stop-VM` haben Sie `New-Container`, `Get-Container`, `Start-Container`, `Stop-Container`.  Es gibt einige Container-spezifische Cmdlets und Parameter, aber die allgemeine Lebenszyklus und Verwaltung von Windows-Container sieht in etwa wie eine Hyper-V-VM.

## Wie Vergleich PowerShell Management im zu Docker? 
Die Container-PowerShell-Cmdlets verfügbar machen eine API, die nicht ganz identisch mit der Docker ist. die Cmdlets sind in der Regel genauer Vorgang. Einige Befehle Docker sind unkompliziert Parallels in PowerShell:

| Docker-Befehl |  PowerShell-Cmdlets |
|----|----|
| `docker ps -a`    | `Get-Container` |
| `docker images`   | `Get-ContainerImage` |
| `docker rm`   | `Remove-Container` |
| `docker rmi` | `Remove-ContainerImage` |
| `docker create`   | `New-Container` |
| `docker commit <container ID>` | `New-ContainerImage -Container <container>` |
| `docker load <tarball>` | `Import-ContainerImage <AppX package>` |
| `docker save` |   `Export-ContainerImage` |
| `docker start` |  `Start-Container` |
| `docker stop` |   `Stop-Container` |

Die PowerShell-Cmdlets sind keine exakten perfekten Gegenstücke. Es gibt ziemlich viele Befehle, für die keine PowerShell-Entsprechung bereitgestellt werden* (insbesondere `docker build` und `docker cp`). Es wird Ihnen jedoch möglicherweise auffallen, dass es keinen einzeiligen Ersatz für `docker run` gibt.

\* Änderungen vorbehalten.

### Aber ich Docker ausführen! Was ist passiert?  
Wir machen, ein paar Dinge, die eine etwas vertrauter Interaktionsmodell für Benutzer bereitstellen, die ihre PowerShell bereits vertraut sind. Wenn Sie die Art und Weise Docker arbeitet verwendet, wird dies natürlich etwas ein Umdenken sein.

1.  Der Lebenszyklus eines Containers im PowerShell-Modell ist etwas anders. Das PowerShell-Modul für Container stellt die differenzierteren Vorgänge `New-Container` (wodurch ein neuer Container erstellt wird, der beendet wird) und `Start-Container` zur Verfügung.
  
  Zwischen erstellen, und starten den Container, können Sie auch den Container Einstellungen konfigurieren. für TP3 ist nur anderen Konfigurationen, die wir bereitstellen möchten die Möglichkeit, die Netzwerkschnittstelle für den Container festlegen. verwenden die (hinzufügen/entfernen/Verbindung herstellen/trennen/Get/Set)-ContainerNetworkAdapter-Cmdlets.

2.  Derzeit kann keinen auszuführenden Befehl innerhalb des Containers auf "Start" übergeben werden. Allerdings können Sie mit `Enter-PSSession -ContainerId <ID of a running container>` immer noch eine interaktive PowerShell-Sitzung mit einem ausgeführten Container erhalten, und Sie können mit `Invoke-Command -ContainerId <container id> -ScriptBlock { code to run inside the container }` oder `Invoke-Command -ContainerId <container id> -FilePath <path to script>` einen Befehl innerhalb eines ausgeführten Containers ausführen.  
Beide Befehle lassen das optionale Flag `-RunAsAdministrator` für Aktionen mit höheren Berechtigungen zu.  


## Vorbehalte und bekannte Probleme
1.  Jetzt, die Container-Cmdlets haben keine Kenntnis über Container oder Bilder, die durch Docker erstellt und Docker weiß nicht, alles zu Containern und Images, die über die PowerShell erstellt. Wenn Sie in der Docker erstellt, mit der Docker verwalten; Wenn Sie es über PowerShell erstellt haben, können verwalten Sie sich über PowerShell.

2.  Wir haben eine ziemlich viel Arbeit, die wir tun, damit um der Endbenutzer – verbesserte Fehlermeldungen, bessere Fortschrittsberichte, ungültiges Ereigniszeichenfolgen usw. zu optimieren möchten. Wenn Sie eine Situation auftreten versehentlich, wo sollen Sie waren Weitere erste, oder Informationen besser, wir gerne Vorschläge zu den Foren zu senden.

## Eine schnelle runthrough
Im folgenden wird erörtert einige allgemeine Workflows.

Dabei wird vorausgesetzt, Sie ein Betriebssystemabbild-Container mit dem Namen "ServerDatacenterCore" installiert haben, und erstellt einen virtuellen Switch mit dem Namen "Virtueller Switch" (mit dem New-VMSwitch).

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

### Erstellen Sie Ihre eigenen Beispiel
Mit `Get-Command -Module Containers` können Sie alle Containers-Cmdlets anzeigen.  Es gibt mehrere andere Cmdlets, die hier nicht beschrieben werden, die wir Ihnen Informationen auf Ihren eigenen lassen.    
**Hinweis** Die Cmdlets `Enter-PSSession` und `Invoke-Command`, die zu den wichtigsten PowerShell-Cmdlets gehören, werden dadurch nicht zurückgegeben.

Außerdem können Sie mit `Get-Help [cmdlet name]` bzw. `[cmdlet name] -?` Hilfe zu jedem Cmdlet anzeigen.  Heute Hilfe wird automatisch generiert und teilt Ihnen nur die Syntax für Befehle. Wir werden weitere Dokumentation hinzufügen werden wie wir näher erhalten an das Cmdlet-Design abschließen.

Eine nützlicher Möglichkeit, die Syntax zu ermitteln ist PowerShell ISE, die Sie nicht vor dem besprochen haben können, wenn Sie PowerShell sehr viel verwendet haben. Wenn Sie auf eine SKU, die es ermöglicht ausgeführt, starten Sie das Befehlsfenster zu öffnen, und wählen im Modul "Container" der grafisch dargestellt, die Cmdlets und ihren Parameter angezeigt, für die ISE.

PS: Um nachzuweisen, dass es möglich ist, hier nun eine PowerShell-Funktion, die aus einigen der Cmdlets besteht, die Sie bereits als Ersatz für `docker run` kennen gelernt haben. (Klar gesagt werden, ist dies eine Machbarkeitsstudie nicht in der aktiven Entwicklung.)

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
Windows-Container können mit Docker-Befehlen verwaltet werden.  Während Windows-Container mit ihren Linux-Entsprechungen vergleichbar sein sollten und über Docker gleich verwaltet werden sollten, gibt es einige Docker-Befehle, die für einen Windows-Container schlicht nicht sinnvoll sind.  Andere einfach noch nicht getestet wurden (wir gehen vorhanden).

In dem Bestreben, die in Docker verfügbare API-Dokumentation nicht zu duplizieren, finden Sie <a href="https://docs.docker.com/engine/reference/commandline/cli/" >hier</a> einen Link zu Ihren Verwaltungs-APIs.  Die exemplarischen Vorgehensweisen sind fantastisch.

Wir überwachen die Dinge, die funktionieren und nicht in die Docker-APIs in unser Dokument In Bearbeitung.



<!--HONumber=Jun16_HO4-->


