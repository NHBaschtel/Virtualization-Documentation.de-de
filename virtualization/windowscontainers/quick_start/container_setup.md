# Bereitstellen eines Windows-Containerhosts auf einem neuen virtuellen Hyper-V-Computer

In diesem Dokument werden die Schritte zum Verwenden eines PowerShell-Skripts zum Bereitstellen eines neuen virtuellen Hyper-V-Computers vorgestellt, der anschließend als Windows-Containerhost konfiguriert wird.

Unter [Direkte Bereitstellung von Windows-Containerhosts](./inplace_setup.md) finden Sie Informationen zur skriptgesteuerten Bereitstellung eines Windows-Containerhosts auf einem virtuellen oder physischen System.

**VOR DEM INSTALLIEREN DES CONTAINERBETRIEBSSYSTEM-IMAGES BITTE LESEN:** Die Lizenzbedingungen für die Microsoft Windows Server-Pre-Release-Software („Lizenzbedingungen“) gelten für Ihre Nutzung der Zusatzsoftware in Form des Microsoft Windows-Containerbetriebssystem-Images (der „Zusatzsoftware“). Durch Herunterladen und Verwenden der Zusatzsoftware stimmen Sie den Lizenzbedingungen zu. Eine Nutzung ist nur gestattet, wenn Sie den Lizenzbedingungen zugestimmt haben. Die Windows Server-Pre-Release-Software und Zusatzsoftware werden von der Microsoft Corporation lizenziert.

Für die Übungen zu **Windows Server-Containern** und **Hyper-V-Containern** in dieser Schnellstartanleitung ist Folgendes erforderlich:

* System mit Windows 10 Build 10586 oder höher/Windows Server Technical Preview 4 oder höher.
* Aktivierte Hyper-V-Rolle ([siehe Anweisungen](https://msdn.microsoft.com/virtualization/hyperv_on_windows/quick_start/walkthrough_install#UsingPowerShell)))
* 20 GB Speicherplatz für Containerhostimage, Betriebssystem-Basis-Image und Setupskripts
* Administratorberechtigungen auf dem Hyper-V-Host

> Ein virtualisierter Containerhost mit ausgeführten Hyper-V-Containern erfordert die geschachtelte Virtualisierung. Auf dem physischen und virtuellen Host muss ein Betriebssystem ausgeführt werden, das die geschachtelte Virtualisierung unterstützt. Weitere Informationen finden Sie unter „Neuerungen bei Hyper-V“ in [Windows Server 2016 Technical Preview](https://technet.microsoft.com/library/dn765471.aspx#BKMK_nested).

## Einrichten eines neuen Containerhosts auf einem neuen virtuellen Computer

Windows-Container bestehen aus mehreren Komponenten, z. B. Windows-Containerhost und Basis-Images für Containerbetriebssysteme. Wir haben ein Skript zusammengestellt, mit diesen Hilfe diese Elemente für Sie heruntergeladen und konfiguriert werden. Befolgen Sie diese Schritte zum Bereitstellen eines neuen virtuellen Hyper-V-Computers und Konfigurieren dieses Systems als Windows-Containerhost.

Starten Sie eine PowerShell-Sitzung als Administrator. Klicken Sie dazu mit der rechten Maustaste auf das PowerShell-Symbol, und wählen Sie „Als Administrator ausführen“ aus. Oder führen Sie in einer PowerShell-Sitzung den folgenden Befehl aus.

``` powershell
PS C:\> start-process powershell -Verb runAs
```

Stellen Sie vor dem Herunterladen und Ausführen des Skripts sicher, dass ein externer virtueller Hyper-V-Switch erstellt wurde. Falls nicht, funktioniert dieses Skript nicht.

Führen Sie Folgendes aus, um eine Liste externer virtueller Switches zurückzugeben. Falls nichts zurückgegeben wird, erstellen Sie einen neuen externen virtuellen Switch und fahren dann mit dem nächsten Schritt in dieser Anleitung fort.

```powershell
PS C:\> Get-VMSwitch | where {$_.SwitchType –eq “External”}
```

Verwenden Sie den folgenden Befehl, um das Konfigurationsskript herunterzuladen. Das Skript kann auch von dieser Adresse ([Konfigurationsskript](https://aka.ms/tp4/New-ContainerHost)) manuell heruntergeladen werden.

``` PowerShell
PS C:\> wget -uri https://aka.ms/tp4/New-ContainerHost -OutFile c:\New-ContainerHost.ps1
```

Führen Sie den folgenden Befehl zum Erstellen und Konfigurieren des Containerhosts aus, wobei `&lt;containerhost&gt;` der Name des virtuellen Computers ist.

``` powershell
PS C:\> c:\New-ContainerHost.ps1 –VmName <containerhost> -WindowsImage ServerDatacenterCore -Hyperv
```

Bei Start des Skripts werden Sie zur Eingabe eines Kennworts aufgefordert. Dies ist das Kennwort, das dem Administratorkonto zugewiesen wurde.

Als Nächstes werden Sie aufgefordert, die Lizenzbedingungen zu lesen und zu akzeptieren.

```
Before installing and using the Windows Server Technical Preview 4 with Containers virtual machine you must:
    1. Review the license terms by navigating to this link: http://aka.ms/tp4/containerseula
    2. Print and retain a copy of the license terms for your records.
By downloading and using the Windows Server Technical Preview 4 with Containers virtual machine you agree to such
license terms. Please confirm you have accepted and agree to the license terms.
[N] No  [Y] Yes  [?] Help (default is "N"):
```

Das Skript beginnt dann mit dem Herunterladen und Konfigurieren der Windows Container-Komponenten. Aufgrund des großen Downloads kann der Prozess einige Zeit dauern. Im Anschluss wird der virtuelle Computer konfiguriert und ist anschließend bereit für das Erstellen und Verwalten von Windows-Containern und Windows-Containerimages mit sowohl PowerShell als auch Docker.

Melden Sie sich nach Ausführung des Konfigurationsskripts am virtuellen Computer mit dem Kennwort an, das während des Konfigurationsprozesses angegeben wurde, und stellen Sie sicher, dass der virtuelle Computer eine gültige IP-Adresse hat. Nach Abschluss dieser Schritte ist Ihr System für das Arbeiten mit Windows-Containern bereit.

## Nächste Schritte: Nutzen von Containern

Nun da Sie über ein Windows Server 2016-System mit ausgeführtem Feature „Windows-Container“ verfügen, können Sie in den folgenden Anleitungen erfahren, wie Sie mit Windows Server- und Hyper-V-Containern arbeiten.

[Schnellstart: Windows-Container und PowerShell](./manage_powershell.md)  
[Schnellstart: Windows-Container und Docker](./manage_docker.md)




<!--HONumber=Jan16_HO2-->
