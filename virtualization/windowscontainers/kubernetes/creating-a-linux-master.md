---
title: Erstellen eines neuen Kubernetes-Master
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Erstellen einen neuen Kubernetes Cluster-Master.
keywords: Kubernetes, 1,13, master, linux
ms.openlocfilehash: 8a3fb073616d115ab84e6cc36f0fb6cedbcf1f7d
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120428"
---
# <a name="creating-a-kubernetes-master"></a>Erstellen eines Kubernetes-Masters #
> [!NOTE]
> Dieses Handbuch wurde auf Kubernetes v1.13 überprüft. Aufgrund der Version auftretenden Unbeständigkeit von Kubernetes Version kann in diesem Abschnitt Annahmen machen, die nicht für alle zukünftigen Versionen "true" enthalten. Offiziellen Dokumentation für die Initialisierung von Kubernetes-Master mit Kubeadm finden Sie [hier](https://kubernetes.io/docs/setup/independent/install-kubeadm/). Aktivieren Sie einfach [scheduling vermischten OS-Abschnitt](#enable-mixed-os-scheduling) oben in diesem.

> [!NOTE]  
> Ein vor kurzem aktualisierter Linux-Computer ist erforderlich, um folgen; Kubernetes master-Ressourcen wie [Kube-Dns](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/), [Kube-Scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)und [Kube-Apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/) nicht zu Windows noch portiert wurden. 

> [!tip]
> Die Linux-Anweisungen sind für **Ubuntu 16.04**zugeschnitten. Andere Linux-Distributionen, die zum Ausführen von Kubernetes-Zertifizierung bieten auch entsprechenden Befehle, die Sie ersetzen können. Sie können auch erfolgreich mit Windows ausgeführt werden.


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
Um Container verwendet werden können, benötigen Sie eine Container-Engine, z. B. Docker. Um die neueste Version zu erhalten, können Sie [diese Anweisungen](https://docs.docker.com/install/linux/docker-ce/ubuntu/) für die Installation von Docker verwenden. Sie können überprüfen, ob diese Docker ordnungsgemäß installiert ist, durch Ausführen einer `hello-world` Container:

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

### <a name="prepare-the-master-node"></a>Vorbereiten des master-Knotens ###
Kubernetes unter Linux erfordert Swap-Bereich deaktiviert ist:

```bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a 
```

### <a name="initialize-master"></a>Initialisieren von master ###
Notieren Sie Ihre Clustersubnetz (z. B. 10.244.0.0/16) und die Dienst-Subnetz (z. B. 10.96.0.0/12), und initialisieren Sie Ihre Master Kubeadm verwenden:

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12
```

Dieser Vorgang kann einige Minuten dauern. Nachdem abgeschlossen, sollten Sie finden Sie in einem Bildschirm, wie diese bestätigen, dass Ihre Master initialisiert wurde:

![Text](media/kubeadm-init.png)

> [!tip]
> Sie sollten diesen Kubeadm Join Befehl beachten. Sollte das Token Kubeadm ablaufen, können Sie `kubeadm token create --print-join-command` um ein neues Token zu erstellen.

> [!tip]
> Wenn Sie eine gewünschte Kubernetes-Version Sie verwenden möchten, übergeben Sie die `--kubernetes-version` Kubeadm-Flag.

Wir sind noch nicht fertig. Mit `kubectl` wie ein normaler Benutzer, führen Sie die folgenden __**in einer unelevated, nicht-Root-Benutzer-Shell**__

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Jetzt können Sie Kubectl bearbeiten oder Anzeigen von Informationen zum Cluster.

### <a name="enable-mixed-os-scheduling"></a>Gemischt-OS Zeitplan aktivieren ###
Standardmäßig werden bestimmte Ressourcen Kubernetes so geschrieben, dass sie auf allen Knoten geplant sind. Jedoch möchten nicht in einer Multi-BS-Umgebung wir Linux-Ressourcen zu beeinträchtigen auf Windows-Knoten, und umgekehrt Double geplant werden. Aus diesem Grund müssen wir [Selektoren](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector) Beschriftungen anwenden. 

In diesem Zusammenhang werden wir uns hier Linux patch Kube-Proxy [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) nur Ziel Linux.

Zunächst erstellen wir ein Verzeichnis zum Speichern von .yaml Manifestdateien:
```bash
mkdir -p kube/yaml && cd kube/yaml
```

Überprüfen Sie, ob die Updatestrategie von `kube-proxy` DaemonSet auf [RollingUpdate](https://kubernetes.io/docs/tasks/manage-daemon/update-daemon-set/)festgelegt ist:

```bash
kubectl get ds/kube-proxy -o go-template='{{.spec.updateStrategy.type}}{{"\n"}}' --namespace=kube-system
```

Als Nächstes die DaemonSet herunterladen [dieser Selektoren](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) patch und wenden Sie es nur Linux Buildziel:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml
kubectl patch ds/kube-proxy --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Sobald der erfolgreich, sollten Sie "Knotenselektoren" angezeigt, der `kube-proxy` und alle anderen DaemonSets auf `beta.kubernetes.io/os=linux`

```bash
kubectl get ds -n kube-system
```

![Text](media/kube-proxy-ds.png)

### <a name="collect-cluster-information"></a>Sammeln von Clusterinformationen ###
So verknüpfen Sie erfolgreich zukünftige Knoten mit dem Master, sollten Sie die folgende Informationen nachzuverfolgen:
  1. `kubeadm join` Befehl ([hier](#initialize-master))
    * Beispiel: `kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>`
  2. Cluster-Subnetz bei definiert `kubeadm init` ([hier](#initialize-master))
    * Beispiel: `10.244.0.0/16`
  3. Dienst-Subnetz bei definiert `kubeadm init` ([hier](#initialize-master))
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

  - Unter `kubectl get pods -n kube-system`, es ist für die [Kubernetes master-Komponenten](https://kubernetes.io/docs/concepts/overview/components/#master-components) in Pods `Running` Zustand.
  - Aufrufen von `kubectl cluster-info` zeigt Informationen über den Kubernetes API-Masterserver neben den Add-Ons DNS an.
  
> [!tip]
> Da Kubeadm Netzwerk nicht eingerichtet ist, DNS-Pods möglicherweise weiterhin in `ContainerCreating` oder `Pending` Zustand. Sie werden zu wechseln, `Running` Status nach dem [Auswählen einer Lösung](./network-topologies.md).

## <a name="next-steps"></a>Nächste Schritte ## 
In diesem Abschnitt behandelt wir zum Einrichten eines Kubernetes-Masters Kubeadm verwenden. Jetzt können Sie Schritt 3:

> [!div class="nextstepaction"]
> [Auswählen einer Lösung](./network-topologies.md)