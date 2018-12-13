---
title: Verknüpfen von Windows-Knoten
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Wenn einen Windows-Knoten zu einem Kubernetes-Cluster mit v1.12.
keywords: Kubernetes, 1.12, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 8051270cac6178bad9adf9a8ef9e2324932f7d01
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/08/2018
ms.locfileid: "6179073"
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

Starten Sie den dockerdienst manuell:

```powershell
Start-Service docker
```

### <a name="create-the-pause-infrastructure-image"></a>Erstellen Sie das Bild "Anhalten" (Infrastruktur) ###
> [!Important]
> Es ist wichtig, darauf achten, dass widersprüchlichen Container-Images. nicht über das erwartete Tag kann dazu führen, dass ein `docker pull` eines inkompatiblen Container-Images, [Bereitstellungsprobleme](./common-problems.md#when-deploying-docker-containers-keep-restarting) verursachen, z. B. auf unbestimmte `ContainerCreating` Status.

Nachdem Sie nun `docker` installiert haben, müssen Sie ein „pause”-Image vorbereiten, das von Kubernetes zum Vorbereiten der Infrastruktur-Pods verwendet wird. Es gibt drei Schritte: 
  1. [Ziehen das Bild](#pull-the-image)
  2. [Markieren Sie es](#tag-the-image) als Microsoft / Nanoserver:latest
  3. und [ausgeführt wird](#run-the-container)


#### <a name="pull-the-image"></a>Ziehen Sie das Bild ####     
 Ziehen Sie das Image für Ihre spezifischen Windows-Version. Beispielsweise, wenn Sie Windows Server 2019 ausgeführt werden:

 ```powershell
docker pull microsoft/nanoserver:1803
 ```

#### <a name="tag-the-image"></a>Markieren Sie das Bild ####
Suchen Sie die dockerfile-Dateien Sie weiter unten in diesem Handbuch verwenden für die `:latest` image-Tag. Markieren Sie das Nanoserver-Bild, das Sie genau wie folgt gezogen:

```powershell
docker tag microsoft/nanoserver:1803 microsoft/nanoserver:latest
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
Kopieren Sie die Zertifikatdatei Kubernetes (`$HOME/.kube/config`) [vom Master](./creating-a-linux-master.md#collect-cluster-information) neuen `C:\k` Verzeichnis.

#### <a name="download-kubernetes-binaries"></a>Herunterladen von Kubernetes-Binärdateien ####
Um Kubernetes ausgeführt werden können, müssen Sie zuerst herunterladen der `kubectl`, `kubelet`, und `kube-proxy` Binärdateien. Sie können diese über die Links im Herunterladen der `CHANGELOG.md` Datei mit den [neuesten Versionen](https://github.com/kubernetes/kubernetes/releases/).
 - Hier sind z. B. die [v1.12 Knoten-Binärdateien](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.12.md#node-binaries).
 - Verwenden Sie ein Tool wie [7-Zip](http://www.7-zip.org/) zum Extrahieren des Archivs, und platzieren Sie die Binärdateien in `C:\k\`.

#### <a name="optional-setup-kubectl-on-windows"></a>(Optional) Setup-Kubectl unter Windows ####
Sollten Sie möchten Cluster von Windows zu steuern, Sie können dazu das `kubectl` Befehl. Erste, um sicherzustellen, `kubectl` außerhalb des verfügbaren der `C:\k\` Verzeichnis und Ändern der `PATH` Umgebungsvariable:

```powershell
$env:Path += ";C:\k"
```

Wenn Sie diese Änderung permanent machen möchten, ändern Sie die Variable im Computerziel:

```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\k", [EnvironmentVariableTarget]::Machine)
```

Als Nächstes werden wir stellen Sie sicher, dass der [Cluster-Zertifikat](#copy-kubernetes-certificate) gültig ist. Um den Speicherort festzulegen, in denen `kubectl` nach der Konfigurationsdatei sucht, übergeben Sie die `--kubeconfig` Parameter oder Ändern der `KUBECONFIG` -Umgebungsvariable. Wenn die Konfiguration beispielsweise unter `C:\k\config` gespeichert ist:

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

Sie sollten überprüfen die Position Kubeconfig oder versuchen Sie es erneut zu kopieren.

Wenn Sie keine Fehler angezeigt ist der Knoten jetzt bereit, dem Cluster beizutreten.

## <a name="joining-the-windows-node"></a>Wenn die Windows-Knoten ##
Je nach [Netzwerk-Lösung, die Sie ausgewählt haben](./network-topologies.md)können Sie folgende Aktionen ausführen:
1. [Windows Server-Knoten zu einem Flannel Cluster beitreten](#joining-a-flannel-cluster)
2. [Verknüpfen von Windows Server-Knoten zu einem Cluster mit einen ToR-switch](#joining-a-tor-cluster)

### <a name="joining-a-flannel-cluster"></a>Beitreten zu einem Cluster Flannel ###
Es ist eine Sammlung von Flannel Bereitstellungsskripts auf [dieses Microsoft-Repository](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge) , mit dem Sie diesen Knoten zum Cluster hinzuzufügen.

Sie können die ZIP-Datei direkt [hier](https://github.com/Microsoft/SDN/archive/master.zip) herunterladen. Ist das einzige, was Sie benötigen die `Kubernetes/flannel/l2bridge` Verzeichnis und dessen Inhalt werden, um extrahiert sollten `C:\k\`:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
wget https://github.com/Microsoft/SDN/archive/master.zip -o master.zip
Expand-Archive master.zip -DestinationPath master
mv master/SDN-master/Kubernetes/flannel/l2bridge/* C:/k/
rm -recurse -force master,master.zip
```

Darüber hinaus sollten Sie sicherstellen, dass das Clustersubnetz (z. B. Kontrollkästchen "10.244.0.0/16") in korrekt ist:
- [NET-conf.json](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/net-conf.json)


Wenn Sie [Ihre Windows-Knoten vorbereitet](#preparing-a-windows-node), und Ihr `c:\k` Verzeichnis sieht folgendermaßen aus, können Sie den Knoten hinzuzufügen.

![Text](./media/flannel-directory.png)

#### <a name="join-node"></a>Verknüpfen von Knoten #### 
Um den Prozess zum Hinzufügen eines Windows-Knotens zu vereinfachen, müssen Sie nur ein einzelnes Windows-Skript zum Starten führen `kubelet`, `kube-proxy`, `flanneld`, und Verbinden des Knotens.

> [!Note]
> [Dieses Skript](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1) wird aktualisiert, z. B. zusätzliche Dateien herunterladen `flanneld` ausführbare Dateien und die [dockerfile-Datei für die Infrastruktur Pod](https://github.com/Microsoft/SDN/blob/master/Kubernetes/windows/Dockerfile) *, und führen Sie diese für Sie*. Möglicherweise gibt es mehrere Powershell-Fenster wird sowie einige Sekunden Netzwerkausfall geöffnet/geschlossen, während die neuen externe vSwitch für das l2bridge-Pod-Netzwerk erstmalig erstellt wird.

```powershell
cd c:\k
.\start.ps1 -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster CIDR> -ServiceCIDR <Service CIDR> -KubeDnsServiceIP <Kube-dns Service IP> 
```

> [!tip]
> Sie notiert haben bereits Sie die Clustersubnetz, Dienst-Subnetz, und Kube-DNS-IP-Master-Linux [früher](./creating-a-linux-master.md#collect-cluster-information)

Nach dem Ausführen dieser sollten Sie in der Lage sein:
  * Zeigen Sie verbundene mithilfe von Windows-Knoten an `kubectl get nodes`
  * Finden Sie unter 3 öffnen, eins für Powershell-Fenstern `kubelet`, eine für `flanneld`, und ein weiteres für `kube-proxy`
  * Finden Sie unter Host-Agent-Prozesse für `flanneld`, `kubelet`, und `kube-proxy` auf dem Knoten ausgeführt


## <a name="joining-a-tor-cluster"></a>Beitreten zu einem Cluster ToR ##
> [!NOTE]
> Wenn Sie Flannel als Ihr Netzwerk Lösung [zuvor](./network-topologies.md#flannel-in-host-gateway-mode)ausgewählt haben, können Sie diesen Abschnitt überspringen.

Zu diesem Zweck müssen Sie die Anweisungen für die [Einrichtung von Windows Server-Container auf Kubernetes für Upstream L3-Routingtopologie](https://kubernetes.io/docs/getting-started-guides/windows/#for-1-upstream-l3-routing-topology-and-2-host-gateway-topology)folgen. Dazu gehören, und stellen Sie sicher, dass Sie den upstream Router neu konfigurieren, z. B., die der Pod CIDR Präfix eines Knotens zugewiesen, dessen jeweiligen Knoten IP zugeordnet ist.

Vorausgesetzt, des neuen Knotens als "Bereit" aufgeführt ist, indem Sie `kubectl get nodes`Kubelet + Kube-Proxy ausgeführt wird, und Sie den Router neu upstream ToR konfiguriert haben, sind Sie bereit für den nächsten Schritten.

## <a name="next-steps"></a>Nächste Schritte ##
In diesem Abschnitt behandelt wir Windows-Worker zu unserem Kubernetes-Cluster beitreten. Jetzt können Sie unter Schritt 5:

> [!div class="nextstepaction"]
> [Linux-Worker beitreten](./joining-linux-workers.md)

Auch wenn Ihnen keine passen alle Linux-Worker überspringen mit Schritt 6:

> [!div class="nextstepaction"]
> [Bereitstellen von Kubernetes-Ressourcen](./deploying-resources.md)