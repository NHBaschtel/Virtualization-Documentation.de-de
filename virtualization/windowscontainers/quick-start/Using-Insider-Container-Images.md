
# <a name="using-insider-container-images"></a>Verwenden von Insider-Containerimages

Diese Übung führt Sie durch die Bereitstellung und Verwendung der Windows-Containerfunktionen der neuesten Insider-Build von Windows Server aus dem Windows Insider Preview-Programm. In dieser Übung installieren Sie die Containerrolle und stellen eine Preview-Edition der Basisbetriebssystemimage bereit. Wenn Sie sich mit Containern vertraut machen möchten, finden Sie unter [Windows-Container](../about/index.md) entsprechende Informationen.

Dieser Schnellstart ist spezifisch für Windows Server-Container im Windows Server Insider Preview-Programm. Bitte machen Sie sich mit dem Programm vertraut, bevor Sie mit diesem Schnellstart fortfahren.

## <a name="prerequisites"></a>Voraussetzungen:

- Nehmen Sie am [Windows-Insider-Programm](https://insider.windows.com/GettingStarted) teil und überprüfen Sie die Nutzungsbedingungen.
- Ein Computersystem (physisch oder virtuell) mit dem neuesten Builds von Windows Server aus dem Windows-Insider-Programm bzw. dem neuesten Build von Windows10 aus dem Windows-Insider-Programm.

> [!IMPORTANT]
> Sie müssen einen Build von Windows Server aus dem Windows Server Insider Preview-Programm oder ein Build von Windows 10 aus dem Windows Insider Preview-Programm, das Basisimage verwenden, die unten beschriebenen. Wenn Sie keine dieser Builds verwenden, kann der Container nicht mit diesen Basisimages gestartet werden.

## <a name="install-docker-enterprise-edition-ee"></a>Installieren der Docker Enterprise Edition (EE)

Für die Arbeit mit Windows-Containern ist Docker EE erforderlich. Docker EE besteht aus dem Docker-Modul und dem Docker-Client.

Verwenden Sie das PowerShell-Modul von OneGet, um Docker EE zu installieren. Der Anbieter aktiviert die Containerfunktion auf Ihrem Computer und installiert Docker EE. Dazu ist ein Neustart erforderlich. Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und führen Sie die folgenden Befehle aus.

> [!NOTE]
> Installieren von Docker EE mit Windows Server-Insider-Builds erfordert einen anderen OneGet-Anbieter als die für nicht-Insider-Builds vorgesehene. Wenn Docker EE und der OneGet-Anbieter DockerMsftProvider bereits installiert sind, entfernen Sie diese Komponenten, bevor Sie fortfahren.

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

```powershell
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>Installieren von Basiscontainerimages

Vor der Arbeit mit Windows-Containern muss ein Basisimage installiert werden. Wenn Sie am Windows-Insider-Programms teilnehmen, können Sie ebenfalls unsere neuesten Builds für die Basisimages testen. Unter den Insider-Basisimages stehen jetzt 4 auf Windows Server basierende Basisimages zur Verfügung. Weitere Informationen zu den verschiedenen Zwecken der einzelnen Basisimages finden Sie in der Tabelle unten:

| Basisbetriebssystemimages                       | Verwendungszweck                      |
|-------------------------------------|----------------------------|
| MCR.Microsoft.com/Windows/servercore         | Produktion und Entwicklung |
| MCR.Microsoft.com/Windows/nanoserver              | Produktion und Entwicklung |
| MCR.Microsoft.com/Windows/servercore/Insider | Nur Entwicklung           |
| MCR.Microsoft.com/Windows/nanoserver/Insider        | Nur Entwicklung           |

Zum Aufrufen des Basisimages für Nano Server Insider führen Sie folgenden Befehl aus:

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Zum Aufrufen des Basisimages für Windows Server Core Insider führen Sie folgenden Befehl aus:

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> Bitte lesen Sie die Windows-Container OS Image [EULA](../EULA.md ) und das Windows-Insider-Programm [Nutzungsbedingungen](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewserver).

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Erstellen und Ausführen einer beispielanwendung](./Nano-RS3-.NET-Core-and-PS.md)
