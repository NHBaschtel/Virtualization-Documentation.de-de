
# <a name="using-insider-container-images"></a>Verwenden von Insider-Containerimages

Diese Übung führt Sie durch die Bereitstellung und Verwendung der Windows-Containerfunktionen der neuesten Insider-Build von Windows Server aus dem Windows Insider Preview-Programm. In dieser Übung installieren Sie die Containerrolle und stellen eine Preview-Edition der Basisbetriebssystemimage bereit. Wenn Sie sich mit Containern vertraut machen möchten, finden Sie unter [Windows-Container](../about/index.md) entsprechende Informationen.

Dieser Schnellstart ist spezifisch für Windows Server-Container im Windows Server Insider Preview-Programm. Bitte machen Sie sich mit dem Programm vertraut, bevor Sie mit diesem Schnellstart fortfahren.

## <a name="prerequisites"></a>Voraussetzungen:

- Nehmen Sie am [Windows-Insider-Programm](https://insider.windows.com/GettingStarted) teil und überprüfen Sie die Nutzungsbedingungen.
- Ein Computersystem (physisch oder virtuell) mit dem neuesten Builds von Windows Server aus dem Windows-Insider-Programm bzw. dem neuesten Build von Windows10 aus dem Windows-Insider-Programm.

> [!IMPORTANT]
> Für Windows muss die Hostbetriebssystem-Version der Container-Betriebssystemversion entsprechen. Wenn Sie einen Container auf der Grundlage eines neueren Windows-Builds ausführen möchten, stellen Sie sicher, dass Sie über einen entsprechenden Host Build verfügen. Andernfalls können Sie die Hyper-V-Isolierung verwenden, um ältere Container auf neuen Host Builds auszuführen. Weitere Informationen zur Kompatibilität der Windows-Container Version finden Sie in unseren Container-Dokumenten.

## <a name="install-docker-enterprise-edition-ee"></a>Installieren der Docker Enterprise Edition (EE)

Für die Arbeit mit Windows-Containern ist Docker EE erforderlich. Docker EE besteht aus dem Docker-Modul und dem Docker-Client.

Verwenden Sie das PowerShell-Modul von OneGet, um Docker EE zu installieren. Der Anbieter aktiviert die Containerfunktion auf Ihrem Computer und installiert Docker EE. Dazu ist ein Neustart erforderlich. Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und führen Sie die folgenden Befehle aus.

> [!NOTE]
> Die Installation von Docker EE mit Windows Server Insider-Builds erfordert einen anderen OneGet-Anbieter als den, der für nicht-Insider-Builds verwendet wird. Wenn Docker EE und der OneGet-Anbieter DockerMsftProvider bereits installiert sind, entfernen Sie diese Komponenten, bevor Sie fortfahren.

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

Vor der Arbeit mit Windows-Containern muss ein Basisimage installiert werden. Wenn Sie am Windows-Insider-Programms teilnehmen, können Sie ebenfalls unsere neuesten Builds für die Basisimages testen. Mit den Insider-Basisbildern sind jetzt 6 verfügbare Basisbilder auf Grundlage von Windows Server verfügbar. Weitere Informationen zu den verschiedenen Zwecken der einzelnen Basisimages finden Sie in der Tabelle unten:

| Basisbetriebssystemimages                       | Verwendungszweck                      |
|-------------------------------------|----------------------------|
| mcr.microsoft.com/windows/servercore         | Produktion und Entwicklung |
| mcr.microsoft.com/windows/nanoserver              | Produktion und Entwicklung |
| mcr.microsoft.com/windows/              | Produktion und Entwicklung |
| mcr.microsoft.com/windows/servercore/insider | Nur Entwicklung           |
| mcr.microsoft.com/windows/nanoserver/insider        | Nur Entwicklung           |
| mcr.microsoft.com/windows/insider        | Nur Entwicklung           |

Informationen zum Abrufen des Server Core-Insider-Basis Bilds finden Sie unter den am [Server Core Insider docker Hub Repo](https://hub.docker.com/_/microsoft-windows-servercore-insider) verfügbaren Tags (n), um das folgende Format zu verwenden:

```console
docker pull mcr.microsoft.com/windows/servercore/insider:10.0.{build}.{revision}
```

Wenn Sie das Image des Nano-Server-Insider-Images ziehen möchten, lesen Sie die auf dem [Nano Server-Insider-andocker-Hub Repo](https://store.docker.com/_/microsoft-windows-nanoserver-insider) verfügbaren Tags (n), um das folgende Format zu verwenden:

```console
docker pull mcr.microsoft.com/windows/nanoserver/insider:10.0.{build}.{revision}
```

Wenn Sie das Windows-Insider-Basis Bild abrufen möchten, lesen Sie die am [Windows Insider-docker-Hub-Repo-Hub-Depot](https://store.docker.com/_/microsoft-windows-insider) verwendeten Tags (s), um das folgende Format zu verwenden:

```console
docker pull mcr.microsoft.com/windows/insider:10.0.{build}.{revision}
```

> [!IMPORTANT]
> Bitte lesen Sie die [Nutzungsbedingungen für](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)Windows-Container-Betriebs [System Bilder und](../EULA.md ) das Windows-Insider-Programm.

## <a name="next-steps"></a>Nächste Schritte

> [!div class="nextstepaction"]
> [Erstellen und Ausführen einer Beispielanwendung](./Nano-RS3-.NET-Core-and-PS.md)
