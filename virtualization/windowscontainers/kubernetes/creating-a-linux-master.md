---
title: Erstellen eines neuen Kubernetes-Master
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: get-started-article
ms.prod: containers
description: Erstellen Sie einen neuen Kubernetes Cluster-Master.
keywords: Kubernetes, 1.9, Master, Linux
ms.openlocfilehash: 3ea338f7af3dd921731fce0ec5a8b2cf8c4fef0c
ms.sourcegitcommit: f542e8c95b5bb31b05b7c88f598f00f76779b519
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/01/2018
---
# <a name="kubernetes-master--from-scratch"></a>Erstellen eines neuen Kubernetes-Master #
Diese Seite führt Sie durch die manuelle Bereitstellung eines Kubernetes-Master von Anfang bis Ende.

Hierzu ist ein vor kurzem aktualisierter, Ubuntu ähnlicher Linux-Computer erforderlich. Windows wird hier überhaupt nicht verwendet; Binärdateien werden aus Linux kompiliert.


> [!Warning]  
> Aufgrund der je nach Version auftretenden Unbeständigkeit von Kubernetes kann dieses Handbuch Annahmen machen, die in Zukunft nicht mehr erfüllt werden können.


## <a name="preparing-the-master"></a>Vorbereiten des Masters ##
Installieren Sie zuerst alle erforderlichen Komponenten:

```bash
sudo apt-get install curl git build-essential docker.io conntrack python2.7
```

Wenn Sie einen Proxy verwenden, definieren Sie die Umgebungsvariablen für die aktuelle Sitzung:
```bash
HTTP_PROXY=http://proxy.example.com:80/
HTTPS_PROXY=http://proxy.example.com:443/
http_proxy=http://proxy.example.com:80/
https_proxy=http://proxy.example.com:443/
```
Wenn Sie diese Einstellung dauerhaft machen möchten, fügen Sie die Variablen zu /etc/environment hinzu (zur Anwendung der Änderung ist eine Abmeldung und erneute Anmeldung erforderlich).

Eine Sammlung der Skripts, die beim Setup behilflich sind, befindet sich in [diesem Repository](https://github.com/Microsoft/SDN/tree/master/Kubernetes/linux). Exportieren Sie diese auf `~/kube/`. Da das gesamte Verzeichnis für zahlreiche Docker-Container in zukünftigen Schritten bereitgestellt wird, sollten Sie die in diesem Handbuch beschriebene Struktur genauso beibehalten.

```bash
mkdir ~/kube
mkdir ~/kube/bin
git clone https://github.com/Microsoft/SDN /tmp/k8s 
cd /tmp/k8s/Kubernetes/linux
chmod -R +x *.sh
chmod +x manifest/generate.py
mv * ~/kube/
```


### <a name="installing-the-linux-binaries"></a>Installieren der Linux-Binärdateien ###

> [!Note]  
> Um Patches oder den neusten Kubernetes Code zu verwenden anstatt integrierte Binärdateien herunterzuladen, lesen Sie [diese Seite](./compiling-kubernetes-binaries.md).

Laden Sie die offiziellen Linux-Binärdateien aus der [Kubernetes-Hauptseite](https://github.com/kubernetes/kubernetes/releases/tag/v1.9.1) herunter und installieren Sie diese wie folgt:

```bash
wget -O kubernetes.tar.gz https://github.com/kubernetes/kubernetes/releases/download/v1.9.1/kubernetes.tar.gz
tar -vxzf kubernetes.tar.gz 
cd kubernetes/cluster 
# follow the prompts from this command, the defaults are generally fine:
./get-kube-binaries.sh
cd ../server
tar -vxzf kubernetes-server-linux-amd64.tar.gz 
cd kubernetes/server/bin
cp hyperkube kubectl ~/kube/bin/
```

Fügen Sie `$PATH` die Binärdateien hinzu, damit diese von überall ausgeführt werden können. Beachten Sie, dass dies nur den Pfad für die Sitzung festlegt. Fügen Sie diese Zeile dem `~/.profile` als permanente Einstellung hinzu.

```bash
$ PATH="$HOME/kube/bin:$PATH"
```

### <a name="install-cni-plugins"></a>Installieren Sie CNI-Plug-Ins ###
Für Kubernetes-Netzwerke sind grundlegende CNI-Plug-Ins erforderlich. Diese können [hier](https://github.com/containernetworking/plugins/releases) heruntergeladen werden und sollten auf `/opt/cni/bin/` extrahiert werden:

```bash
DOWNLOAD_DIR="${HOME}/kube/cni-plugins"
CNI_BIN="/opt/cni/bin/"
mkdir ${DOWNLOAD_DIR}
cd $DOWNLOAD_DIR
curl -L $(curl -s https://api.github.com/repos/containernetworking/plugins/releases/latest | grep browser_download_url | grep 'amd64.*tgz' | head -n 1 | cut -d '"' -f 4) -o cni-plugins-amd64.tgz
tar -xvzf cni-plugins-amd64.tgz
sudo mkdir -p ${CNI_BIN}
sudo cp -r !(*.tgz) ${CNI_BIN}
ls ${CNI_BIN}
```


### <a name="certificates"></a>Zertifikate ###
Rufen Sie zunächst die lokale IP-Adresse ab, entweder über `ifconfig` oder:

```bash
$ ip addr show dev eth0
```

wenn der Schnittstellenname bekannt ist. Darauf wird in Verlauf dieses Prozesses oft verwiesen. Ein Festlegen auf eine Umgebungsvariable erleichtert den Vorgang. Der folgende Codeausschnitt legt dies vorübergehend fest. Wenn die Sitzung beendet wird oder die Befehlsshell geschlossen wird, muss sie erneut festgelegt werden.

```bash
$ MASTER_IP=10.123.45.67   # example! replace
```

Vorbereiten der Zertifikate, die für die Kommunikation der Knoten im Cluster verwendet werden:

```bash
cd ~/kube/certs
chmod u+x generate-certs.sh
./generate-certs.sh $MASTER_IP
```

### <a name="prepare-manifests--addons"></a>Vorbereiten der Manifeste und Add-Ons ###
Erstellen Sie einen Satz an YAML-Dateien, die durch ein Übergeben der Master-IP-Adresse und der *vollständigen* Cluster-CIDR an das Python-Skript im `manifest`-Ordner die Kubernetes Systempods angeben:

```bash
cd ~/kube/manifest
./generate.py $MASTER_IP --cluster-cidr 192.168.0.0/16
```

Verschieben Sie das Python-Skript, damit Kubernetes dies nicht für ein Manifest hält. Andernfalls treten später Probleme auf.

> [!Important]  
> Wenn die Kubernetes-Version von diesem Handbuch abweicht, verwenden Sie die verschiedenen Versionsverwaltungskennzeichen im Skript (z.B. `--api-version`), um [das Bild anzupassen](https://console.cloud.google.com/gcr/images/google-containers/GLOBAL/hyperkube-amd64), das die Pods bereitstellen. Nicht alle Manifeste verwenden dasselbe Bild und verfügen über unterschiedliche Versionsschemas (insbesondere `etcd` und Add-On-Manager).


#### <a name="manifest-customization"></a>Anpassung des Manifests ####
Hier können spezifische Änderungen des Setup wünschenswert sein. Es kann beispielsweise eine manuelle Zuweisung von Subnetzen zu Knoten erforderlich werden, anstatt diese automatisch von Kubernetes verwaltet werden zu lassen. Diese spezifische Konfiguration verfügt über eine Option im Skript (siehe `--help` für eine Erläuterung der `--im-sure`-Parameter):

```bash
./generate.py $MASTER_IP --im-sure
```

Andere benutzerdefinierte Konfigurationsoptionen erfordern eine manuelle Änderung der erstellten Manifeste.


### <a name="configure--run-kubernetes"></a>Konfigurieren und Ausführen von Kubernetes ###
Konfigurieren Sie Kubernetes, um die generierten Zertifikate zu verwenden. Dadurch entsteht eine Konfiguration unter `~/.kube/config`:

```bash
cd ~/kube
./configure-kubectl.sh $MASTER_IP
```

Kopieren Sie die Datei anschließend dorthin, wo die Pods sich später befinden sollen:

```bash
mkdir ~/kube/kubelet
sudo cp ~/.kube/config ~/kube/kubelet/
```

Der Kubernetes "Client", `kubelet` kann jetzt gestartet werden. Die folgenden Skripts werden beide auf unbestimmte Zeit ausgeführt. Öffnen Sie danach eine andere Terminalsitzung, um Ihre Arbeit fortzusetzen:

```bash
cd ~/kube
sudo ./start-kubelet.sh
```

Führen Sie das Skript „Kubeproxy” aus und übergeben Sie unvollständige Cluster-CIDR:

```bash
cd ~/kube
sudo ./start-kubeproxy.sh 192.168
```


> [!Important]  
> Dies ist die *vollständige* erwartete /16 CIDR, worunter sich der Knoten befindet, *auch wenn kein Kubernetes-Datenverkehr auf diesem CIDR vorhanden ist.* Kubeproxy gilt *nur* für Kubernetes-Datenverkehr auf das *Dienst*-Subnetz, daher sollte es den Datenverkehr anderer Hosts nicht beeinträchtigen.

> [!Note]  
> Diese Skripts können als Daemon genutzt werden. In diesem Handbuch wird nur die manuelle Ausführung behandelt, da dies während des Setups zum Abfangen von Fehlern nützlich ist.


## <a name="verifying-the-master"></a>Überprüfen des Masters ##
Nach ein paar Minuten sollte das System folgenden Status aufweisen:

  - `docker ps` sollte ungefähr 23 Worker und Pod-Container enthalten.
  - Das Aufrufen von `kubectl cluster-info` zeigt Informationen über den Kubernetes API-Masterserver neben den Add-Ons für DNS und Heapster an.
  - `ifconfig` zeigt eine neue Schnittstelle `cbr0` mit dem ausgewählten Cluster-CIDR an.

