# Docker und Windows

**Dieser Inhalt ist vorläufig und kann geändert werden.**

Docker ist eine Plattform für die Bereitstellung und Verwaltung von Containern, die sich für Linux- und Windows-Container eignet. Docker dient zum Erstellen, Verwalten und Löschen von Containern und Containerimages. Docker ermöglicht das Speichern von Containerimages in einer öffentlichen Registrierung (Docker Hub) und privaten Registrierungen (Docker Trusted Registries). Docker bietet außerdem mit Docker Swarm Funktionen für das Erstellen von Containerhostclustern und mit Docker Compose eine Automatisierung der Bereitstellung. Weitere Informationen zu Docker und zum Docker-Toolset finden Sie auf [Docker.com](https://www.docker.com/).

> Das Feature „Windows-Container“ muss aktiviert werden, damit Docker zum Erstellen und Verwalten von Windows Server- und Hyper-V-Containern verwendet werden kann. Informationen zum Aktivieren dieses Features finden Sie in der [Bereitstellungsanleitung für Containerhosts](./docker_windows.md).

## Windows Server

### Installieren von Docker

Der Docker-Daemon und die Befehlszeilenschnittstelle (CLI) gehören nicht zum Funktionsumfang von Windows Server und Windows Server Core und werden nicht mit dem Feature „Windows-Container“ installiert. Docker muss separat installiert werden. In diesem Dokument wird die manuelle Installation des Docker-Daemons und Docker-Clients erläutert. Automatisierte Methoden für diese Aufgaben werden ebenfalls bereitgestellt.

Der Docker-Daemon und die Docker-Befehlszeilenschnittstelle wurden in der Sprache Go entwickelt. Derzeit lässt sich „docker.exe“ nicht als Windows-Dienst installieren. Der Windows-Dienst kann auf unterschiedliche Weise erstellt werden. In einem hier gezeigten Beispiel wird `nssm.exe` verwendet.

Laden Sie „docker.exe“ von `https://aka.ms/ContainerTools` herunter, und speichern Sie die Datei auf dem Containerhost im Verzeichnis „System32“.

```powershell
PS C:\> wget https://aka.ms/ContainerTools -OutFile $env:SystemRoot\system32\docker.exe
```

Erstellen Sie ein Verzeichnis mit dem Namen `c:\programdata\docker`. Erstellen Sie in diesem Verzeichnis eine Datei namens `runDockerDaemon.cmd`.

```powershell
PS C:\> New-Item -ItemType File -Path C:\ProgramData\Docker\runDockerDaemon.cmd -Force
```

Kopieren Sie den folgenden Text in die Datei `runDockerDaemon.cmd`. Diese Batchdatei startet den Docker-Daemon mit dem Befehl `docker daemon –D –b „Virtueller Switch“`. Hinweis: Der Name des virtuellen Switches in dieser Datei muss dem Namen des virtuellen Switches entsprechen, den Container für das Verbinden mit dem Netzwerk nutzen.

```powershell
@echo off
set certs=%ProgramData%\docker\certs.d

if exist %ProgramData%\docker (goto :run)
mkdir %ProgramData%\docker

:run
if exist %certs%\server-cert.pem (goto :secure)

docker daemon -D -b "Virtual Switch"
goto :eof

:secure
docker daemon -D -b "Virtual Switch" -H 0.0.0.0:2376 --tlsverify --tlscacert=%certs%\ca.pem --tlscert=%certs%\server-cert.pem --tlskey=%certs%\server-key.pem
```
Laden Sie „nssm.exe“ von [https://nssm.cc/release/nssm-2.24.zip](https://nssm.cc/release/nssm-2.24.zip) herunter.

```powershell
PS C:\> wget https://nssm.cc/release/nssm-2.24.zip -OutFile $env:ALLUSERSPROFILE\nssm.zip
```

Extrahieren Sie die Dateien, und kopieren Sie `nssm-2.24\win64\nssm.exe` in das Verzeichnis `c:\windows\system32`.

```powershell
PS C:\> Expand-Archive -Path $env:ALLUSERSPROFILE\nssm.zip $env:ALLUSERSPROFILE
PS C:\> Copy-Item $env:ALLUSERSPROFILE\nssm-2.24\win64\nssm.exe $env:SystemRoot\system32
```
Führen Sie `nssm install` aus, um den Docker-Dienst zu konfigurieren.

```powershell
PS C:\> start-process nssm install
```

Geben Sie die folgenden Daten in die entsprechenden Felder im NSSM-Installationsprogramm ein.

Registerkarte „Anwendung“:

- **Pfad:** C:\Windows\System32\cmd.exe

- **Startverzeichnis:** C:\Windows\System32

- **Argumente:** /s /c C:\ProgramData\docker\runDockerDaemon.cmd

- **Dienstname:** Docker

![](media/nssm1.png)

Registerkarte „Details“:

- **Anzeigename:** Docker

- **Beschreibung:** Der Docker-Daemon bietet Funktionen zur Verwaltung von Containern für Docker-Clients.


![](media/nssm2.png)

Registerkarte „E/A“:

- **Ausgabe (stdout):** C:\ProgramData\docker\daemon.log

- **Fehler (stderr):** C:\ProgramData\docker\daemon.log


![](media/nssm3.png)

Klicken Sie abschließend auf die Schaltfläche `Dienst installieren`.

Im Anschluss wird beim Start von Windows der Docker-Daemon (Dienst) ebenfalls gestartet.

### Entfernen von Docker

Wenn Sie diese Anleitung zum Erstellen eines Windows-Diensts anhand von „docker.exe“ befolgen, wird der Dienst mit dem folgenden Befehl entfernt.

```powershell
PS C:\> sc.exe delete Docker

[SC] DeleteService SUCESS
```

## Nano Server

### Installieren von Docker

Laden Sie „docker.exe“ von `https://aka.ms/ContainerTools` herunter, und speichern Sie die Datei auf dem Nano Server-Containerhost im Ordner `windows\system32`.

Führen Sie den folgenden Befehl aus, um den Docker-Daemon zu starten: Dieser Befehl muss jedes Mal ausgeführt werden, wenn der Containerhost gestartet wird. Dieser Befehl startet den Docker-Daemon, gibt einen virtuellen Switch für die Netzwerkverbindung des Containers an und legt den Daemon so fest, dass dieser Port 2375 auf eingehende Docker-Anforderungen überwacht. In dieser Konfiguration kann Docker von einem Remotecomputer aus verwaltet werden.

```powershell
PS C:\> start-process cmd "/k docker daemon -D -b <Switch Name> -H 0.0.0.0:2375”
```

### Entfernen von Docker

Um den Docker-Daemon und die CLI von Nano Server zu entfernen, löschen Sie `docker.exe` aus dem Verzeichnis „Windows\System32“.

```powershell
PS C:\> Remove-Item $env:SystemRoot\system32\docker.exe
```




