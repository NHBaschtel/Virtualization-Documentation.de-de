---
title: Verknüpfen von Windows-Knoten
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Wenn einen Windows-Knoten zu einem Kubernetes-Cluster mit v1.13.
keywords: Kubernetes, 1.13, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: ed0f13bd429e88f05469f91c3fc691bf0188b0a2
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578241"
---
# <a name="joining-windows-server-nodes-to-a-cluster"></a>Hinzufügen von Windows Server-Knoten zu einem Cluster #
Wenn Sie die [Einrichtung eines Kubernetes-master-Knotens](./creating-a-linux-master.md) und [Ihrer Projektmappe gewünschte Netzwerk ausgewählt](./network-topologies.md)haben, sind Sie bereit, Windows Server-Knoten zu einem Cluster beizutreten. Dies erfordert einige [Vorbereitung auf den Windows-Knoten](#preparing-a-windows-node) , bevor Sie einbinden.

## <a name="preparing-a-windows-node"></a>Vorbereiten eines Windows-Knotens ##
> [!NOTE]  
> Alle Codeausschnitte in den Windows-Abschnitten müssen in PowerShell mit _erhöhten Rechten_ ausgeführt werden.

### <a name="install-docker-requires-reboot"></a>Installieren von Docker (erfordert Neustart) ###
Kubernetes verwendet [Docker](https://www.docker.com/) als die Container-Engine, daher wir es installieren müssen. Folgen Sie den [offiziellen Dokumentanweisungen](../manage-docker/configure-docker-daemon.md#install-docker), den [Docker-Anweisungen](https://store.docker.com/editions/enterprise/docker-ee-server-windows) oder gehen Sie folgendermaßen vor:

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

Wenn nach dem Neustart die folgende Fehlermeldung angezeigt:

![Text](media/docker-svc-error.png)

Starten Sie dann den Docker-Dienst manuell:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>Erstellen Sie das Bild "Anhalten" (Infrastruktur) ###
> [!Important]
> Es ist wichtig, darauf achten, dass widersprüchlichen Container-Images. ohne das erwartete Tag kann dazu führen, dass eine `docker pull` eines Bilds inkompatiblen Container [Bereitstellungsprobleme](./common-problems.md#when-deploying-docker-containers-keep-restarting) verursachen, z. B. auf unbestimmte `ContainerCreating` Status.

Nachdem Sie nun `docker` installiert haben, müssen Sie ein „pause”-Image vorbereiten, das von Kubernetes zum Vorbereiten der Infrastruktur-Pods verwendet wird. Es gibt drei Schritte: 
  1. [Ziehen das Bild](#pull-the-image)
  2. [Markieren Sie es](#tag-the-image) als Microsoft / Nanoserver:latest
  3. und [ausgeführt wird](#run-the-container)


#### <a name="pull-the-image"></a>Ziehen Sie das Bild ####     
 Ziehen Sie das Bild für Ihre spezifischen Windows-Version. Beispielsweise, wenn Sie Windows Server 2019 ausgeführt werden:

 ```powershell
docker pull mcr.microsoft.com/windows/nanoserver:1809
 ```

#### <a name="tag-the-image"></a>Markieren Sie das Bild ####
Suchen Sie die dockerfile-Dateien Sie später in diesem Handbuch verwenden für die `:latest` image-Tag. Markieren Sie das Nanoserver-Bild, das Sie genau wie folgt gezogen:

```powershell
docker tag mcr.microsoft.com/windows/nanoserver:1809 microsoft/nanoserver:latest
```

#### <a name="run-the-container"></a>Führen Sie den container ####
Überprüfen Sie, dass der Container tatsächlich auf Ihrem Computer ausgeführt wird:

```powershell
docker run microsoft/nanoserver:latest
```

Sie sollte etwa wie folgt angezeigt:

![Text](./media/docker-run-sample.png)

> [!tip]
> Wenn Sie den Container Bitte finden Sie unter ausführen können: [übereinstimmende Version der Container-Host mit Container-Image](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility#matching-container-host-version-with-container-image-versions)


#### <a name="prepare-kubernetes-for-windows-directory"></a>Vorbereiten von Kubernetes für Windows-Verzeichnis ####
Erstellen Sie ein "Kubernetes für Windows"-Verzeichnis zum Speichern von Kubernetes-Binärdateien, sowie alle Bereitstellungsskripts und Config-Dateien.

```powershell
mkdir c:\k
```

#### <a name="copy-kubernetes-certificate"></a>Kopieren Sie Kubernetes-Zertifikat #### 
Kopieren Sie die Zertifikatdatei Kubernetes (`$HOME/.kube/config`) [Master](./creating-a-linux-master.md#collect-cluster-information) neuen `C:\k` Verzeichnis.

> [!tip]
> Sie können Tools wie [Xcopy](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/xcopy) oder [WinSCP](https://winscp.net/eng/download.php) verwenden, die Konfigurationsdatei zwischen Knoten übertragen.

#### <a name="download-kubernetes-binaries"></a>Herunterladen von Kubernetes-Binärdateien ####
Um Kubernetes ausgeführt werden können, müssen Sie zuerst herunterladen der `kubectl`, `kubelet`, und `kube-proxy` Binärdateien. Sie können diese über die Links im Herunterladen der `CHANGELOG.md` Datei die [neuesten Versionen](https://github.com/kubernetes/kubernetes/releases/).
 - Hier sind z. B. die [v1.13 Knoten-Binärdateien](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.13.md#node-binaries).
 - Verwenden Sie ein Tool wie [Erweitern-Archiv](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.archive/expand-archive?view=powershell-6) zum Extrahieren des Archivs aus und platzieren die Binärdateien in `C:\k\`.

#### <a name="optional-setup-kubectl-on-windows"></a>(Optional) Setup-Kubectl unter Windows ####
Sollten Sie möchten Cluster von Windows zu steuern, Sie können dazu die `kubectl` Befehl. Erste, um sicherzustellen, `kubectl` außerhalb des verfügbaren der `C:\k\` Verzeichnis und Ändern der `PATH` Umgebungsvariable:

```powershell
$env:Path += ";C:\k"
```

Wenn Sie diese Änderung permanent machen möchten, ändern Sie die Variable im Computerziel:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

Als Nächstes werden wir stellen Sie sicher, dass der [Cluster-Zertifikat](#copy-kubernetes-certificate) gültig ist. Um den Speicherort festzulegen, in denen `kubectl` nach der Konfigurationsdatei sucht, übergeben Sie die `--kubeconfig` Parameter oder Ändern der `KUBECONFIG` Umgebungsvariable. Wenn die Konfiguration beispielsweise unter `C:\k\config` gespeichert ist:

```powershell
$env:KUBECONFIG="C:\k\config"
```

Um diese Einstellung für den aktuellen Benutzerbereich permanent zu machen:

```powershell
[Environment]::SetEnvironmentVariable("KUBECONFIG", "C:\k\config", [EnvironmentVariableTarget]::User)
```

Schließlich, um zu überprüfen, ob die Konfiguration ordnungsgemäß erkannt wurde, können Sie:

```powershell
kubectl config view
```

Falls ein Verbindungsfehler auftritt,

```
Unable to connect to the server: dial tcp [::1]:8080: connectex: No connection could be made because the target machine actively refused it.
```

Sie sollten überprüfen den Speicherort Kubeconfig oder versuchen Sie es erneut zu kopieren.

Der Knoten ist jetzt bereit, dem Cluster beizutreten, wenn keine Fehler angezeigt werden.

## <a name="joining-the-windows-node"></a>Wenn die Windows-Knoten ##
Je nach [Netzwerk-Lösung, die Sie ausgewählt haben](./network-topologies.md)können Sie folgende Aktionen ausführen:
1. [Windows Server-Knoten zu einem Flannel (Vxlan oder Host-gw) Cluster beitreten](#joining-a-flannel-cluster)
2. [Verknüpfen von Windows Server-Knoten zu einem Cluster mit einen ToR-switch](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Beitreten zu einem Cluster Flannel ###
Es ist eine Sammlung von Flannel Bereitstellungsskripts auf [dieses Microsoft-Repository](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/overlay) , mit der Sie diesen Knoten zum Cluster hinzuzufügen.

Herunterladen des [Flannel start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) -Skripts, dessen Inhalt werden, um extrahiert sollten `C:\k`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/start.ps1 -o c:\k\start.ps1
```

Wenn Sie [Ihre Windows-Knoten vorbereitet](#preparing-a-windows-node), und Ihr `c:\k` Verzeichnis sieht folgendermaßen aus, können Sie die Knoten hinzuzufügen.

![Text](./media/flannel-directory.png)

#### <a name="join-node"></a>Verknüpfen von Knoten #### 
Um den Prozess zum Hinzufügen eines Windows-Knotens zu vereinfachen, müssen Sie nur ein einzelnes Windows-Skript zum Starten führen `kubelet`, `kube-proxy`, `flanneld`, und Verbinden des Knotens.

> [!Note]
> [Start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) verweist auf [install.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/install.ps1), welches die zusätzliche Dateien wie z. B. Herunterladen der `flanneld` ausführbare Dateien und die [dockerfile-Datei für Infrastruktur Pod](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *und installieren Sie diese für Sie*. Für den überlagerungsnetzwerkmodus wird der [Firewall](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/helper.psm1#L111) für lokale UDP-Port 4789 geöffnet werden. Möglicherweise gibt es mehrere Powershell-Fenster wird geöffnet/sowie einige Sekunden Netzwerkausfall geschlossen, während der neuen externe vSwitch für das Netzwerk Pod erstmalig erstellt wird.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -NetworkMode <network mode>  -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Log directory>
```
# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
Die IP-Adresse zugewiesen, mit dem Windows-Knoten. Sie können `ipconfig` , diesen zu finden.

|  |  | 
|---------|---------|
|Parameter     | `-ManagementIP`        |
|Standardwert    | entfällt **required**        |

# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
Den Netzwerkmodus `l2bridge` (Flannel Host-gw) oder `overlay` (Flannel Vxlan) als eine [Netzwerk-Lösung](./network-topologies.md)ausgewählt.

> [!Important] 
> `overlay` Netzwerkmodus (Flannel Vxlan) erfordert Kubernetes v1.14-Binärdateien oder höher.

|  |  | 
|---------|---------|
|Parameter     | `-NetworkMode`        |
|Standardwert    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
Die [Cluster-Subnetz-Bereich](./getting-started-kubernetes-windows.md#cluster-subnet-def).

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
[Kubernetes DNS-Dienst-IP](./getting-started-kubernetes-windows.md#kube-dns-def).

|  |  | 
|---------|---------|
|Parameter     | `-KubeDnsServiceIP`        |
|Standardwert    | `10.96.0.10`        |


# [<a name="interfacename"></a>Schnittstellenname](#tab/InterfaceName)
Der Name der Netzwerkschnittstelle des Hosts Windows. Sie können `ipconfig` , diesen zu finden.

|  |  | 
|---------|---------|
|Parameter     | `-InterfaceName`        |
|Standardwert    | `Ethernet`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
Das Verzeichnis, in denen Kubelet und Kube-Proxy-Protokolle in ihren jeweiligen Ausgabedateien umgeleitet wurden.

|  |  | 
|---------|---------|
|Parameter     | `-LogDir`        |
|Standardwert    | `C:\k`        |


---

> [!tip]
> Sie notiert haben bereits Sie die Clustersubnetz, Dienst-Subnetz, und Kube-DNS-IP-vom Linux-Master [früheren](./creating-a-linux-master.md#collect-cluster-information)

Nach dem Ausführen dieser sollten Sie in der Lage sein:
  * Zeigen Sie verbundene Windows-Knoten mit an `kubectl get nodes`
  * Finden Sie unter 3 öffnen, eins für Powershell-Fenstern `kubelet`, eine für `flanneld`, und ein weiteres für `kube-proxy`
  * Finden Sie unter Host-Agent-Prozesse für `flanneld`, `kubelet`, und `kube-proxy` auf dem Knoten ausgeführt

Falls erfolgreich, weiterhin den [nächsten Schritten](#next-steps).

## <a name="joining-a-tor-cluster"></a>Beitreten zu einem Cluster ToR ##
> [!NOTE]
> Wenn Sie Flannel als Ihr Netzwerk Lösung [zuvor](./network-topologies.md#flannel-in-host-gateway-mode)ausgewählt haben, können Sie diesen Abschnitt überspringen.

Zu diesem Zweck müssen Sie die Anweisungen für die [Einrichtung von Windows Server-Container auf Kubernetes für Upstream L3-Routingtopologie](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology)folgen. Dazu gehören, und stellen Sie sicher, dass Sie den upstream Router neu konfigurieren, z. B., die der Pod CIDR Präfix eines Knotens zugewiesen, dessen jeweiligen Knoten IP zugeordnet ist.

Den neuen Knoten vorausgesetzt als "Bereit" aufgeführt ist, nach `kubectl get nodes`Kubelet + Kube-Proxy ausgeführt wird, und Sie den upstream ToR Router neu konfiguriert haben, sind Sie bereit für den nächsten Schritten.

## <a name="next-steps"></a>Nächste Schritte ##
In diesem Abschnitt behandelt wir Windows-Worker zu unserem Kubernetes-Cluster beitreten. Jetzt sind Sie bereit für Schritt 5:

> [!div class="nextstepaction"]
> [Linux-Worker beitreten](./joining-linux-workers.md)

Auch wenn Ihnen keine passen alle Linux-Worker überspringen mit Schritt 6:

> [!div class="nextstepaction"]
> [Bereitstellen von Kubernetes-Ressourcen](./deploying-resources.md)
