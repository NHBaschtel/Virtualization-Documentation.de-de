---
title: Netzwerktopologien
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Unterstützte Netzwerk Topologien unter Windows und Linux.
keywords: kubernetes, 1,14, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6b0e13258b749ad3dfd5c8349200ca8a54908952
ms.sourcegitcommit: 42cb47ba4f3e22163869d094bd0c9cff415a43b0
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 08/02/2019
ms.locfileid: "9884981"
---
# <a name="network-solutions"></a>Netzwerklösungen #

Nachdem Sie [einen Kubernetes-Masterknoten eingerichtet](./creating-a-linux-master.md) haben, können Sie eine Netzwerklösung aussuchen. Es gibt mehrere Möglichkeiten, das virtuelle [Cluster-Subnetz](./getting-started-kubernetes-windows.md#cluster-subnet-def) über Knoten hinweg routingfähig zu machen. Wählen Sie heute eine der folgenden Optionen für Kubernetes unter Windows:

1. Verwenden Sie ein cni-Plug-in wie [Flanell](#flannel-in-vxlan-mode) , um ein Overlay-Netzwerk für Sie einzurichten.
2. Verwenden Sie ein cni-Plug-in wie [Flanell](#flannel-in-host-gateway-mode) , um Routen für Sie zu programmieren (verwendet den l2bridge-Netzwerkmodus).
3. Konfigurieren Sie einen intelligenten [Top-of-Rack-Schalter (Tor)](#configuring-a-tor-switch) , um das Subnetz weiterzuleiten.

> [!tip]  
> Es gibt eine vierte Netzwerklösung unter Windows, die Open Vswitch (OvS) und Open Virtual Network (OVN) nutzt. Das dokumentieren dieses Dokuments ist außerhalb des gültigen Bereichs für dieses Dokument, doch Sie können [diese Anweisungen](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) lesen, um es einzurichten.

## <a name="flannel-in-vxlan-mode"></a>Flanell im vxlan-Modus

Flanell im vxlan-Modus kann verwendet werden, um ein konfigurierbares virtuelles Overlay-Netzwerk einzurichten, das vxlan-Tunneling verwendet, um Pakete zwischen Knoten weiterzuleiten.

### <a name="prepare-kubernetes-master-for-flannel"></a>Vorbereiten des Kubernetes-Masters für Flanell
Einige kleinere Vorbereitungen sind für den [Kubernetes-Master](./creating-a-linux-master.md) in unserem Cluster empfehlenswert. Es wird empfohlen, bei Verwendung von Flanell über brückter IPv4-Datenverkehr zu iptables-Ketten zu aktivieren. Dies kann mithilfe des folgenden Befehls erfolgen:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Download #a0 Konfigurieren von Flanell ###
Laden Sie das neueste Flanell-Manifest herunter:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Es gibt zwei Abschnitte, die Sie ändern sollten, um das vxlan-Netzwerk-Back-End zu aktivieren:

1. Doppelklicken `net-conf.json` Sie im Abschnitt `kube-flannel.yml`ihrer auf:
 * Das Cluster-Subnetz (z.b. "10.244.0.0/16") wird wie gewünscht eingestellt.
 * VNI 4096 wird im Back-End eingestellt
 * Port 4789 wird im Back-End-Programm eingestellt
2. Ändern Sie `cni-conf.json` im Abschnitt Ihres `kube-flannel.yml`den Netzwerknamen in `"vxlan0"`.

Nachdem Sie die obigen Schritte angewendet haben `net-conf.json` , sollten Sie wie folgt aussehen:
```json
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan",
        "VNI" : 4096,
        "Port": 4789
      }
    }
```

> [!NOTE]  
> Der vni muss auf 4096 und Port 4789 für Flanell auf Linux eingestellt sein, um mit Flanell unter Windows zu interagieren. Support für andere VNIs wird in Kürze verfügbar sein. Eine Erläuterung dieser Felder finden Sie unter [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) .

Ihr `cni-conf.json` sollte wie folgt aussehen:
```json
cni-conf.json: |
    {
      "name": "vxlan0",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
```
> [!tip]  
> Weitere Informationen zu den oben genannten Optionen finden Sie in den offiziellen cni [Flanell](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference)-, [portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)-und [Bridge](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) -Plug-in-Dokumenten für Linux.

### <a name="launch-flannel--validate"></a>Starten von Flanell #a0 überprüfen ###
Starten Sie Flanell mit:

```bash
kubectl apply -f kube-flannel.yml
```

Nachdem die Flanell Hülsen Linux-basiert sind, wenden Sie den Linux- [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) - `kube-flannel-ds` Patch auf daemonset an, um nur auf Linux zu Zielen (wir werden den Flanell-"Flanell"-Host-Agent-Prozess auf Windows später beim Beitritt starten):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Wenn keine Knoten x86-64-basiert sind, `-amd64` ersetzen Sie oben durch ihre Prozessorarchitektur.

Nach ein paar Minuten sollten alle Pods als ausgeführt angezeigt werden, wenn das Flanell-Pod-Netzwerk bereitgestellt wurde.

```bash
kubectl get pods --all-namespaces
```

![Text](media/kube-master.png)

Das Flanell-daemonset sollte auch die NodeSelector `beta.kubernetes.io/os=linux` angewendet haben.

```bash
kubectl get ds -n kube-system
```

![Text](media/kube-daemonset.png)

> [!tip]  
> Für das restliche Flanell-DS-*-DaemonSets können diese entweder ignoriert oder gelöscht werden, da Sie nicht geplant werden, wenn keine Knoten vorhanden sind, die dieser Prozessorarchitektur entsprechen.

> [!tip]  
> Verwirrt? Im folgenden finden Sie ein vollständiges [Beispiel Kube-Flannel. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) für Flanell v 0.11.0 mit den folgenden Schritten, die `10.244.0.0/16`für das Standard-Cluster-Subnetz bereits angewendet wurden.

Wenn Sie erfolgreich sind, fahren Sie mit den [nächsten Schritten](#next-steps)fort.

## <a name="flannel-in-host-gateway-mode"></a>Flanell im Host-Gateway-Modus

Neben [Flanell-vxlan](#flannel-in-vxlan-mode)ist eine weitere Option für Flanell *-Netzwerke der Host-Gateway-Modus* (Host-GW), der die Programmierung von statischen Routen auf jedem Knoten zu den Pod-Subnetzen eines anderen Knotens unter Verwendung der Hostadresse des Zielknotens als nächster Hop beinhaltet.

### <a name="prepare-kubernetes-master-for-flannel"></a>Vorbereiten des Kubernetes-Masters für Flanell

Einige kleinere Vorbereitungen sind für den [Kubernetes-Master](./creating-a-linux-master.md) in unserem Cluster empfehlenswert. Es wird empfohlen, bei Verwendung von Flanell über brückter IPv4-Datenverkehr zu iptables-Ketten zu aktivieren. Dies kann mithilfe des folgenden Befehls erfolgen:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Download #a0 Konfigurieren von Flanell ###
Laden Sie das neueste Flanell-Manifest herunter:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Es gibt eine Datei, die Sie ändern müssen, um Host-GW-Netzwerke über Windows/Linux zu ermöglichen.

Überprüfen `net-conf.json` Sie im Abschnitt Ihres Kube-Flannel. yml Folgendes:
1. Der Typ des verwendeten Netzwerk-Back `host-gw` `vxlan`-Ends ist auf "statt" gesetzt.
2. Das Cluster-Subnetz (z.b. "10.244.0.0/16") wird wie gewünscht eingestellt.

Nachdem Sie die beiden Schritte angewendet haben `net-conf.json` , sollten Sie wie folgt aussehen:
```json
net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "host-gw"
      }
    }
```

### <a name="launch-flannel--validate"></a>Starten von Flanell #a0 überprüfen ###
Starten Sie Flanell mit:

```bash
kubectl apply -f kube-flannel.yml
```

Nachdem die Flanell Hülsen Linux-basiert sind, wenden Sie unseren Linux [NodeSelector](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) -Patch `kube-flannel-ds` auf daemonset an, um nur auf Linux zu Zielen (wir werden den Flanell-"Flanell"-Host-Agent-Prozess auf Windows später beim beitreten starten):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Wenn keine Knoten x86-64-basiert sind, `-amd64` ersetzen Sie oben durch die gewünschte Prozessorarchitektur.

Nach ein paar Minuten sollten alle Pods als ausgeführt angezeigt werden, wenn das Flanell-Pod-Netzwerk bereitgestellt wurde.

```bash
kubectl get pods --all-namespaces
```

![Text](media/kube-master.png)

Das Flanell-daemonset sollte auch die NodeSelector angewendet haben.

```bash
kubectl get ds -n kube-system
```

![Text](media/kube-daemonset.png)

> [!tip]  
> Für das restliche Flanell-DS-*-DaemonSets können diese entweder ignoriert oder gelöscht werden, da Sie nicht geplant werden, wenn keine Knoten vorhanden sind, die dieser Prozessorarchitektur entsprechen.

> [!tip]  
> Verwirrt? Im folgenden finden Sie ein vollständiges [Beispiel Kube-Flannel. yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) für Flanell v 0.11.0 mit diesen zwei Schritten, die für `10.244.0.0/16`das standardmäßige Cluster-Subnetz vordefiniert sind.

Wenn Sie erfolgreich sind, fahren Sie mit den [nächsten Schritten](#next-steps)fort.

## <a name="configuring-a-tor-switch"></a>Konfigurieren eines Tor-Schalters ##
> [!NOTE]
> Sie können diesen Abschnitt überspringen, wenn Sie [Flanell als Netzwerklösung](#flannel-in-host-gateway-mode)gewählt haben.
Die Konfiguration des Tor-Schalters erfolgt außerhalb ihrer eigentlichen Knoten. Weitere Informationen hierzu finden Sie unter [offizielle Kubernetes-Dokumente](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Nächste Schritte ## 
In diesem Abschnitt wird beschrieben, wie Sie eine Netzwerklösung aussuchen und konfigurieren. Jetzt sind Sie bereit für Schritt 4:

> [!div class="nextstepaction"]
> [Beitreten zu Windows-Mitarbeitern](./joining-windows-workers.md)