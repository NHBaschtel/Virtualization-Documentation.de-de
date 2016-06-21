---
author: scooley
redirect_url: ../quick_start/manage_docker
---


# Vergleichen der Verwaltung von Windows-Containern mit PowerShell und Docker

Es gibt viele Methoden zum Verwalten von Windows-Containern sowohl mit integrierten Windows-Tools (in dieser Vorschau PowerShell) als auch Open-Source-Verwaltungstools wie Docker.  
Führungslinien Gliedern beide einzeln erhältlich hier:
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Verwalten von Windows-Containern mit Docker</g><g id="1CapsExtId3" ctype="x-title"></g></g>
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Verwalten von Windows-Containern mit PowerShell</g><g id="1CapsExtId3" ctype="x-title"></g></g>

Diese Seite ist einem mehr Tiefe Referenz Vergleich von Docker-Tools und PowerShell-Verwaltungstools.

## PowerShell für Container im Vergleich zu Hyper-V-VMs

Sie können Windows-Container mithilfe von PowerShell-Cmdlets erstellen, ausführen und damit interagieren. Alles, was Sie brauchen, ist standardmäßig verfügbar.

Wenn Sie Hyper-V-PowerShell verwendet haben, sollte das Design der Cmdlets Recht vertraut sein. Ein Großteil der Workflow ist ähnlich wie eine virtuelle Maschine mit dem Hyper-V-Modul verwaltet. Anstelle von <g id="2" ctype="x-code">New-VM</g>, <g id="4" ctype="x-code">Get-VM</g>, <g id="6" ctype="x-code">Start-VM</g> und <g id="8" ctype="x-code">Stop-VM</g> verwenden Sie <g id="10" ctype="x-code">New-Container</g>, <g id="12" ctype="x-code">Get-Container</g>, <g id="14" ctype="x-code">Start-Container</g> und <g id="16" ctype="x-code">Stop-Container</g>. Es gibt einige Container-spezifische Cmdlets und Parameter, aber die allgemeine Lebenszyklus und Verwaltung von Windows-Container sieht in etwa wie eine Hyper-V-VM.

## Wie Vergleich PowerShell Management im zu Docker?

Die Container-PowerShell-Cmdlets verfügbar machen eine API, die nicht ganz identisch mit der Docker ist. die Cmdlets sind in der Regel genauer Vorgang. Einige Befehle Docker sind unkompliziert Parallels in PowerShell:

| Docker-Befehl| PowerShell-Cmdlets|
|----|----|
| <g id="1" ctype="x-code">docker ps -a</g>| <g id="1" ctype="x-code">Get-Container</g>|
| <g id="1" ctype="x-code">docker images</g>| <g id="1" ctype="x-code">Get-ContainerImage</g>|
| <g id="1" ctype="x-code">docker rm</g>| <g id="1" ctype="x-code">Remove-Container</g>|
| <g id="1" ctype="x-code">docker rmi</g>| <g id="1" ctype="x-code">Remove-ContainerImage</g>|
| <g id="1" ctype="x-code">docker create</g>| <g id="1" ctype="x-code">New-Container</g>|
| <g id="1" ctype="x-code">docker commit <Container-ID></g>| <g id="1" ctype="x-code">New-ContainerImage -Container &lt;Container&gt;</g>|
| <g id="1" ctype="x-code">docker load &lt;tarball&gt;</g>| <g id="1" ctype="x-code">Import-ContainerImage <AppX-Paket></g>|
| <g id="1" ctype="x-code">docker save</g>| <g id="1" ctype="x-code">Export-ContainerImage</g>|
| <g id="1" ctype="x-code">docker start</g>| <g id="1" ctype="x-code">Start-Container</g>|
| <g id="1" ctype="x-code">docker stop</g>| <g id="1" ctype="x-code">Stop-Container</g>|

Die PowerShell-Cmdlets sind keine exakten perfekten Gegenstücke. Es gibt ziemlich viele Befehle, für die wir keine PowerShell-Entsprechung bieten* (insbesondere <g id="2" ctype="x-code">docker build</g> und <g id="4" ctype="x-code">docker cp</g>). Es wird Ihnen jedoch möglicherweise auffallen, dass es keinen einzeiligen Ersatz für <g id="2" ctype="x-code">docker run</g> gibt.

\ * Vorbehalten.

### Aber ich Docker ausführen! Was ist passiert?

Wir machen, ein paar Dinge, die eine etwas vertrauter Interaktionsmodell für Benutzer bereitstellen, die ihre PowerShell bereits vertraut sind. Wenn Sie die Art und Weise Docker arbeitet verwendet, wird dies natürlich etwas ein Umdenken sein.

1.  Der Lebenszyklus eines Containers im PowerShell-Modell ist etwas anders. Im PowerShell-Modul für Container machen wir die differenzierteren Vorgänge <g id="2" ctype="x-code">New-Container</g> (wodurch ein neuer Container erstellt wird, der beendet wird) und <g id="4" ctype="x-code">Start-Container</g> verfügbar.

  Zwischen erstellen, und starten den Container, können Sie auch den Container Einstellungen konfigurieren. für TP3 ist nur anderen Konfigurationen, die wir bereitstellen möchten die Möglichkeit, die Netzwerkschnittstelle für den Container festlegen. verwenden die (hinzufügen/entfernen/Verbindung herstellen/trennen/Get/Set)-ContainerNetworkAdapter-Cmdlets.

2.  Derzeit kann keinen auszuführenden Befehl innerhalb des Containers auf "Start" übergeben werden. Sie können allerdings weiter eine interaktive PowerShell-Sitzung mit einem ausgeführten Container einrichten. Verwenden Sie dazu <g id="2" ctype="x-code">Enter-PSSession -ContainerId <ID eines ausgeführten Containers)</g>. Sie können auch einen Befehl in einem ausgeführten Container ausführen. Verwenden Sie dazu <g id="4" ctype="x-code">Invoke-Command -ContainerId <Container-ID> -ScriptBlock {im Container auszuführender Code}</g> oder <g id="6" ctype="x-code">Invoke-Command -ContainerId <Container-ID> -FilePath <Pfad zum Skript></g>.  
Beide Befehle lassen das optionale Kennzeichen <g id="2" ctype="x-code">-RunAsAdministrator</g> für Aktionen mit höheren Berechtigungen zu.


## Einschränkungen und bekannte Probleme

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

Mit dem Befehl <g id="2" ctype="x-code">Get-Command -Module Containers</g> können Sie alle Cmdlets für Container anzeigen. Es gibt mehrere andere Cmdlets, die hier nicht beschrieben werden, die wir Ihnen Informationen auf Ihren eigenen lassen.    
<g id="1" ctype="x-strong">Hinweis</g> Die Cmdlets <g id="3" ctype="x-code">Enter-PSSession</g> und <g id="5" ctype="x-code">Invoke-Command</g>, die Teil der wichtigsten PowerShell-Cmdlets sind, werden dadurch nicht zurückgegeben.

Über <g id="2" ctype="x-code">Get-Help [Name des Cmdlets]</g> oder <g id="4" ctype="x-code">[Name des Cmdlets] -?</g> erhalten Sie Hilfe zu sämtlichen Cmdlets. Heute Hilfe wird automatisch generiert und teilt Ihnen nur die Syntax für Befehle. Wir werden weitere Dokumentation hinzufügen werden wie wir näher erhalten an das Cmdlet-Design abschließen.

Eine nützlicher Möglichkeit, die Syntax zu ermitteln ist PowerShell ISE, die Sie nicht vor dem besprochen haben können, wenn Sie PowerShell sehr viel verwendet haben. Wenn Sie auf eine SKU, die es ermöglicht ausgeführt, starten Sie das Befehlsfenster zu öffnen, und wählen im Modul "Container" der grafisch dargestellt, die Cmdlets und ihren Parameter angezeigt, für die ISE.

PS: Um nachzuweisen, dass es möglich ist, hier nun eine PowerShell-Funktion, die aus einigen der Cmdlets besteht, die wir bereits als Ersatz für <g id="2" ctype="x-code">docker run</g> gesehen haben. (Klar gesagt werden, ist dies eine Machbarkeitsstudie nicht in der aktiven Entwicklung.)

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

Windows-Container können mit Docker-Befehlen verwaltet werden. Während Windows-Container mit ihren Linux-Entsprechungen vergleichbar sein sollten und über Docker gleich verwaltet werden sollten, gibt es einige Docker-Befehle, die für einen Windows-Container schlicht nicht sinnvoll sind. Andere einfach noch nicht getestet wurden (wir gehen vorhanden).

In dem Bestreben, die in Docker verfügbare API-Dokumentation nicht zu duplizieren, finden Sie <g id="2" ctype="x-html"></g>hier<g id="4" ctype="x-html"></g> einen Link zu Ihren Verwaltungs-APIs. Die exemplarischen Vorgehensweisen sind fantastisch.

Wir überwachen die Dinge, die funktionieren und nicht in die Docker-APIs in unser Dokument In Bearbeitung.






<!--HONumber=Apr16_HO4-->


