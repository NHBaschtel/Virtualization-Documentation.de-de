---
author: neilpeterson
---

# Azure – Schnellstart

Vor dem Erstellen und Verwalten von Windows Server-Containern in Azure müssen Sie ein Image von Windows Server 2016 Technical Preview bereitstellen, das mit dem Feature „Windows Server-Container“ vorkonfiguriert wurde. In dieser Anleitung begleiten wir Sie durch diesen Prozess.

> Microsoft Azure unterstützt keine Hyper-V-Container. Für Übungen mit Hyper-V-Containern benötigen Sie einen lokalen Containerhost.

## Starten Sie mithilfe von Azure-Portal

Wenn Sie ein Azure-Konto haben, fahren Sie mit [Erstellen einer Containerhost-VM](#CreateacontainerhostVM) fort.

1. Wechseln Sie zu [azure.com](https://azure.com), und befolgen Sie die Anleitung zum Beziehen einer [kostenlosen Azure-Testversion](https://azure.microsoft.com/en-us/pricing/free-trial/).
2. Melden Sie sich mit Ihrem Microsoft-Konto.
3. Wenn Ihr Konto einsatzbereit ist, melden Sie sich beim [Azure-Verwaltungsportal](https://portal.azure.com) an.

## Erstellen Sie einen Container-Host-VM

Suchen Sie in Azure Marketplace nach „Container“. Als Ergebnis erhalten Sie „Windows Server 2016 Core with Containers Tech Preview 4“.

![](./media/newazure1.png)

Wählen Sie das Image aus, und klicken Sie auf `Erstellen`.

![](./media/tp41.png)

Geben Sie den virtuellen Computer einen Namen ein, wählen Sie einen Benutzernamen und ein Kennwort.

![](media/newazure2.png)

Wählen Sie die optionale Konfiguration > Endpunkte >, und geben Sie einen HTTP-Endpunkt mit einem privaten und öffentlichen Port 80, wie unten dargestellt. Wenn Fertig klicken Sie auf "ok" zweimal.

![](./media/newazure3.png)

Klicken Sie auf die Schaltfläche `Erstellen`, um den Bereitstellungsprozess für den virtuellen Computer zu starten.

![](media/newazure2.png)

Wenn die VM-Bereitstellung abgeschlossen ist, wählen Sie die Schaltfläche "Verbinden", um eine RDP-Sitzung mit dem Windows Server-Host-Container zu starten.

![](media/newazure6.png)

Melden Sie sich auf dem virtuellen Computer mit dem Benutzernamen und Kennwort angegeben werden, während der Assistent zum Erstellen von VM. Nach der Anmeldung in Suchen Sie in einer Windows-Befehlszeile.

![](media/newazure7.png)

## Aktualisieren des Docker-Moduls

Zur Verwendung von `docker pull` mit dem Azure Windows Container Technical Preview-Image, muss das Docker-Modul aktualisiert werden. Führen Sie die folgenden PowerShell-Befehle auf dem virtuellen Azure-Computer aus, um das Update abzuschließen.

```powershell
PS C:\> wget https://raw.githubusercontent.com/Microsoft/Virtualization-Documentation/live/windows-server-container-tools/Update-ContainerHost/Update-ContainerHost.ps1 -OutFile Update-ContainerHost.ps1

PS C:\> ./Update-ContainerHost.ps1
```

## Video zur exemplarischen Vorgehensweise

<iframe src="https://channel9.msdn.com/Blogs/containers/Quick-Start-Configure-Windows-Server-Containers-in-Microsoft-Azure/player#ccLang=de" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>


## Nächste Schritte: Nutzen von Containern

Nun da Sie über ein Windows Server 2016-System mit ausgeführtem Feature „Windows Server-Container“ verfügen, können Sie in den folgenden Anleitungen erfahren, wie Sie mit Windows Server-Containern und -Images arbeiten.

[Schnellstart: Windows-Container und Docker](./manage_docker.md)  
[Schnellstart: Windows-Container und PowerShell](./manage_powershell.md)



<!--HONumber=Mar16_HO3-->
