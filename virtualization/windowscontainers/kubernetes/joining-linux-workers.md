---
title: Verknüpfen von Linux-Knoten
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Hinzufügen einer Linux-Knotens zu einem Kubernetes-Cluster mit v1.13.
keywords: Kubernetes, 1,13, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: c32cc300fd97eb53605e2f51e6a83e5889747561
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120468"
---
# <a name="joining-linux-nodes-to-a-cluster"></a>Hinzufügen von Linux-Knoten zu einem Cluster

Wenn Sie die [Einrichtung eines Kubernetes-master-Knotens](creating-a-linux-master.md) und [die gewünschte Netzwerk-Projektmappe ausgewählt](network-topologies.md)haben, sind Sie bereit, Linux-Knoten zum Cluster beizutreten. Dies erfordert einige [Vorbereitung auf den Linux-Knoten](joining-linux-workers.md#preparing-a-linux-node) , bevor Sie einbinden.
> [!tip]
> Die Linux-Anweisungen sind für **Ubuntu 16.04**zugeschnitten. Andere Linux-Distributionen, die zum Ausführen von Kubernetes-Zertifizierung bieten auch entsprechenden Befehle, die Sie ersetzen können. Sie können auch erfolgreich mit Windows ausgeführt werden.

## <a name="preparing-a-linux-node"></a>Vorbereiten eines Linux-Knotens

> [!NOTE]
> Sofern nicht explizit anders angegeben, führen Sie alle Befehle in einer **mit erhöhten Rechten, Root-Benutzer-Shell**.

Rufen Sie zunächst in eine Stamm-Shell:

```bash
sudo –s
```

Stellen Sie sicher, dass Ihre Computer auf dem neuesten Stand ist:

```bash
apt-get update && apt-get upgrade
```

## <a name="install-docker"></a>Installieren von Docker

Um Container verwendet werden können, benötigen Sie eine Container-Engine, z. B. Docker. Um die neueste Version zu erhalten, können Sie [diese Anweisungen](https://docs.docker.com/install/linux/docker-ce/ubuntu/) für die Installation von Docker verwenden. Sie können überprüfen, ob diese Docker wird durch Ausführen ordnungsgemäß installiert `hello-world` Bild:

```bash
docker run hello-world
```

## <a name="install-kubeadm"></a>Installieren von kubeadm

Herunterladen `kubeadm` Binärdateien für Ihre Linux-Distributionen und Initialisieren des Clusters.

> [!Important]  
> Je nach Linux-Distribution müssen Sie möglicherweise ersetzen `kubernetes-xenial` unten mit dem richtigen [Codename](https://wiki.ubuntu.com/Releases).

``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update && apt-get install -y kubelet kubeadm kubectl 
```

## <a name="disable-swap"></a>Deaktivieren der SwapChain

Kubernetes unter Linux erfordert Swap-Bereich deaktiviert ist:

``` bash
nano /etc/fstab  # (remove a line referencing 'swap.img' , if it exists)
swapoff -a
```

## <a name="flannel-only-enable-bridged-ipv4-traffic-to-iptables"></a>(Nur Flannel) Überbrückte IPv4-Datenverkehr zu Iptables aktivieren

Wenn Sie Flannel wie die Netzwerk-Projektmappe ausgewählt haben, die empfiehlt es sich um aktivieren, Überbrückte IPv4-Datenverkehr zu Iptables Ketten. Sie sollten [dies für den Master bereits getan](network-topologies.md#flannel-in-host-gateway-mode) haben und nun müssen sie für den Linux-Knoten, die zum Beitreten zu wiederholen. Es ist möglich, mit dem folgenden Befehl:

``` bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

## <a name="copy-kubernetes-certificate"></a>Kopieren Sie Kubernetes-Zertifikat

**Als reguläre, (nicht-Root) Benutzer**, die folgenden 3 Schritte.

1. Erstellen Sie Kubernetes für Linux-Verzeichnis:

```bash
mkdir -p $HOME/.kube
```

2. Kopieren Sie die Zertifikatdatei Kubernetes (`$HOME/.kube/config`) [vom Master](./creating-a-linux-master.md#collect-cluster-information) und speichern Sie als `$HOME/.kube/config` auf der Worker.

> [!tip]
> Scp-basierten Tools wie z. B. [WinSCP](https://winscp.net/eng/download.php) können Sie die Konfigurationsdatei zwischen Knoten übertragen.

3. Legen Sie Dateibesitz der kopierten Konfigurationsdatei wie folgt:

``` bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## <a name="joining-node"></a>Wenn Knoten

Um dem Cluster beizutreten, führen Sie abschließend die `kubeadm join` [wir bereits erwähnt nach unten](./creating-a-linux-master.md#initialize-master) **als Root**-Befehl:

```bash
kubeadm join <Master_IP>:6443 --token <some_token> --discovery-token-ca-cert-hash <some_hash>
```

Falls erfolgreich, sollten Sie ähnliche Ausgabe folgt finden Sie unter:

![Text](./media/node-join.png)

## <a name="next-steps"></a>Nächste Schritte

In diesem Abschnitt behandelt wir Linux-Worker zu unserer Kubernetes-Cluster beitreten. Jetzt sind Sie bereit für Schritt 6:
> [!div class="nextstepaction"]
> [Bereitstellen von Kubernetes-Ressourcen](./deploying-resources.md)