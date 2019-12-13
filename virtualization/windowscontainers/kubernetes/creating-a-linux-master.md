---
title: Erstellen eines neuen Kubernetes-Master
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Erstellen eines Kubernetes-Cluster Masters
keywords: kubernetes, 1,14, Master, Linux
ms.openlocfilehash: b1ec23b039ce6f5c42859452ecf3a8a5b35e006c
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910420"
---
# <a name="creating-a-kubernetes-master"></a>Erstellen eines Kubernetes-Masters #
> [!NOTE]
> Dieses Handbuch wurde auf Kubernetes v 1.14 überprüft. Aufgrund der Volatilität von Kubernetes von Version zu Version kann dieser Abschnitt Annahmen treffen, die für alle zukünftigen Versionen nicht wahr sind. Die offizielle Dokumentation für die Initialisierung von Kubernetes Masters mithilfe von kubeadm finden Sie [hier](https://kubernetes.io/docs/setup/independent/install-kubeadm/). Aktivieren Sie einfach auf der Grundlage dieses Abschnitts den Bereich für die [Planung gemischter Betriebssysteme](#enable-mixed-os-scheduling) .

> [!NOTE]  
> Ein kürzlich aktualisierter Linux-Computer muss folgendermaßen befolgt werden. Kubernetes Master Ressourcen wie [Kube-DNS](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), [Kube-Scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)und [Kube-APISERVER](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) wurden noch nicht in Windows portiert. 

> [!tip]
> Die Linux-Anweisungen sind auf **Ubuntu 16,04**zugeschnitten. Andere Linux-Distributionen, die zum Ausführen von Kubernetes zertifiziert sind, sollten auch entsprechende Befehle anbieten, die Sie ersetzen können. Sie werden auch erfolgreich mit Windows zusammenarbeiten.


## <a name="initialization-using-kubeadm"></a>Initialisierung mit kubeadm ##
Wenn nicht anders angegeben, führen Sie alle unten aufgeführten Befehle als **root**aus.

Steigen Sie zunächst in eine Shell mit erhöhten Rechten ein:

```bash
sudo –s
```

Stellen Sie sicher, dass Ihr Computer auf dem neuesten Stand ist:

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>Installieren von Docker ###
Um Container verwenden zu können, benötigen Sie eine Container-Engine, z. b. Docker. Um die neueste Version zu erhalten, können Sie [diese Anweisungen](https://docs.docker.com/install/linux/docker-ce/ubuntu/) für die Docker-Installation verwenden. Sie können überprüfen, ob docker ordnungsgemäß installiert ist, indem Sie einen `hello-world` Container ausführen:

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Installieren von kubeadm ###
Laden Sie `kubeadm` Binärdateien für Ihre Linux-Distribution herunter, und initialisieren Sie Ihren Cluster.

> [!Important]  
> Abhängig von Ihrer Linux-Distribution müssen Sie möglicherweise `kubernetes-xenial` unten durch den richtigen [Codenamen](https://wiki.ubuntu.com/Releases)ersetzen.

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>Vorbereiten des Master Knotens ###
Kubernetes unter Linux erfordert, dass der Auslagerungs Bereich ausgeschaltet wird:

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>Master initialisieren ###
Notieren Sie sich Ihr Clustersubnetz (z. b. 10.244.0.0/16) und das dienstsubnetz (z. b. 10.96.0.0/12), und initialisieren Sie den Master mithilfe von kubeadm:

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

Dieser Vorgang kann einige Minuten dauern. Nach Abschluss des Vorgangs sollte ein Bildschirm wie dieser angezeigt werden, der bestätigt, dass Ihr Master initialisiert wurde:

![Text](media/kubeadm-init.png)

> [!tip]
> Beachten Sie diesen kubeadm Join-Befehl. Das kubeadm-Token läuft ab. Sie können `kubeadm token create --print-join-command` verwenden, um ein neues Token zu erstellen.

> [!tip]
> Wenn Sie über eine gewünschte Kubernetes-Version verfügen, die Sie verwenden möchten, können Sie das `--kubernetes-version`-Flag an kubeadm übergeben.

Das ist noch nicht abgeschlossen. Um `kubectl` als normalen Benutzer zu verwenden, führen Sie den folgenden Befehl  __**in einer nicht erhöhten, nicht stammenden Benutzershell aus** .__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Nun können Sie kubectl zum Bearbeiten oder Anzeigen von Informationen über Ihren Cluster verwenden.

### <a name="enable-mixed-os-scheduling"></a>Aktivieren von Zeitplänen für gemischte Betriebssysteme ###
Standardmäßig werden bestimmte Kubernetes-Ressourcen so geschrieben, dass Sie auf allen Knoten geplant sind. In einer Umgebung mit mehreren Betriebssystemen möchten wir jedoch nicht, dass Linux-Ressourcen auf Windows-Knoten beeinträchtigt werden, und umgekehrt. Aus diesem Grund müssen wir [nodäselector](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) -Bezeichnungen anwenden. 

In diesem Zusammenhang wird das Linux-Kube-Proxy- [daemonset](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) nur auf das Linux-Ziel aktualisiert.

Erstellen Sie zunächst ein Verzeichnis, in dem die YAML-Manifest-Dateien gespeichert werden:
```bash
mkdir -p kube/yaml && cd kube/yaml
```

Vergewissern Sie sich, dass die Aktualisierungs Strategie von `kube-proxy` daemonset auf [rollingupdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/)festgelegt ist:

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

Als nächstes Patchen Sie den daemonset, indem Sie [diesen nodeselector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) herunterladen und auf nur Ziel-Linux anwenden:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Nach erfolgreicher Ausführung sollten "knotenselektoren" von `kube-proxy` und alle anderen daemonsets auf festgelegt werden `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![Text](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>Sammeln von Cluster Informationen ###
Um dem Master erfolgreich zukünftige Knoten hinzuzufügen, sollten Sie die folgenden Informationen nachverfolgen:
  1. `kubeadm join` Befehl aus der Ausgabe ([hier](#initialize-master))
    * Beispiel: `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. Während der `kubeadm init` definierte Clustersubnetze ([hier](#initialize-master))
    * Beispiel: `10.244.0.0/16`
  3. Während der `kubeadm init` definierte dienstsubnetze ([hier](#initialize-master))
    * Beispiel: `10.96.0.0/12`
    * Kann auch mit `kubectl cluster-info dump | grep -i service-cluster-ip-range` gefunden werden.
  4. Kube-DNS-Dienst-IP 
    * Beispiel: `10.96.0.10`
    * Sie finden ihn im Feld "Cluster-IP" unter Verwendung `kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes `config` Datei generiert, die nach `kubeadm init` ([hier](#initialize-master)) generiert wurde. Wenn Sie die Anweisungen befolgt haben, finden Sie diese in den folgenden Pfaden:
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>Überprüfen des Masters ##
Nach ein paar Minuten sollte das System folgenden Status aufweisen:

  - Unter `kubectl get pods -n kube-system`werden Pods für die [Kubernetes-Master Komponenten](https://kubernetes.io/docs/concepts/overview/components/#master-components) in `Running` Zustand angezeigt.
  - Beim Aufrufen von `kubectl cluster-info` werden neben DNS-Addons auch Informationen zum Kubernetes-Master-API-Server angezeigt.
  
> [!tip]
> Da kubeadm keine Netzwerke eingibt, können sich DNS-Pods weiterhin in `ContainerCreating` oder `Pending` Status befinden. Nachdem Sie [eine Netzwerklösung ausgewählt](./network-topologies.md)haben, wechseln Sie zu `Running` Zustand.

## <a name="next-steps"></a>Nächste Schritte ## 
In diesem Abschnitt wird beschrieben, wie Sie einen Kubernetes-Master mithilfe von kubeadm einrichten. Nun sind Sie bereit für Schritt 3:

> [!div class="nextstepaction"]
> [Auswählen einer Netzwerklösung](./network-topologies.md)