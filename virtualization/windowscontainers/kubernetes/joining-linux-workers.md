---
title: Beitreten zu Linux-Knoten
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Hinzufügen eines Linux-Knotens zu einem Kubernetes-Cluster mit v 1,14.
keywords: kubernetes, 1,14, Windows, Getting Started
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 88207939c82bfe8ffa0b088cfd61cf4ab22cb10a
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909950"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Hinzufügen von Linux-Knoten zu einem Cluster

Nachdem Sie [einen Kubernetes-Master Knoten eingerichtet](creating-a-linux-master.md) und [die gewünschte Netzwerklösung ausgewählt](network-topologies.md)haben, können Sie Linux-Knoten zu Ihrem Cluster hinzufügen. Dies erfordert vor dem Beitritt einen gewissen [Vorbereitungs Aufwand für den Linux-Knoten](joining-linux-workers.md#preparing-a-linux-node) .
> [!tip]
> Die Linux-Anweisungen sind auf **Ubuntu 16,04**zugeschnitten. Andere Linux-Distributionen, die zum Ausführen von Kubernetes zertifiziert sind, sollten auch entsprechende Befehle anbieten, die Sie ersetzen können. Sie werden auch erfolgreich mit Windows zusammenarbeiten.

## <a name="preparing-a-linux-node"></a>Vorbereiten eines Linux-Knotens

> [!NOTE]
> Wenn nicht anders angegeben, führen Sie alle Befehle in einer **Shell mit erhöhten Rechten**aus.

Holen Sie sich zuerst eine root-Shell:

```bash
sudo –s
```

Stellen Sie sicher, dass Ihr Computer auf dem neuesten Stand ist:

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Installieren von Docker

Um Container verwenden zu können, benötigen Sie eine Container-Engine, z. b. Docker. Um die neueste Version zu erhalten, können Sie [diese Anweisungen](https://docs.docker.com/install/linux/docker-ce/ubuntu/) für die Docker-Installation verwenden. Sie können überprüfen, ob docker ordnungsgemäß installiert ist, indem Sie `hello-world` Abbild ausführen:

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Installieren von kubeadm

Laden Sie `kubeadm` Binärdateien für Ihre Linux-Distribution herunter, und initialisieren Sie Ihren Cluster.

> [!Important]  
> Abhängig von Ihrer Linux-Distribution müssen Sie möglicherweise `kubernetes-xenial` unten durch den richtigen [Codenamen](https://wiki.ubuntu.com/Releases)ersetzen.

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>Austausch deaktivieren

Kubernetes unter Linux erfordert, dass der Auslagerungs Bereich ausgeschaltet wird:

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(Nur Flannel) Aktivieren des überbrückten IPv4-Datenverkehrs in iptables

Wenn Sie die Option "Flannel" als Netzwerklösung gewählt haben, empfiehlt es sich, den überbrückten IPv4-Datenverkehr an iptables Sie sollten [dies bereits für den Master ausgeführt](network-topologies.md#flannel-in-host-gateway-mode) haben und ihn nun für den Linux-Knoten wiederholen, der beitreten möchte. Dies kann mithilfe des folgenden Befehls erfolgen:

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Kubernetes Zertifikat kopieren

Führen Sie die folgenden drei Schritte **als regulärer Benutzer (Benutzer ohne Stamm)** aus.

1. Erstellen Sie Kubernetes für das Linux-Verzeichnis:

```bash
mkdir -p $HOME/.kube
```

2. Kopieren Sie die Kubernetes-Zertifikatsdatei (`$HOME/.kube/config`) [vom Master](./creating-a-linux-master.md#collect-cluster-information) , und speichern Sie Sie als `$HOME/.kube/config` auf dem Worker.

> [!tip]
> Sie können SCP-basierte Tools wie [WinSCP](https://winscp.net/eng/download.php) verwenden, um die Konfigurationsdatei zwischen Knoten zu übertragen.

3. Legen Sie den Dateibesitz der kopierten Konfigurationsdatei wie folgt fest:

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>Beitreten zum Knoten

Zum Schluss führen Sie den `kubeadm join` Befehl aus, den [Sie zuvor notiert](./creating-a-linux-master.md#initialize-master) haben, um den Cluster zu **erhalten:**

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

Wenn erfolgreich, sollte eine ähnliche Ausgabe angezeigt werden:

![Text](./media/node-join.png)

## <a name="next-steps"></a>Nächste Schritte

In diesem Abschnitt wird beschrieben, wie Sie Linux-Worker mit unserem Kubernetes-Cluster verknüpfen. Nun sind Sie bereit für Schritt 6:
> [!div class="nextstepaction"]
> [Bereitstellen von Kubernetes Ressourcen](./deploying-resources.md)