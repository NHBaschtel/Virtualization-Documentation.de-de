---
title: Erstellen eines neuen Kubernetes-Master
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Erstellen einen neuen Kubernetes Cluster-Master.
keywords: Kubernetes, 1.12, master, linux
ms.openlocfilehash: 2bbcf2d382f20d140c73d9b34cf0f13a74debdfa
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178853"
---
# <a name="creating-a-kubernetes-master"></a>Erstellen eines Kubernetes-Masters #
> [!NOTE]
> Dieses Handbuch wurde auf Kubernetes v1.12 überprüft. Aufgrund der Unbeständigkeit von Kubernetes Version zu Version kann in diesem Abschnitt Annahmen machen, die nicht für alle zukünftigen Versionen "true" enthalten. Offiziellen Dokumentation für die Initialisierung von Kubernetes-Master mit Kubeadm finden Sie [hier](https://kubernetes.io/docs/setup/independent/install-kubeadm/). Aktivieren Sie einfach [scheduling vermischten OS-Abschnitt](#enable-mixed-os-scheduling) oben in diesem an.

> [!NOTE]  
> Ein vor kurzem aktualisierter Linux-Computer ist erforderlich, um nachvollziehen; Kubernetes master-Ressourcen wie [Kube-Dns](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), [Kube-Scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)und [Kube-Apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) nicht zu Windows noch portiert wurden. 

> [!tip]
> Die Linux-Anweisungen sind für **Ubuntu 16.04**zugeschnitten. Andere Linux-Distributionen, die zum Ausführen von Kubernetes-Zertifizierung sollten auch entsprechende Befehle anbieten, die Sie ersetzen können. Sie können auch erfolgreich mit Windows ausgeführt werden.


## <a name="initialization-using-kubeadm"></a>Initialisierung mit kubeadm ##
Sofern nicht explizit anders angegeben, führen Sie alle Befehle unten als **Stamm**.

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
Notieren Sie Ihre Clustersubnetz (z. B. 10.244.0.0/16) und die Dienst-Subnetz (z. B. 10.96.0.0/12), und initialisieren Sie Ihre Master mit Kubeadm zu:

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

Dieser Vorgang kann einige Minuten dauern. Sobald abgeschlossen ist, sollten Sie sehen einen Bildschirm wie dieser bestätigt, dass Ihre Master initialisiert wurde:

![Text](media/kubeadm-init.png)

> [!tip]
> Notieren Sie Kubeadm Beitritt Ausgabe des Befehls in der Abbildung oben *jetzt* bevor verloren geht.

> [!tip]
> Wenn Sie eine gewünschte Kubernetes-Version Sie verwenden möchten, übergeben Sie die `--kubernetes-version` Kubeadm-Flag.

Wir sind noch nicht fertig. Sie verwenden `kubectl` wie ein normaler Benutzer, führen Sie die folgenden __**in einer unelevated, nicht-Root-Benutzer-Shell**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Jetzt können Sie Kubectl bearbeiten oder Anzeigen von Informationen über Ihre Cluster.

### <a name="enable-mixed-os-scheduling"></a>Aktivieren Sie die Planung vermischten OS ###
Standardmäßig werden bestimmte Ressourcen Kubernetes so geschrieben, dass sie auf allen Knoten geplant sind. Allerdings möchten nicht in einer Multi-BS-Umgebung wir Linux-Ressourcen zu beeinträchtigen oder auf Windows-Knoten, und umgekehrt Double geplant werden. Aus diesem Grund müssen wir [Selektoren](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) Beschriftungen anwenden. 

In diesem Zusammenhang werden wir Linux patch Kube-Proxy [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) Linux nur bestimmt.

Überprüfen Sie, ob die Updatestrategie von `kube-proxy` DaemonSet auf [RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/)festgelegt ist:

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

Als Nächstes Patchen Sie der DaemonSet durch [Diese Selektoren](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) herunterladen und wenden Sie an, um nur Linux als Ziel:

```bash
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Sobald erfolgreich, sollten Sie "Knotenselektoren" angezeigt, der `kube-proxy` und alle anderen DaemonSets festgelegt `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![Text](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>Sammeln von Informationen über die Clusterknoten ###
So verknüpfen Sie zukünftige Knoten erfolgreich an den Master, sollten Sie die folgenden Informationen notieren:
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

  - Unter `kubectl get pods -n kube-system`, es werden für alle die [Kubernetes master-Komponenten](https://kubernetes.io/docs/concepts/overview/components/#master-components) in Pods `Running` Zustand.
  - Aufrufen von `kubectl cluster-info` zeigt Informationen über den Kubernetes API-Masterserver neben den Add-Ons DNS an.

## <a name="next-steps"></a>Nächste Schritte ## 
In diesem Abschnitt behandelt wir zum Einrichten eines Kubernetes-Masters Kubeadm verwenden. Jetzt sind Sie bereit für Schritt 3:

> [!div class="nextstepaction"]
> [Auswählen einer Lösung](./network-topologies.md)