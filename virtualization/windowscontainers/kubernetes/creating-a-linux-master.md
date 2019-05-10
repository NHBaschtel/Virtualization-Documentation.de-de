---
title: Erstellen eines neuen Kubernetes-Master
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Erstellen einen neuen Kubernetes Cluster-Master.
keywords: Kubernetes, 1,14, master, linux
ms.openlocfilehash: b1ec23b039ce6f5c42859452ecf3a8a5b35e006c
ms.sourcegitcommit: aaf115a9de929319cc893c29ba39654a96cf07e1
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 05/10/2019
ms.locfileid: "9622955"
---
# <a name="creating-a-kubernetes-master"></a>Erstellen eines Kubernetes-Masters #
> [!NOTE]
> Dieses Handbuch wurde auf Kubernetes v1.14 überprüft. Aufgrund der Version auftretenden Unbeständigkeit von Kubernetes Version kann in diesem Abschnitt Annahmen machen, die nicht für alle zukünftigen Versionen "true" enthalten. Offiziellen Dokumentation für die Initialisierung von Kubernetes-Master mit Kubeadm finden Sie [hier](https://kubernetes.io/docs/setup/independent/install-kubeadm/). Aktivieren Sie einfach [scheduling vermischten OS-Abschnitt](#enable-mixed-os-scheduling) oben in diesem an.

> [!NOTE]  
> Ein vor kurzem aktualisierter Linux-Computer ist erforderlich, um nachvollziehen; Kubernetes master-Ressourcen wie [Kube-Dns](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), [Kube-Scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)und [Kube-Apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) nicht zu Windows noch portiert wurden. 

> [!tip]
> Die Linux-Anweisungen sind für **Ubuntu 16.04**zugeschnitten. Andere Linux-Distributionen, die zum Ausführen von Kubernetes zertifiziert sollte auch entsprechende Befehle bieten, die Sie ersetzen können. Sie können auch erfolgreich mit Windows ausgeführt werden.


## <a name="initialization-using-kubeadm"></a>Initialisierung mit kubeadm ##
Sofern nicht explizit anders angegeben, führen Sie alle Befehle unten als **Root**aus.

Rufen Sie zunächst in einer erhöhten Root-Shell:

```bash
sudo –s
```

Stellen Sie sicher, dass Ihre Computer auf dem neuesten Stand ist:

```bash
apt-get update -y && apt-get upgrade -y
```

### <a name="install-docker"></a>Installieren von Docker ###
Um Container verwendet werden können, benötigen Sie eine Container-Engine, z. B. Docker. Um die neueste Version zu erhalten, können Sie [diese Anweisungen](https://docs.docker.com/install/linux/docker-ce/ubuntu/) für die Installation von Docker verwenden. Sie können überprüfen, ob diese Docker ordnungsgemäß installiert ist, mit einem `hello-world` Container:

```bash
docker run hello-world
```

### <a name="install-kubeadm"></a>Installieren von kubeadm ###
Herunterladen `kubeadm` Binärdateien für Ihre Linux-Distributionen und Initialisieren des Clusters.

> [!Important]  
> Je nach Linux-Distribution müssen Sie möglicherweise ersetzen `kubernetes-xenial` unten mit dem richtigen [Codename](https://wiki.ubuntu.com/Releases).

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

### <a name="prepare-the-master-node"></a>Vorbereiten der master-Knotens ###
Kubernetes unter Linux erfordert Swap-Bereich deaktiviert ist:

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>Initialisieren Sie master ###
Notieren Sie Ihre Clustersubnetz (z. B. 10.244.0.0/16) und die Dienst-Subnetz (z. B. 10.96.0.0/12), und initialisieren Sie Ihre Master Kubeadm verwenden:

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

Dieser Vorgang kann einige Minuten dauern. Nachdem abgeschlossen, sollten Sie sehen einen Bildschirm wie dieser bestätigt, dass Ihre Master initialisiert wurde:

![Text](media/kubeadm-init.png)

> [!tip]
> Sie sollten diesen Kubeadm Beitritt Befehl beachten. Sollte das Token Kubeadm ablaufen, können Sie `kubeadm token create --print-join-command` um ein neues Token zu erstellen.

> [!tip]
> Wenn Sie eine gewünschte Kubernetes-Version Sie verwenden möchten, übergeben Sie die `--kubernetes-version` Flag zum Kubeadm.

Wir sind noch nicht fertig. Mit `kubectl` wie ein normaler Benutzer, führen Sie die folgenden __**in einer unelevated, nicht-Root-Benutzer-Shell**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Jetzt können Sie Kubectl bearbeiten oder Anzeigen von Informationen über Ihre Cluster.

### <a name="enable-mixed-os-scheduling"></a>Aktivieren Sie die Planung vermischten OS ###
Standardmäßig werden bestimmte Kubernetes-Ressourcen so geschrieben, dass sie auf allen Knoten geplant sind. Allerdings möchten nicht in einer Multi-BS-Umgebung wir Linux-Ressourcen zu beeinträchtigen, oder auf Windows-Knoten und umgekehrt Double geplant werden. Aus diesem Grund müssen wir [Selektoren](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) Beschriftungen anwenden. 

In diesem Zusammenhang werden wir uns hier Linux patch Kube-Proxy [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) Linux nur bestimmt.

Zunächst erstellen wir ein Verzeichnis zum Speichern von .yaml Manifestdateien:
```bash
mkdir -p kube/yaml && cd kube/yaml
```

Überprüfen Sie, ob die Updatestrategie von `kube-proxy` DaemonSet auf [RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/)festgelegt ist:

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

Als Nächstes Patchen Sie der DaemonSet durch [Diese Selektoren](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) herunterladen und wenden Sie an, um nur Linux als Ziel:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Sobald der erfolgreich, sollten Sie "Knotenselektoren" angezeigt, der `kube-proxy` und alle anderen DaemonSets festgelegt `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![Text](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>Sammeln von Clusterinformationen ###
So verknüpfen Sie erfolgreich zukünftige Knoten mit dem Master, sollten Sie die folgende Informationen nachzuverfolgen:
  1. `kubeadm join` Befehl Ausgabe ([hier](#initialize-master))
    * Beispiel: `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. Cluster-Subnetz, die während der definierten `kubeadm init` ([hier](#initialize-master))
    * Beispiel: `10.244.0.0/16`
  3. Dienst-Subnetz, die während der definierten `kubeadm init` ([hier](#initialize-master))
    * Beispiel: `10.96.0.0/12`
    * Kann auch mit gefunden werden `kubectl cluster-info dump | grep -i service-cluster-ip-range`
  4. Kube-DNS-Dienst-IP 
    * Beispiel: `10.96.0.10`
    * Finden Sie in "Cluster-IP-" Feld mit `kubectl get svc/kube-dns -n kube-system`
  5. Kubernetes `config` Datei nach der `kubeadm init` ([hier](#initialize-master)). Wenn Sie die Anweisungen befolgt, kann dies in den folgenden Pfaden gefunden werden:
    * `/etc/kubernetes/admin.conf`
    * `$HOME/.kube/config`

## <a name="verifying-the-master"></a>Überprüfen des Masters ##
Nach ein paar Minuten sollte das System folgenden Status aufweisen:

  - Unter `kubectl get pods -n kube-system`, werden für die [Kubernetes master-Komponenten](https://kubernetes.io/docs/concepts/overview/components/#master-components) in Pods `Running` Zustand.
  - Aufrufen `kubectl cluster-info` zeigt Informationen über den Kubernetes API-Masterserver neben den Add-Ons DNS an.
  
> [!tip]
> Da Kubeadm nicht Netzwerk einrichten DNS-Pods möglicherweise weiterhin `ContainerCreating` oder `Pending` Zustand. Sie werden zu wechseln, `Running` Status nach dem [Auswählen einer Lösung](./network-topologies.md).

## <a name="next-steps"></a>Nächste Schritte ## 
In diesem Abschnitt behandelt wir zum Einrichten eines Kubeadm mit Kubernetes-Masters. Jetzt sind Sie bereit für Schritt 3:

> [!div class="nextstepaction"]
> [Auswählen einer Lösung](./network-topologies.md)