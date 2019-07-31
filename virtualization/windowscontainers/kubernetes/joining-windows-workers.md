---
title: Beitreten zu Windows-Knoten
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Verknüpfen eines Windows-Knotens mit einem Kubernetes-Cluster mit v 1.14
keywords: kubernetes, 1,14, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c9dbfec968d52d9fbc528892f0e3749270e3ff70
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/31/2019
ms.locfileid: "9882983"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Beitreten zu Windows Server-Knoten zu einem Cluster #
Nachdem Sie [einen Kubernetes-Masterknoten eingerichtet](./creating-a-linux-master.md) und [die gewünschte Netzwerklösung ausgewählt](./network-topologies.md)haben, können Sie an Windows Server-Knoten teilnehmen, um einen Cluster zu bilden. Dies erfordert eine gewisse [Vorbereitung auf die Windows-Knoten](#preparing-a-windows-node) , bevor Sie beitreten.

## <a name="preparing-a-windows-node"></a>Vorbereiten eines Windows-Knotens ##
> [!NOTE]  
> Alle Codeausschnitte in den Windows-Abschnitten müssen in PowerShell mit _erhöhten Rechten_ ausgeführt werden.

### <a name="install-docker-requires-reboot"></a>Docker installieren (Neustart erforderlich) ###
Kubernetes verwendet [docker](https://www.docker.com/) als Containermodul, sodass wir es installieren müssen. Folgen Sie den [offiziellen Dokumentanweisungen](../manage-docker/configure-docker-daemon.md#install-docker), den [Docker-Anweisungen](https://store.docker.com/editions/enterprise/docker-ee-server-windows) oder gehen Sie folgendermaßen vor:

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

Wenn nach dem Neustart die folgende Fehlermeldung angezeigt wird:

![Text](media/docker-svc-error.png)

Starten Sie den Andock Dienst dann manuell:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>Erstellen des Bilds "Pause" (Infrastruktur) ###
> [!Important]
> Es ist wichtig, in Konflikt stehende Container Bilder vorsichtig zu sein. Wenn das erwartete Tag nicht angezeigt wird, `docker pull` kann dies zu einem inkompatiblen Container Bild führen, das zu [Bereitstellungsproblemen](./common-problems.md#when-deploying-docker-containers-keep-restarting) wie dem unbegrenzten `ContainerCreating` Status führt.

Nachdem Sie nun `docker` installiert haben, müssen Sie ein „pause”-Image vorbereiten, das von Kubernetes zum Vorbereiten der Infrastruktur-Pods verwendet wird. Dazu gibt es drei Schritte: 
  1. [Ziehen des Bilds](#pull-the-image)
  2. [Tagging](#tag-the-image) als Microsoft/Server: aktuell
  3. und [Ausführen](#run-the-container)


#### <a name="pull-the-image"></a>Ziehen des Bilds ####     
 Ziehen Sie das Bild für Ihre spezifische Windows-Version. Wenn Sie beispielsweise Windows Server 2019 ausführen, gehen Sie wie folgt vor:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>Markieren des Bilds ####
Die Dockerfiles, die Sie später in diesem Leitfaden verwenden werden, `:latest` suchen nach dem Image-Tag. Markieren Sie das Bild, das Sie soeben abgerufen haben, wie folgt:

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>Ausführen des Containers ####
Überprüfen Sie, ob der Container tatsächlich auf Ihrem Computer ausgeführt wird:

```powershell
docker run microsoft/nanoserver:latest
```

Sie sollten etwa folgendes sehen:

![Text](./media/docker-run-sample.png)

> [!tip]
> Wenn Sie den Container nicht ausführen können, lesen Sie: [passende Container-Host-Version mit Container-Bild](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Vorbereiten von Kubernetes für Windows-Verzeichnis ####
Erstellen Sie ein Verzeichnis "Kubernetes für Windows", um Kubernetes-Binärdateien sowie alle Bereitstellungsskripts und Konfigurationsdateien zu speichern.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Kopieren des Kubernetes-Zertifikats #### 
Kopieren Sie die Kubernetes-Zertifikats`$HOME/.kube/config`Datei () [vom Master](./creating-a-linux-master.md#collect-cluster-information) in `C:\k` dieses neue Verzeichnis.

> [!tip]
> Sie können Tools wie [xcopy](https://docs.microsoft.com/windows-server/administration/windows-commands/xcopy) oder [WinSCP](https://winscp.net/eng/download.php) verwenden, um die Konfigurationsdatei zwischen Knoten zu übertragen.

#### <a name="download-kubernetes-binaries"></a>Herunterladen von Kubernetes-Binärdateien ####
Um Kubernetes ausführen zu können, müssen Sie zuerst die `kubectl`, `kubelet`und `kube-proxy` Binärdateien herunterladen. Sie können diese über die Links in der `CHANGELOG.md` Datei der [neuesten Versionen](https://github.com/kubernetes/kubernetes/releases/)herunterladen.
 - Hier sind beispielsweise die [Binärdateien des v 1.14](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.14.md#node-binaries)-Knotens.
 - Verwenden Sie ein Tool wie " [Expand-Archive](https://docs.microsoft.com/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) ", um das Archiv zu extrahieren und `C:\k\`die Binärdateien in zu platzieren.

#### <a name="optional-setup-kubectl-on-windows"></a>Optional Einrichten von kubectl unter Windows ####
Wenn Sie den Cluster über Windows steuern möchten, können Sie dies über den `kubectl` Befehl tun. Ändern Sie zunächst die `kubectl` `PATH` Umgebungsvariable, damit `C:\k\` Sie außerhalb des Verzeichnisses verfügbar ist:

```powershell
$env:Path += ";C:\k"
```

Wenn Sie diese Änderung permanent machen möchten, ändern Sie die Variable im Computerziel:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

Als nächstes werden wir überprüfen, ob das [Cluster Zertifikat](#copy-kubernetes-certificate) gültig ist. Um den Speicherort `kubectl` für die Konfigurationsdatei zu bestimmen, können Sie den `--kubeconfig` Parameter übergeben oder die `KUBECONFIG` Umgebungsvariable ändern. Wenn die Konfiguration beispielsweise unter `C:\k\config` gespeichert ist:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Um diese Einstellung für den aktuellen Benutzerbereich permanent zu machen:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Um zu überprüfen, ob die Konfiguration richtig erkannt wurde, können Sie Folgendes verwenden:

```powershell
kubectl config view
```

Falls ein Verbindungsfehler auftritt,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Überprüfen Sie den kubeconfig-Speicherort, oder versuchen Sie, ihn erneut zu kopieren.

Wenn keine Fehler angezeigt werden, ist der Knoten nun bereit, dem Cluster beizutreten.

## <a name="joining-the-windows-node"></a>Beitreten zum Windows-Knoten ##
Je nach [ausgewählter Netzwerklösung](./network-topologies.md)haben Sie folgende Möglichkeiten:
1. [Teilnehmen an Windows Server-Knoten zu einem Flanell-Cluster (vxlan oder Host-GW)](#joining-a-flannel-cluster)
2. [Teilnehmen an Windows Server-Knoten zu einem Cluster mit einem Tor-Schalter](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Beitreten zu einem Flanell-Cluster ###
In [diesem Microsoft-Repository](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) finden Sie eine Sammlung von Flanell-Bereitstellungsskripts, die Ihnen bei der Teilnahme an diesem Knoten beim Cluster helfen.

Laden Sie `C:\k`das [Flanell-Start. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) -Skript herunter, dessen Inhalt extrahiert werden soll:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

Vorausgesetzt, Sie haben [Ihren Windows-Knoten vorbereitet](#preparing-a-windows-node), und Ihr `c:\k` Verzeichnis sieht wie folgt aus: Sie sind bereit, dem Knoten beizutreten.

![Text](./media/flannel-directory.png)

#### <a name="join-node"></a>Knoten beitreten #### 
Um den Prozess der Teilnahme an einem Windows-Knoten zu vereinfachen, müssen Sie nur ein einzelnes Windows-Skript `kubelet`ausführen `kube-proxy`, `flanneld`um das Programm zu starten, und dem Knoten beizutreten.

> [!Note]
> [Start. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) verweist auf [install. ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1), mit dem zusätzliche Dateien wie die `flanneld` ausführbare Datei und die [Dockerfile für Infrastructure Pod](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) heruntergeladen *und für Sie installiert*werden. Bei Overlay-Netzwerkmodus wird die [Firewall](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) für den lokalen UDP-Port 4789 geöffnet. Möglicherweise werden mehrere PowerShell-Fenster geöffnet/geschlossen sowie einige Sekunden des Netzwerkausfalls, während das neue externe Vswitch für das Pod-Netzwerk zum ersten Mal erstellt wird.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
Die dem Windows-Knoten zugewiesene IP-Adresse. Sie können diese `ipconfig` Informationen verwenden.

|  |  | 
|---------|---------|
|Parameter     | `-ManagementIP`        |
|Standardwert    | n.A. **required**        |

# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
Der Netzwerkmodus `l2bridge` (Flanell-Host-GW) `overlay` oder (Flanell-vxlan), der als [Netzwerklösung](./network-topologies.md)ausgewählt wurde.

> [!Important] 
> `overlay` der Netzwerkmodus (Flanell vxlan) erfordert Kubernetes v 1.14-Binärdateien (oder höher) und [KB4489899](https://support.microsoft.com/help/4489899).

|  |  | 
|---------|---------|
|Parameter     | `-NetworkMode`        |
|Standardwert    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
Der [Bereich des Cluster](./getting-started-kubernetes-windows.md#cluster-subnet-def)-Subnetzes.

|  |  | 
|---------|---------|
|Parameter     | `-ClusterCIDR`        |
|Standardwert    | `10.244.0.0/16`        |


# [<a name="servicecidr"></a>ServiceCIDR](#tab/ServiceCIDR)
Der [Dienst-Subnetz-Bereich](./getting-started-kubernetes-windows.md#service-subnet-def).

|  |  | 
|---------|---------|
|Parameter     | `-ServiceCIDR`        |
|Standardwert    | `10.96.0.0/12`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
Die [IP-Adresse des Kubernetes-DNS-Diensts](./getting-started-kubernetes-windows.md#plan-ip-addressing-for-your-cluster).

|  |  | 
|---------|---------|
|Parameter     | `-KubeDnsServiceIP`        |
|Standardwert    | `10.96.0.10`        |


# [<a name="interfacename"></a>InterfaceName](#tab/InterfaceName)
Der Name der Netzwerkschnittstelle des Windows-Hosts. Sie können diese `ipconfig` Informationen verwenden.

|  |  | 
|---------|---------|
|Parameter     | `-InterfaceName`        |
|Standardwert    | `Ethernet`        |


# [<a name="logdir"></a>LOGDIR](#tab/LogDir)
Das Verzeichnis, in dem kubelet-und Kuben-Proxy-Protokolle in ihre jeweiligen Ausgabedateien umgeleitet werden.

|  |  | 
|---------|---------|
|Parameter     | `-LogDir`        |
|Standardwert    | `C:\k`        |


---

> [!tip]
> Sie haben bereits das Cluster-Subnetz, das Service-Subnetz und die Kuben-DNS-IP vom Linux-Master [zuvor](./creating-a-linux-master.md#collect-cluster-information) notiert

Nach dem Ausführen dieser Aktion sollten Sie in der Lage sein:
  * Anzeigen von verbundenen Windows-Knoten mithilfe von `kubectl get nodes`
  * Siehe 3 PowerShell-Fenster geöffnet, eins `kubelet`für, eins `flanneld`für und ein anderes für `kube-proxy`
  * Sehen Sie sich die Host- `flanneld`Agent `kubelet`-Prozesse `kube-proxy` für und die Ausführung auf dem Knoten an.

Wenn Sie erfolgreich sind, fahren Sie mit den [nächsten Schritten](#next-steps)fort.

## <a name="joining-a-tor-cluster"></a>Beitreten zu einem Tor-Cluster ##
> [!NOTE]
> Sie können diesen Abschnitt überspringen, wenn Sie [zuvor](./network-topologies.md#flannel-in-host-gateway-mode)Flanell als Netzwerklösung gewählt haben.

Zu diesem Zweck müssen Sie die Anweisungen zum [Einrichten von Windows Server-Containern auf Kubernetes für die Upstream-Routing Topologie](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology)befolgen. Dazu müssen Sie sicherstellen, dass Sie den Upstream-Router so konfigurieren, dass das einem Knoten zugewiesene Pod CIDR-Präfix der jeweiligen Knoten-IP-Adresse zugeordnet ist.

Vorausgesetzt, dass der neue Knoten als "bereit" `kubectl get nodes`aufgeführt wird, kubelet + Kuben-Proxy ausgeführt wird und Sie den Upstream-Router konfiguriert haben, sind Sie bereit für die nächsten Schritte.

## <a name="next-steps"></a>Nächste Schritte ##
In diesem Abschnitt wird beschrieben, wie Sie Windows-Mitarbeitern zu unserem Kubernetes-Cluster beitreten können. Jetzt sind Sie bereit für Schritt 5:

> [!div class="nextstepaction"]
> [Teilnahme an Linux-Mitarbeitern](./joining-linux-workers.md)

Wenn Sie aber keine Linux-Mitarbeiter haben, können Sie mit Schritt 6 fortfahren:

> [!div class="nextstepaction"]
> [Bereitstellen von Kubernetes-Ressourcen](./deploying-resources.md)
