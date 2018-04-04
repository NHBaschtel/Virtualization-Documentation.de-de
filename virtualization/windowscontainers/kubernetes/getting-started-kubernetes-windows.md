---
title: Kubernetes unter Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: Hinzufügen eines Windows-Knotens zu einem Kubernetes-Cluster mit der Betaversion v1.9.
keywords: Kubernetes, 1.9, Windows, Erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 124895e93cbaee50c66b6b5a7cc2c71c144dad67
ms.sourcegitcommit: 6e3c3b2ff125f949c03a342c3709a6e57c5f736c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/17/2018
---
# <a name="kubernetes-on-windows"></a>Kubernetes unter Windows #

Mit der neuesten Version von Kubernetes 1.9 und Windows Server [Version 1709](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1709#networking)können Benutzer die neuesten Features des Windows-Netzwerks nutzen:

  - **freigegebene Pod-Depots**: Infrastruktur- und Worker-Pods teilen sich jetzt einen Netzwerkdepot (vergleichbar mit einem Linux-Namespace)
  - **Optimierung der Endpunkt**: Dank der Depotfreigabe müssen Container-Dienste (mindestens) nur halb so viele Endpunkte wie zuvor nachverfolgen
  - **Optimierung des Datenpfads**: Verbesserungen der virtuellen Filterplattform und des Host Networking Service ermöglichen einen Kernel-basierten Lastenausgleich


Diese Seite dient als Leitfaden für die ersten Schritte, wenn ein völlig neuer Windows-Knoten einem vorhandenen Linux-basierten Cluster hinzugefügt wird. Informationen, um von Grund auf neu zu beginnen, finden Sie [auf dieser Seite](./creating-a-linux-master.md) &mdash;. Hier finden Sie viele Ressourcen zur Bereitstellung eines Kubernetes-Clusters &mdash; und um einen Master von Grund auf neu einzurichten, wie von uns durchgeführt.

<a name="definitions"></a> Hier sind Definitionen für Begriffe, die in diesem Handbuch verwendet werden:

  - Das **externe Netzwerk** ist das Netzwerk, über das Ihre Knoten kommunizieren.
  - <a name="cluster-subnet-def"></a>Das **Cluster-Subnetz** ist ein routingfähiges virtuelles Netzwerk. Von hier werden Knoten kleinere Subnetze zugewiesen, die deren Pods verwenden können.
  - Das **Dienst-Subnetz** ist ein nicht routingfähiges, rein virtuelles Subnetz auf 11.0/16, das von Pods verwendet wird, um einheitlich auf Dienste zuzugreifen, ohne Rücksicht auf die Netzwerktopologie. Es wird von `kube-proxy` von/auf routingfähige Adressbereiche übersetzt, die auf diesen Knoten ausgeführt werden.

## <a name="what-you-will-accomplish"></a>Was Sie erreichen ##

Am Ende dieser Anleitung haben Sie:

> [!div class="checklist"]  
> * Einen [Linux Master](#preparing-the-linux-master)-Knoten konfiguriert.  
> * Einen [Windows-Worker-Knoten](#preparing-a-windows-node) hinzugefügt.  
> * Unsere [Netzwerktopologie](#network-topology) vorbereitet.  
> * Ein [Beispiel für einen Windows-Dienst](#running-a-sample-service) bereitgestellt.  
> * [Allgemeine Probleme und Fehler ](./common-problems.md) angesprochen.  

## <a name="preparing-the-linux-master"></a>Linux-Master vorbereiten ##

Unabhängig davon, ob Sie [die Anweisungen](./creating-a-linux-master.md) befolgt haben oder bereits über einen vorhandenen Cluster verfügen, es ist nur ein Element vom Linux-Master erforderlich: die Kubernetes-Zertifikatkonfiguration. Dies ist möglicherweise im `/etc/kubernetes/admin.conf`, `~/.kube/config` oder an anderer Stelle vorhanden, je nach Setup.

## <a name="preparing-a-windows-node"></a>Vorbereiten eines Windows-Knotens ##

> [!NOTE]  
> Alle Codeausschnitte in den Windows-Abschnitten müssen in PowerShell mit _erhöhten Rechten_ ausgeführt werden.

Kubernetes verwendet [Docker](https://www.docker.com/) als Container-Orchestrator, daher müssen wir es installieren. Folgen Sie den [offiziellen Dokumentanweisungen](../manage-docker/configure-docker-daemon.md#install-docker), den [Docker-Anweisungen](https://store.docker.com/editions/enterprise/docker-ee-server-windows) oder gehen Sie folgendermaßen vor:

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

Wenn Sie einen Proxyserver verwenden, müssen die folgenden PowerShell-Umgebungsvariablen definiert werden:
```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
```

Eine Sammlung der Skripts befindet sich in [diesem Microsoft-Repository](https://github.com/Microsoft/SDN). Diese hilft dabei, den Knoten zum Cluster hinzuzufügen. Sie können die ZIP-Datei direkt [hier](https://github.com/Microsoft/SDN/archive/master.zip) herunterladen. Sie brauchen lediglich den `Kubernetes/windows`-Ordner, dessen Inhalt auf `C:\k\` verschoben werden soll:

```powershell
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

Kopieren Sie die [vorher identifizierte](#preparing-the-linux-master) Zertifikatdatei in das neue `C:\k`-Verzeichnis.

## <a name="network-topology"></a>Netzwerktopologie ##

Es gibt mehrere Möglichkeiten, virtuelle [Clustersubnetze](#cluster-subnet-def) routingfähig zu machen. Sie haben folgende Möglichkeiten:

  - Konfigurieren Sie den [Hostgateway-Modus](./configuring-host-gateway-mode.md), indem Sie die Routen des nächsten Abschnitts zwischen Knoten festlegen und so eine Pod-zu‑Pod-Kommunikation ermöglichen.
  - Konfigurieren Sie einen TOR-Switch, um das Subnetz weiterzuleiten.
  - Verwenden Sie Drittanbieter-Plug-Ins wie beispielsweise [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html) (der Windows-Support für Flannel ist in der Betaversion).

### <a name="creating-the-pause-image"></a>Erstellen eines „pause”-Images ###

Nachdem Sie nun `docker` installiert haben, müssen Sie ein „pause”-Image vorbereiten, das von Kubernetes zum Vorbereiten der Infrastruktur-Pods verwendet wird.

```powershell
docker pull microsoft/windowsservercore:1709
docker tag microsoft/windowsservercore:1709 microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!NOTE]
> Wir bezeichnen es als `:latest`, weil der Beispieldienst, den Sie später bereitstellen werden, davon abhängt, obwohl dies möglicherweise _nicht_ das neueste verfügbare Windows Server Core-Image ist. Bei widersprüchlichen Container-Images können Probleme entstehen, denn wenn sie nicht das erwartete Tag besitzen, kann dies den `docker pull` eines inkompatiblen Container-Images verursachen, was zu [Bereitstellungsproblemen](./common-problems.md#when-deploying-docker-containers-keep-restarting) führt. 


### <a name="downloading-binaries"></a>Herunterladen von Binärdateien ###
Während `pull` durchgeführt wird, laden Sie die folgenden Binärdateien für den Client von Kubernetes herunter:

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

Sie können diese über die Links in der `CHANGELOG.md`-Datei der neuesten Version 1.9 herunterladen. Diese ist momentan [1.9.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1), und die Windows-Binärdateien befinden sich [hier](https://storage.googleapis.com/kubernetes-release/release/v1.9.1/kubernetes-node-windows-amd64.tar.gz). Verwenden Sie ein Tool wie [7-Zip](http://www.7-zip.org/) zum Extrahieren des Archivs, und speichern Sie die Binärdateien unter `C:\k\`.

Um sicherzustellen, das der Befehl `kubectl` außerhalb des Verzeichnisses `C:\k\` verfügbar ist, müssen Sie die Umgebungsvariable `PATH` ändern:

```powershell
$env:Path += ";C:\k"
```

Wenn Sie diese Änderung permanent machen möchten, ändern Sie die Variable im Computerziel:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

### <a name="joining-the-cluster"></a>Dem Cluster beitreten ###
Stellen Sie wie folgt sicher, dass die Clusterkonfiguration gültig ist:

```powershell
kubectl version
```

Falls ein Verbindungsfehler auftritt,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

überprüfen Sie, ob die Konfiguration ordnungsgemäß erkannt wurde:

```powershell
kubectl config view
```

Um den Speicherort zu ändern, an dem `kubectl` nach der Konfigurationsdatei sucht, können Sie den Parameter `--kubeconfig` übergeben oder die Umgebungsvariable `KUBECONFIG` ändern. Wenn die Konfiguration beispielsweise unter `C:\k\config` gespeichert ist:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Um diese Einstellung für den aktuellen Benutzerbereich permanent zu machen:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Der Knoten ist jetzt bereit, dem Cluster beizutreten. Führen Sie in zwei separaten PowerShell-Fenstern *mit erhöhten Rechten* diese Skripts (in dieser Reihenfolge) aus. Der `-ClusterCidr`-Parameter im ersten Skript ist das konfigurierte [Clustersubnetz](#cluster-subnet-def). Hier ist es `192.168.0.0/16`.

```powershell
./start-kubelet.ps1 -ClusterCidr 192.168.0.0/16
./start-kubeproxy.ps1
```

Der Windows-Knoten ist im Master-Linux unter `kubectl get nodes` innerhalb einer Minute sichtbar!


### <a name="validating-your-network-topology"></a>Überprüfen der Topologie des Netzwerks ###

Es gibt einige grundlegende Tests, mit denen eine ordnungsgemäße Konfiguration überprüft werden kann:

  - **Knoten-zu-Knoten-Konnektivität**: Pings zwischen Master- und Windows-Worker-Knoten sollten in beiden Richtungen erfolgreich ausgeführt werden.

  - **Pod-Subnetz-zu-Knoten-Konnektivität**: Pings zwischen der virtuellen Pod-Benutzeroberfläche und den Knoten. Suchen Sie die Gatewayadresse unter `route -n` und `ipconfig` auf Linux und Windows. Suchen Sie jeweils nach der `cbr0`-Schnittstelle.

Sollte einer der folgenden grundlegenden Tests nicht funktionieren, versuchen Sie es mit der [Seite für die Problembehandlung](./common-problems.md#common-networking-errors) für häufig auftretende Probleme.


## <a name="running-a-sample-service"></a>Ausführen eines Beispieldiensts ##

Sie werden einen sehr einfachen [PowerShell-basierten Webdienst](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) bereitstellen, um sicherzustellen, dass der Cluster erfolgreich hinzugefügt und unser Netzwerk ordnungsgemäß konfiguriert wurde.

Laden Sie auf dem Linux-Master den folgenden Dienst herunter und führen Sie ihn aus:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/WebServer.yaml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Dadurch entsteht eine Bereitstellung und ein Dienst, der die Pods auf unbestimmte Zeit überwacht, um ihren Status zu verfolgen. Drücken Sie einfach `Ctrl+C` zum Beenden des Befehls `watch`, wenn Sie mit der Beobachtung fertig sind.

Wenn alles erfolgreich verlaufen ist:

  - werden vier Container im Befehl `docker ps` auf der Windows-Seite angezeigt.
  - `curl` für die *Pod*-IDs auf Port 80 des Linux-Masters eine Antwort vom Web-Server erhalten. Dies bestätigt die ordnungsgemäße Kommunikation zwischen Pod und Knoten im Netzwerk.
  - `curl` die IP des *Knotens* auf Port 4444 erhält eine Antwort vom Web-Server. Dies veranschaulicht die ordnungsgemäße Host-zu-Container-Port-Zuordnung.
  - der Ping *zwischen Pods* (einschließlich zwischen Hosts, wenn Sie mehrere Windows-Knoten haben) über `docker exec`. Dies veranschaulicht die ordnungsgemäße Pod-zu-Pod-Kommunikation
  - `curl` die virtuelle *Dienst-IP* (unter `kubectl get services`) vom Linux-Master und den einzelnen Pods.
  - `curl` der *Dienstname* mit dem Kubernetes [Standard-DNS-Suffix](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services), der die DNS-Funktionen veranschaulicht.

> [!WARNING]  
> Windows-Knoten können nicht auf die Dienst-IP zugreifen. Dies ist eine [bekannte Plattformeinschränkung](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip), die bearbeitet wird.
