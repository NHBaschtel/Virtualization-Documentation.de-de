---
title: Kubernetes unter Windows
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: "Hinzufügen eines Windows-Knotens zu einem Kubernetes-Cluster mit der Betaversion v1.9."
keywords: Kubernetes, 1.9, Windows, Erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: d88ab46dc0046256ebed9c6696a99104a7197fad
ms.sourcegitcommit: ad5f6344230c7c4977adf3769fb7b01a5eca7bb9
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/05/2017
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


## <a name="what-we-will-accomplish"></a>Was wir erreichen ##
Am Ende dieser Anleitung haben wir:

> [!div class="checklist"]  
> * Unsere [Netzwerktopologie](#network-topology) vorbereitet.  
> * Einen [Linux Master](#preparing-the-linux-master)-Knoten konfiguriert.  
> * Einen [Windows-Worker-Knoten](#preparing-a-windows-node) hinzugefügt.  
> * Ein [Beispiel für einen Windows-Dienst](#running-a-sample-service) bereitgestellt.  
> * [Allgemeine Probleme und Fehler ](./common-problems.md) angesprochen.  


## <a name="network-topology"></a>Netzwerktopologie ##
Es gibt mehrere Möglichkeiten, virtuelle [Clustersubnetze](#cluster-subnet-def) routingfähig zu machen. Sie haben folgende Möglichkeiten:

  - Konfigurieren Sie den [Hostgateway-Modus](./configuring-host-gateway-mode.md), indem Sie die Routen des nächsten Abschnitts zwischen Knoten festlegen und so eine Pod-zu‑Pod-Kommunikation ermöglichen.
  - Konfigurieren Sie einen TOR-Switch, um das Subnetz weiterzuleiten.
  - Verwenden Sie Drittanbieter-Plug-Ins wie beispielsweise [Flannel](https://coreos.com/flannel/docs/latest/kubernetes.html) (der Windows-Support für Flannel ist in der Betaversion).


## <a name="preparing-the-linux-master"></a>Vorbereiten des Linux-Masters ##
Unabhängig davon, ob Sie [unsere Anweisungen](./creating-a-linux-master.md) befolgt haben oder bereits über einen vorhandenen Cluster verfügen, es ist nur ein Element vom Linux-Master erforderlich: die Kubernetes-Zertifikatkonfiguration. Dies ist möglicherweise im `/etc/kubernetes/admin.conf`, `~/.kube/config` oder an anderer Stelle vorhanden, je nach Setup.


## <a name="preparing-a-windows-node"></a>Vorbereiten eines Windows-Knotens ##
> [!Note]  
> Alle Codeausschnitte in den Windows-Abschnitten müssen in PowerShell mit erhöhten Rechten ausgeführt werden.

Kubernetes verwendet [Docker](https://www.docker.com/) als Container-Orchestrator, daher müssen wir es installieren. Folgen Sie den [offiziellen MSDN Anweisungen](virtualization/windowscontainers/manage-docker/configure-docker-daemon.md#install-docker), den [Docker-Anweisungen](https://store.docker.com/editions/enterprise/docker-ee-server-windows) oder gehen Sie folgendermaßen vor:

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name Docker -ProviderName DockerMsftProvider
Restart-Computer -Force
```

Eine Sammlung der Skripts befindet sich in [diesem Microsoft-Repository](https://github.com/Microsoft/SDN), das dabei hilft, diesen Knoten zum Cluster hinzuzufügen. Sie können die ZIP-Datei direkt [hier](https://github.com/Microsoft/SDN/archive/master.zip) herunterladen. Wir brauchen lediglich den `Kubernetes/windows`-Ordner, dessen Inhalt auf `C:\k\` verschoben werden sollte:

```powershell
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mkdir C:/k/
mv master/SDN-master/Kubernetes/windows/* C:/k/
rm -recurse -force master,master.zip
```

Kopieren Sie die [vorher identifizierte](#preparing-the-linux-master) Zertifikatdatei in das neue `C:\k`-Verzeichnis.


### <a name="creating-the-pause-image"></a>Erstellen Sie das Bild "Anhalten" ###
Nachdem Sie nun `docker` installiert haben, müssen wir das Bild "Anhalten" vorbereiten, das von Kubernetes zum Vorbereiten der Infrastruktur-Pods verwendet wird.

```powershell
docker pull microsoft/windowsservercore:1709
docker tag $(docker images -q) microsoft/windowsservercore:latest
cd C:/k/
docker build -t kubeletwin/pause .
```

> [!Note]  
> Wir markieren dies als `:latest`, da dies vom Beispieldienst erwartet wird, den wir später bereitstellen.


### <a name="downloading-binaries"></a>Herunterladen von Binärdateien ###
Während `pull` durchgeführt wird, laden Sie die folgenden Binärdateien für den Client von Kubernetes herunter:

  - `kubectl.exe`
  - `kubelet.exe`
  - `kube-proxy.exe`

Sie können diese über die Links in der `CHANGELOG.md`-Datei der neuesten Version 1.9 herunterladen. Hier ist dies [1.9.0-beta.1](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.0-beta.1). Die Windows-Binärdateien befinden sich [hier](https://dl.k8s.io/v1.9.0-beta.1/kubernetes-node-windows-amd64.tar.gz). Verwenden Sie ein Tool wie [7-Zip](http://www.7-zip.org/) zum Extrahieren des Archivs, und speichern Sie die Binärdateien unter `C:\k\`.

> [!Warning]  
> Zum Zeitpunkt dieses Artikels erfordert `kube-proxy.exe` eine ausstehende Kubernetes [Pull-Anforderung](https://github.com/kubernetes/kubernetes/pull/56529), damit alles ordnungsgemäß funktioniert. Sie müssen möglicherweise die [die Binärdateien manuell erstellen](./compiling-kubernetes-binaries.md), um das Problem zu umgehen.


### <a name="joining-the-cluster"></a>Hinzufügen des Clusters ###
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

Sollte einer der folgenden grundlegenden Tests nicht funktionieren, versuchen Sie es mit der [Seite für die Problembehandlung](./common-problems.md#network-connectivity) für häufig auftretende Probleme.


## <a name="running-a-sample-service"></a>Ausführen eines Beispieldiensts ##
Wir werden einen sehr einfachen [PowerShell-basierten Webdienst](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) bereitstellen, um sicherzustellen, dass der Cluster erfolgreich hinzugefügt und unser Netzwerk ordnungsgemäß konfiguriert sind.


Laden Sie auf dem Linux-Master den folgenden Dienst herunter und führen Sie ihn aus:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/WebServer.yaml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Dadurch wird eine Bereitstellung und ein Dienst erstellt, der die Pods auf unbestimmte Zeit überwacht, um ihren Status zu verfolgen. Drücken Sie einfach `Ctrl+C` zum Beenden des Befehls `watch`, wenn Sie mit der Beobachtung fertig sind.


Wenn alles erfolgreich verläuft, können Sie folgende Möglichkeiten überprüfen:

  - es werden 4 Container im Befehl `docker ps` auf der Windows-Seite angezeigt
  - `curl` die IP-Adressen der *Pods* auf Port 80 des Linux-Masters erhält eine Antwort vom Web-Server. Dies veranschaulicht die ordnungsgemäße Kommunikation zwischen Pod und Knoten im Netzwerk.
  - `curl` die IP des *Knotens* auf Port 4444 erhält eine Antwort vom Web-Server. Dies veranschaulicht die ordnungsgemäße Host-zu-Container-Port-Zuordnung.
  - der Ping *zwischen Pods* (einschließlich zwischen Hosts, wenn Sie mehrere Windows-Knoten haben) über `docker exec`. Dies veranschaulicht die ordnungsgemäße Pod-zu-Pod-Kommunikation
  - `curl` die virtuelle *Dienst-IP* (unter `kubectl get services`) vom Linux-Master und den einzelnen Pods.
  - `curl` der *Dienstname* mit dem Kubernetes [Standard-DNS-Suffix](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services), der die DNS-Funktionen veranschaulicht.

> [!Warning]  
> Windows-Knoten können nicht auf die Dienst-IP zugreifen. Dies ist eine [bekannte Einschränkung](./common-problems.md#common-windows-errors).
