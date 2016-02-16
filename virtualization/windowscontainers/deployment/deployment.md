# Bereitstellung von Containerhosts – Windows Server

**Dieser Inhalt ist vorläufig und kann geändert werden.**

Für die Bereitstellung eines Windows-Containerhosts sind je nach Betriebssystem und Typ des Hostsystems (physisch oder virtuell) unterschiedliche Schritte erforderlich. Die Schritte in diesem Dokument dienen dazu, Windows Server 2016 oder Windows Server Core 2016 auf einem physischen oder virtuellen System einen Windows-Containerhost bereitzustellen. Informationen zum Installieren eines Windows-Containerhosts auf Nano Server finden Sie unter [Bereitstellung von Containerhosts – Nano Server](./deployment_nano.md).

Einzelheiten zu Systemanforderungen finden Sie unter [Systemanforderungen von Windows-Containerhosts](./system_requirements.md).

Es stehen auch PowerShell-Skripts zum Automatisieren der Bereitstellung eines Windows-Containerhosts zur Verfügung.
- [Bereitstellen eines Containerhosts auf einem neuen virtuellen Hyper-V-Computer](../quick_start/container_setup.md).
- [Bereitstellen eines Containerhost auf einem vorhandenen System](../quick_start/inplace_setup.md).

# Windows Server-Host

Anhand der Schritte in dieser Tabelle kann ein Containerhost unter Windows Server 2016 und Windows Server 2016 Core bereitgestellt werden. Enthalten sind die für Windows Server- und Hyper-V-Container erforderlichen Konfigurationen.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>Bereitstellungsaktion</strong></td>
<td width="70%"><strong>Details</strong></td>
</tr>
<tr>
<td>[Installieren des Features „Container“](#role)</td>
<td>Das Feature „Container“ ermöglicht die Verwendung von Windows Server- und Hyper-V-Containern.</td>
</tr>
<tr>
<td>[Erstellen des virtuellen Switches](#vswitch)</td>
<td>Container verbinden sich über einen virtuellen Switch mit dem Netzwerk.</td>
</tr>
<tr>
<td>[Konfigurieren von NAT](#nat)</td>
<td>Wenn ein virtueller Switch mit Netzwerkadressübersetzung (Network Address Translation, NAT) konfiguriert ist, muss NAT selbst konfiguriert werden.</td>
</tr>
<tr>
<td>[Installieren von Containerbetriebssystem-Images](#img)</td>
<td>Betriebssystemimages bilden die Basis für Containerbereitstellungen.</td>
</tr>
<tr>
<td>[Installieren von Docker](#docker)</td>
<td>Optional, jedoch erforderlich, um Windows-Container mit Docker zu erstellen und zu verwalten.</td>
</tr>
</table>

Bei Verwendung von Hyper-V-Containern müssen diese Schritte ausgeführt werden. Beachten Sie, dass die mit * gekennzeichneten Schrittes nur erforderlich sind, wenn es sich beim Containerhost selbst um einen virtuellen Hyper-V-Computer handelt.

<table border="1" style="background-color:FFFFCC;border-collapse:collapse;border:1px solid FFCC00;color:000000;width:100%" cellpadding="5" cellspacing="5">
<tr valign="top">
<td width="30%"><strong>Bereitstellungsaktion</strong></td>
<td width="70%"><strong>Details</strong></td>
</tr>
<tr>
<td>[Aktivieren der Hyper-V-Rolle](#hypv) </td>
<td>Hyper-V ist nur erforderlich, wenn Hyper-V-Container verwendet werden.</td>
</tr>
<tr>
<td>[Aktivieren der geschachtelten Virtualisierung *](#nest)</td>
<td>Wenn der Containerhost selbst ein virtueller Hyper-V-Computer ist, muss „Geschachtelte Virtualisierung“ aktiviert werden.</td>
</tr>
<tr>
<td>[Konfigurieren virtueller Prozessoren *](#proc)</td>
<td>Wenn der Containerhost selbst ein virtueller Hyper-V-Computer ist, müssen mindestens zwei virtuelle Prozessoren konfiguriert werden.</td>
</tr>
<tr>
<td>[Deaktivieren des dynamischen Speichers *](#dyn)</td>
<td>Wenn der Containerhost selbst ein virtueller Hyper-V-Computer ist, muss dynamischer Arbeitsspeicher deaktiviert werden.</td>
</tr>
<tr>
<td>[Konfigurieren des Spoofings von MAC-Adressen *](#mac)</td>
<td>Wenn der Containerhost virtualisiert ist, muss das Spoofing von MAC-Adressen aktiviert werden.</td>
</tr>
</table>

## Bereitstellungsschritte

### <a name=role></a>Installieren des Features „Container“

Das Feature „Container“ kann unter Windows Server 2016 oder Windows Server 2016 Core mithilfe von Windows Server-Manager oder PowerShell installiert werden.

Um die Rolle mithilfe von PowerShell zu installieren, führen Sie den folgenden Befehl in einem PowerShell-Sitzung mit erhöhten Rechten aus.

```powershell
PS C:\> Install-WindowsFeature containers
```
Das System muss neu gestartet werden, nachdem die Containerrolleninstallation abgeschlossen wurde.

```powershell
PS C:\> shutdown /r 
```
Führen Sie nach dem Neustart des Systems den Befehl `Get-ContainerHost` aus, um sicherzustellen, dass die Containerrolle erfolgreich installiert wurde:

```powershell
PS C:\> Get-ContainerHost

Name            ContainerImageRepositoryLocation
----            --------------------------------
WIN-LJGU7HD7TEP C:\ProgramData\Microsoft\Windows\Hyper-V\Container Image Store
```

### <a name=vswitch></a>Erstellen des virtuellen Switches

Jeder Container muss an einen virtuellen Switch angefügt werden, um über ein Netzwerk zu kommunizieren. Ein virtueller Switch wird mit dem Befehl `New-VMSwitch` erstellt. Container unterstützen einen virtuellen Switch vom Typ `Extern` oder `NAT`. Weitere Informationen zu Windows-Containernetzwerken finden Sie unter [Containernetzwerke](../management/container_networking.md).

In diesem Beispiel wird ein virtueller Switch mit dem Namen „Virtual Switch“ mit dem Typ NAT und dem NAT-Subnetz 172.16.0.0/12 erstellt.

```powershell
PS C:\> New-VMSwitch -Name "Virtual Switch" -SwitchType NAT -NATSubnetAddress 172.16.0.0/12
```

### <a name=nat></a>Konfigurieren von NAT

Zusätzlich zur Erstellung des virtuellen Switches muss, wenn der Switchtyp NAT ist, ein NAT-Objekt erstellt werden. Dies erfolgt über den Befehl `New-NetNat`. In diesem Beispiel werden ein NAT-Objekt mit dem Namen `ContainerNat` und ein Adresspräfix erstellt, das dem NAT-Subnetz entspricht, das dem Containerswitch zugewiesen ist.

```powershell
PS C:\> New-NetNat -Name ContainerNat -InternalIPInterfaceAddressPrefix "172.16.0.0/12"

Name                             : ContainerNat
ExternalIPInterfaceAddressPrefix :
InternalIPInterfaceAddressPrefix : 172.16.0.0/12
IcmpQueryTimeout                 : 30
TcpEstablishedConnectionTimeout  : 1800
TcpTransientConnectionTimeout    : 120
TcpFilteringBehavior             : AddressDependentFiltering
UdpFilteringBehavior             : AddressDependentFiltering
UdpIdleSessionTimeout            : 120
UdpInboundRefresh                : False
Store                            : Local
Active                           : True
```

### <a name=img></a>Installieren von Betriebssystemimages

Ein Betriebssystemimage dient als Basis aller Windows Server- oder Hyper-V-Container. Das Image wird zum Bereitstellen eines Containers verwendet, der anschließend geändert und in einem neuen Containerimage erfasst werden kann. Betriebssystemimages werden mit Windows Server Core und Nano Server als zugrunde liegendem Betriebssystem erstellt.

Containerbetriebssystem-Images können mithilfe des PowerShell-Moduls „ContainerProvider“ bestimmt und installiert werden. Sie müssen dieses Modul erst installieren, um es verwenden zu können. Installieren Sie das Modul über die folgenden Befehle.

```powershell
PS C:\> Install-PackageProvider ContainerProvider -Force
```

Verwenden Sie `Find-ContainerImage`, um eine Liste der Images aus dem PowerShell OneGet-Paket-Manager zurückzugeben:
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

```powershell
PS C:\> Install-ContainerImage -Name WindowsServerCore -Version 10.0.10586.0

Downloaded in 0 hours, 2 minutes, 28 seconds.
```

**Problem:** Die Cmdlets „Save-ContainerImage“ und „Install-ContainerImage“ funktionieren in einer PowerShell-Remotesitzung nicht mit einem WindowsServerCore-Containerimage.<br /> **Abhilfe**: Melden Sie sich über Remotedesktop am Computer an, und verwenden Sie das Cmdlet „Save-ContainerImage“ direkt.

Überprüfen Sie mit dem Befehl `Get-ContainerImage`, ob die Images Installiert wurden.

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```
Weitere Informationen zur Verwaltung von Containerimages finden Sie unter [Windows-Containerimages](../management/manage_images.md).


### <a name=docker></a>Installieren von Docker

Der Docker-Daemon und die Befehlszeilenschnittstelle gehören nicht zum Funktionsumfang von Windows und werden nicht mit dem Feature „Windows-Container“ installiert. Docker ist keine Voraussetzung für das Arbeiten mit Windows-Containern. Wenn Sie Docker installieren möchten, befolgen Sie die Anweisungen im Artikel [Docker und Windows](./docker_windows.md).


## Hyper-V-Containerhost

### <a name=hypv></a>Aktivieren der Hyper-V-Rolle

Bei der Bereitstellung von Hyper-V-Containern muss die Hyper-V-Rolle auf dem Containerhost aktiviert werden. Die Hyper-V-Rolle kann unter Windows Server 2016 oder Windows Server 2016 Core mithilfe des Befehls `Install-WindowsFeature` installiert werden. Wenn der Containerhost selbst ein virtueller Hyper-V-Computer ist, muss zuerst die geschachtelte Virtualisierung aktiviert werden. Lesen Sie dazu [Konfigurieren der geschachtelten Virtualisierung](#nest).

```powershell
PS C:\> Install-WindowsFeature hyper-v
```

### <a name=nest></a>Konfigurieren der geschachtelten Virtualisierung

Wenn der Containerhost selbst auf einem virtuellen Hyper-V-Computer ausgeführt wird und auch Hyper-V-Container hostet, muss die geschachtelte Virtualisierung aktiviert werden. Dies kann mithilfe des folgenden PowerShell-Befehls erreicht werden:

**Hinweis**: Beim Ausführen dieses Befehls muss der virtuelle Computer ausgeschaltet sein.

```powershell
PS C:\> Set-VMProcessor -VMName <VM Name> -ExposeVirtualizationExtensions $true
```

### <a name=proc></a>Konfigurieren virtueller Prozessoren

Wenn der Containerhost selbst auf einem virtuellen Hyper-V-Computer ausgeführt wird und auch Hyper-V-Container hostet, erfordert der virtuelle Computer mindestens zwei Prozessoren. Dies kann über die Einstellungen des virtuellen Computers oder mit dem folgenden Befehl konfiguriert werden.

**Hinweis**: Beim Ausführen dieses Befehls muss der virtuelle Computer ausgeschaltet sein.

```poweshell
PS C:\> Set-VMProcessor –VMName <VM Name> -Count 2
```

### <a name=dyn></a>Deaktivieren des dynamischen Speichers

Wenn der Containerhost selbst ein virtueller Hyper-V-Computer ist, muss auf dem virtuellen Containerhostcomputer der dynamische Arbeitsspeicher deaktiviert werden. Dies kann über die Einstellungen des virtuellen Computers oder mit dem folgenden Befehl konfiguriert werden.

**Hinweis**: Beim Ausführen dieses Befehls muss der virtuelle Computer ausgeschaltet sein.

```poweshell
PS C:\> Set-VMMemory <VM Name> -DynamicMemoryEnabled $false
```

### <a name=mac></a>Konfigurieren des Spoofings von MAC-Adressen

Wenn schließlich der Containerhost in einem virtuellen Hyper-V-Computer ausgeführt wird, muss das Spoofing von MAC-Adressen aktiviert werden. Dadurch kann jeder Container eine IP-Adresse erhalten. Um das Spoofing von MAC-Adressen zu aktivieren, führen Sie auf dem Hyper-V-Host den folgenden Befehl aus. Die „VMName“-Eigenschaft entspricht dem Namen des Containerhosts.

```powershell
PS C:\> Get-VMNetworkAdapter -VMName <VM Name> | Set-VMNetworkAdapter -MacAddressSpoofing On
```





<!--HONumber=Feb16_HO1-->
