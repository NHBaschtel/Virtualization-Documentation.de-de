
# <a name="using-insider-container-images"></a>Verwenden von Insider-Containerimages

Diese Übung führt Sie durch die Bereitstellung und Verwendung der Windows-Containerfunktionen der neuesten Insider-Build von Windows Server aus dem Windows Insider Preview-Programm. In dieser Übung installieren Sie die Containerrolle und stellen eine Preview-Edition der Basisbetriebssystemimage bereit. Wenn Sie sich mit Containern vertraut machen möchten, finden Sie unter [Windows-Container](../about/index.md) entsprechende Informationen.

Dieser Schnellstart ist spezifisch für Windows Server-Container im Windows Server Insider Preview-Programm. Bitte machen Sie sich mit dem Programm vertraut, bevor Sie mit diesem Schnellstart fortfahren.

**Voraussetzungen:**

- Nehmen Sie am [Windows-Insider-Programm](https://insider.windows.com/GettingStarted) teil und überprüfen Sie die Nutzungsbedingungen.
- Ein Computersystem (physisch oder virtuell) mit dem neuesten Builds von Windows Server aus dem Windows-Insider-Programm bzw. dem neuesten Build von Windows10 aus dem Windows-Insider-Programm.

>Sie müssen einen Build von Windows Server aus dem Windows Server Insider Preview-Programm oder einen Build von Windows10 aus dem Windows Insider Preview-Programm verwenden, um das unten beschriebene Basisimage verwenden zu können. Wenn Sie keine dieser Builds verwenden, kann der Container nicht mit diesen Basisimages gestartet werden.

## <a name="install-docker-enterprise-edition-ee"></a>Installieren der Docker Enterprise Edition (EE)
Für die Arbeit mit Windows-Containern ist Docker EE erforderlich. Docker EE besteht aus dem Docker-Modul und dem Docker-Client. 

Verwenden Sie das PowerShell-Modul von OneGet, um Docker EE zu installieren. Der Anbieter aktiviert die Containerfunktion auf Ihrem Computer und installiert Docker EE. Dazu ist ein Neustart erforderlich. Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und führen Sie die folgenden Befehle aus.

>Hinweis: Zum Installieren von Docker EE mit Windows Server-Insider-Builds ist ein anderer OneGet-Anbieter erforderlich als der für Nicht-Insider-Builds vorgesehene. Wenn Docker EE und der OneGet-Anbieter DockerMsftProvider bereits installiert sind, entfernen Sie diese Komponenten, bevor Sie fortfahren.
```powershell 
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

Installieren Sie das PowerShell-Modul OneGet für die Verwendung mit Windows-Insider-Builds.
```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```
Verwenden Sie OneGet, um die neueste Vorschauversion von Docker EE zu installieren.
```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```
Wenn die Installation abgeschlossen ist, starten Sie den Computer neu.
```
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>Installieren von Basiscontainerimages

Vor der Arbeit mit Windows-Containern muss ein Basisimage installiert werden. Wenn Sie am Windows-Insider-Programms teilnehmen, können Sie ebenfalls unsere neuesten Builds für die Basisimages testen. Unter den Insider-Basisimages stehen jetzt 4 auf Windows Server basierende Basisimages zur Verfügung. Weitere Informationen zu den verschiedenen Zwecken der einzelnen Basisimages finden Sie in der Tabelle unten:

| Basisbetriebssystemimages                       | Verwendungszweck                      |
|-------------------------------------|----------------------------|
| microsoft/windowsservercore         | Produktion und Entwicklung |
| microsoft/nanoserver                | Produktion und Entwicklung |
| microsoft/windowsservercore-insider | Nur Entwicklung           |
| microsoft/nanoserver-insider        | Nur Entwicklung           |

Zum Aufrufen des Basisimages für Nano Server Insider führen Sie folgenden Befehl aus:

```
docker pull microsoft/nanoserver-insider
```

Zum Aufrufen des Basisimages für Windows Server Core Insider führen Sie folgenden Befehl aus:

```
docker pull microsoft/windowsservercore-insider
```

Bitte lesen Sie den [Endbenutzer-Lizenzvertrag](../EULA.md ) zum Betriebssystemimage für Windows-Container durch. Die Nutzungsbedingungen für das Windows-Insider-Programm befinden sich unter [Nutzungsbedingungen](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewserver).

## <a name="next-steps"></a>Nächste Schritte

[Erstellen und Ausführen einer Anwendung mit oder ohne .NET Core 2.0 oder PowerShell Core 6](./Nano-RS3-.NET-Core-and-PS.md)
