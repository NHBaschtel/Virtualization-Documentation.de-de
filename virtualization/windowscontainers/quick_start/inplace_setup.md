



# Bereitstellen eines Windows-Containerhosts auf einem vorhandenen virtuellen oder physischen System

In diesem Dokument werden die Schritte zum Bereitstellen und Konfigurieren der Windows-Containerrolle auf einem vorhandenen physischen oder virtuellen System mithilfe eines PowerShell-Skripts vorgestellt.

Unter [Neuer Hyper-V-Windows-Containerhost](./container_setup.md) finden Sie Informationen zur skriptgesteuerten Bereitstellung eines neuen virtuellen Hyper-V-Computers, der als Windows-Containerhost konfiguriert wird.

**VOR DEM INSTALLIEREN DES CONTAINERBETRIEBSSYSTEM-IMAGES BITTE LESEN:** Die Lizenzbedingungen für die Microsoft Windows Server-Pre-Release-Software („Lizenzbedingungen“) gelten für Ihre Nutzung der Zusatzsoftware in Form des Microsoft Windows-Containerbetriebssystem-Images (der „Zusatzsoftware“). Durch Herunterladen und verwenden die Zusatzsoftware, stimmen Sie dem Lizenzvertrag und können Sie nicht verwenden, wenn Sie dem Lizenzvertrag nicht akzeptiert haben. Die Windows Server-Pre-Release-Software und Zusatzsoftware werden von der Microsoft Corporation lizenziert.

Für die Übungen zu Windows Server- und Hyper-V-Containern in dieser Schnellstartanleitung ist Folgendes erforderlich:

* System mit Windows Server Technical Preview 4 oder höher
* 10 GB Speicherplatz für Containerhostimage, Betriebssystem-Basis-Image und Setupskripts
* Administratorberechtigungen auf dem System

## Einrichten eines vorhandenen VM- oder Bare-Metal-Hosts für Container

Windows-Container benötigen die Basis-Images für Containerbetriebssysteme. Wir haben ein Skript zusammengestellt, mit diesen Hilfe diese Elemente für Sie heruntergeladen und installiert werden. Führen Sie diese Schritte aus, um Ihr System als Windows-Containerhost zu konfigurieren. Weitere Informationen finden Sie unter „Neuerungen bei Hyper-V“ in [Windows Server 2016 Technical Preview](https://tnstage.redmond.corp.microsoft.com/en-US/library/dn765471.aspx#BKMK_nested).

Starten einer PowerShell-Sitzungs als Administrator an. Dies kann erfolgen, indem Sie den folgenden Befehl von der Befehlszeile aus ausführen.

``` powershell
PS C:\> powershell.exe
```

Stellen Sie sicher, dass der Titel des Windows "Administrator: Windows PowerShell". Wenn sie Administrator nicht dazu aufgefordert werden, führen Sie diesen Befehl mit Admin-Berechtigungen ausgeführt:

``` powershell
PS C:\> start-process powershell -Verb runas
```

Verwenden Sie den folgenden Befehl aus, um das Setup-Skript herunterzuladen. Das Skript kann auch von dieser Adresse ([Konfigurationsskript](https://aka.ms/tp4/Install-ContainerHost)) manuell heruntergeladen werden.

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/Install-ContainerHost -OutFile C:\Install-ContainerHost.ps1
```

 Nachdem der Download abgeschlossen ist, führen Sie das Skript.
``` PowerShell
PS C:\> powershell.exe -NoProfile C:\Install-ContainerHost.ps1 -HyperV
```

Das Skript beginnt dann mit dem Herunterladen und Konfigurieren der Windows Container-Komponenten. Aufgrund des großen Downloads kann der Prozess einige Zeit dauern. Der Computer wird möglicherweise während des Vorgangs neu gestartet. Im Anschluss wird der Computer konfiguriert und ist anschließend bereit für das Erstellen und Verwalten von Windows-Containern und Windows-Containerimages mit sowohl PowerShell als auch Docker.

 Nach Abschluss dieser Schritte ist Ihr System für das Arbeiten mit Windows-Containern bereit.

## Nächste Schritte: Nutzen von Containern

Nun da Sie über ein Windows Server 2016-System mit ausgeführtem Feature „Windows-Container“ verfügen, können Sie in den folgenden Anleitungen erfahren, wie Sie mit Windows Server- und Hyper-V-Containern arbeiten.

[Schnellstart: Windows-Container und Docker](./manage_docker.md)

[Schnellstart: Windows-Container und PowerShell](./manage_powershell.md)






<!--HONumber=Feb16_HO3-->


