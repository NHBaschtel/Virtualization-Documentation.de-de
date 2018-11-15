---
title: Netzwerktopologien
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Netzwerktopologien für Windows und Linux unterstützt.
keywords: Kubernetes, 1.12, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: bcbd7b530b58b663305ea5d8b84a75eaf971f997
ms.sourcegitcommit: 4412583b77f3bb4b2ff834c7d3f1bdabac7aafee
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/15/2018
ms.locfileid: "6948059"
---
# <a name="network-solutions"></a>Netzwerk-Lösungen #

Sobald Sie mit der [Einrichtung eines Kubernetes-master-Knotens](./creating-a-linux-master.md) haben, sind Sie bereit sind, wählen Sie eine Netzwerke-Lösung. Es gibt mehrere Möglichkeiten, die dem virtuellen [Clustersubnetz](./getting-started-kubernetes-windows.md#cluster-subnet-def) über Knoten routingfähige vornehmen. Wählen Sie eine der folgenden Optionen für Kubernetes heute unter Windows:

1. Verwenden Sie ein Drittanbieter-CNI-Plug-in z. B. [Flannel](network-topologies.md#flannel-in-host-gateway-mode) Setup Routes für Sie.
1. Konfigurieren Sie eine intelligente [Top-des-Rack (ToR) wechseln](network-topologies.md#configuring-a-tor-switch) um das Subnetz weiterzuleiten.

> [!tip]  
> Es gibt eine dritte networking Lösung unter Windows die Open vSwitch (OvS) nutzt und öffnen virtuellen Netzwerk (OVN). Dokumentieren dies nicht zum Umfang dieses Dokuments ist, aber lesen Sie [diese Anweisungen,](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) um es einzurichten.

## <a name="flannel-in-host-gateway-mode"></a>Flannel im Host-gatewaymodus

Eine der verfügbaren Optionen für Flannel Netzwerke ist *hostgateway - Modus* (Host-gw), was die Konfiguration von statischen Routen zwischen Pod-Subnetzen auf allen Knoten verbunden ist.
> [!NOTE]  
> Dies unterscheidet sich um *den überlagerungsnetzwerkmodus in Flannel, die verwendet VXLAN Kapselung und in der Entwicklung jetzt befindet* . Inhalt beobachten …

### <a name="prepare-kubernetes-master-for-flannel"></a>Vorbereiten von Kubernetes-Master für Flannel

Einige geringfügige Vorbereitung wird auf dem [Kubernetes-Master](./creating-a-linux-master.md) in unserem Cluster empfohlen. Es wird empfohlen, Überbrückte IPv4-Datenverkehr zu Iptables Ketten aktivieren, wenn Flannel verwenden. Dies kann mit dem folgenden Befehl:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Konfigurieren von Flannel & herunterladen ###
Herunterladen der neuesten Flannel Manifests:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Es gibt zwei Dinge, die Sie tun müssen, um Netzwerke über beide Windows/Linux-Host-gw aktivieren.

In der `net-conf.json` Abschnitt von Ihrer Kube-flannel.yml, überprüfen Sie noch einmal, die:
1. Die Art der Netzwerk-Back-End verwendet wird festgelegt ist `host-gw` anstelle von `vxlan`.
2. Das Clustersubnetz (z. B. "10.244.0.0/16") festgelegt ist wie gewünscht.

Nach dem Anwenden der 2 Schritten Ihrer `net-conf.json` sollte wie folgt aussehen:
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Starten Sie Flannel & überprüfen ###
Starten Sie die Flannel verwenden:

```bash
kubectl apply -f kube-flannel.yml
```

Da die Pods Flannel Linux-basierten sind, weisen Sie dann unsere Linux- [Selektoren](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) Patch `kube-flannel-ds` DaemonSet nur Linux Ziel (wir startet den Flannel "Flanneld" Host-Agent-Prozess unter Windows später für den Beitritt):

```
kubectl patch ds/kube-flannel-ds --patch "$(cat node-selector-patch.yml)" -n=kube-system
```

Nach ein paar Minuten sollte angezeigt werden alle Pods als ausgeführt, wenn das Flannel Pod-Netzwerk bereitgestellt wurde.

```bash
kubectl get pods --all-namespaces
```

![Text](media/kube-master.png)

Die Flannel DaemonSet sollte auch die Selektoren angewendet haben.

```bash
kubectl get ds -n kube-system
```

![Text](media/kube-daemonset.png)
> [!tip]  
> Verwechselt? Hier ist ein vollständiges [Beispiel Kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) für Flannel v0.9.1 mit 2 folgendermaßen vorab für Standard-Cluster-Subnetz angewendet `10.244.0.0/16`.

## <a name="configuring-a-tor-switch"></a>Konfigurieren einen ToR-switch ##
> [!NOTE]
> Wenn Sie [Flannel als Ihr Netzwerk Lösung](#flannel-in-host-gateway-mode)ausgewählt haben, können Sie diesen Abschnitt überspringen.
Konfiguration des Schalters ToR tritt außerhalb Ihrer tatsächlichen Knoten auf. Weitere Informationen hierzu finden Sie in [offiziellen Kubernetes-Dokumentation](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Nächste Schritte ## 
In diesem Abschnitt behandelt wir so wählen Sie eine Netzwerke-Lösung. Jetzt können Sie unter Schritt 4:

> [!div class="nextstepaction"]
> [Verknüpfen von Windows-Worker](./joining-windows-workers.md)