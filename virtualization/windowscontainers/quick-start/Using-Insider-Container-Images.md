# Verwenden von Insider-Containerimages

Diese Übung führt Sie durch die Bereitstellung und Verwendung der Windows-Containerfunktionen der neuesten Insider-Build von Windows Server aus dem Windows Insider Preview-Programm. In dieser Übung installieren Sie die Containerrolle und stellen eine Preview-Edition der Basisbetriebssystemimage bereit. Machen Sie sich vor diesem Schnellstart mit grundlegenden Containerkonzepten und der Terminologie vertraut. Diese Informationen finden Sie unter [Windows Containers Quick Start (Windows-Container – Schnellstart)](./index.md).

Dieser Schnellstart ist spezifisch für Windows Server-Container im Windows Server Insider Preview-Programm. Bitte machen Sie sich mit dem Programm vertraut, bevor Sie mit diesem Schnellstart fortfahren.

**Voraussetzungen:**

- Nehmen Sie am [Windows-Insider-Programm](https://insider.windows.com/GettingStarted) teil und überprüfen Sie die Nutzungsbedingungen. 
- Ein Computersystem (physisch oder virtuell) mit dem neuesten Builds von Windows Server aus dem Windows-Insider-Programm bzw. dem neuesten Build von Windows10 aus dem Windows-Insider-Programm.

>Sie müssen einen Build von Windows Server aus dem Windows Server Insider Preview-Programm oder einen Build von Windows10 aus dem Windows Insider Preview-Programm verwenden, um das unten beschriebene Basisimage verwenden zu können. Wenn Sie keine dieser Builds verwenden, kann der Container nicht mit diesen Basisimages gestartet werden.

## Installieren von Docker
Für die Arbeit mit Windows-Containern ist Docker erforderlich. Docker besteht aus dem Docker-Modul und dem Docker-Client. Sie müssen ebenfalls eine Version von Docker verwenden, die mehrstufige Builds unterstützt und für ein optimales Ergebnis das vom Container optimierte Nano Server-Image verwenden.

Verwenden Sie das PowerShell-Modul von OneGet, um Docker zu installieren. Der Anbieter aktiviert die Containerfunktion auf Ihrem Computer und installiert Docker. Dies macht einen Neustart erforderlich. Beachten Sie, dass es mehrere Kanäle mit den verschiedenen Version von Docker gibt, die in unterschiedlichen Fällen verwendet werden können. Für diese Übung verwenden wir die neueste Community Edition-Version von Docker vom Stable-Kanal. Es gibt ebenfalls einen Edge-Kanal, wenn Sie die neuesten Entwicklungen von Docker testen möchten. 

Öffnen Sie eine PowerShell-Sitzung mit erhöhten Rechten, und führen Sie die folgenden Befehle aus.

>Hinweis: Das Installieren von Docker in Insider-Builds erfordert einen anderen Anbieter als der regulär von Ihnen im Moment verwendeten. Bitte beachten Sie folgenden Unterschied.

Installieren Sie das PowerShell-Modul „OneGet“.
```powershell
Install-Module -Name DockerMsftProviderInsider -Repository PSGallery -Force
```
Verwenden Sie OneGet, um die neueste Version von Docker zu installieren.
```powershell
Install-Package -Name docker -ProviderName DockerMsftProviderInsider -RequiredVersion 17.06.0-ce
```
Wenn die Installation abgeschlossen ist, starten Sie den Computer neu.
```none
Restart-Computer -Force
```

## Installieren von Basiscontainerimages

Vor der Arbeit mit Windows-Containern muss ein Basisimage installiert werden. Wenn Sie am Windows-Insider-Programms teilnehmen, können Sie ebenfalls unsere neuesten Builds für die Basisimages testen. Unter den Insider-Basisimages stehen jetzt 4 auf Windows Server basierende Basisimages zur Verfügung. Weitere Informationen zu den verschiedenen Zwecken der einzelnen Basisimages finden Sie in der Tabelle unten:

| Basisbetriebssystemimages                       | Verwendungszweck                      |
|-------------------------------------|----------------------------|
| microsoft/windowsservercore         | Produktion und Entwicklung |
| microsoft/nanoserver                | Produktion und Entwicklung |
| microsoft/windowsservercore-insider | Nur Entwicklung           |
| microsoft/nanoserver-insider        | Nur Entwicklung           |

Zum Aufrufen des Basisimages für Nano Server Insider führen Sie folgenden Befehl aus:

```none
docker pull microsoft/nanoserver-insider
```

Zum Aufrufen des Basisimages für Windows Server Core Insider führen Sie folgenden Befehl aus:

```none
docker pull microsoft/windowsservercore-insider
```

Bitte lesen Sie den [Endbenutzer-Lizenzvertrag](../EULA.md ) zum Betriebssystemimage für Windows-Container durch. Die Nutzungsbedingungen für das Windows-Insider-Programm befinden sich unter [Nutzungsbedingungen](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewserver). 

## Nächste Schritte

[Erstellen und Ausführen einer Anwendung mit oder ohne .NET Core 2.0 oder PowerShell Core 6](./Nano-RS3-.NET-Core-and-PS.md)
