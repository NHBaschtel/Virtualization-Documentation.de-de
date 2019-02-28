---
title: Netzwerktopologien
author: daschott
ms.author: daschott
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Netzwerktopologien für Windows und Linux unterstützt.
keywords: Kubernetes, 1,13, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 9f96fcc80c533b74ab46d93beecc7ca8629ce395
ms.sourcegitcommit: 41318edba7459a9f9eeb182bf8519aac0996a7f1
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 02/28/2019
ms.locfileid: "9120448"
---
# <a name="network-solutions"></a>Network Solutions #

Wenn Sie mit der [Einrichtung eines Kubernetes-master-Knotens](./creating-a-linux-master.md) haben sind Sie bereit sind, wählen Sie eine Netzwerke-Lösung. Es gibt mehrere Möglichkeiten, zu dem virtuellen [Clustersubnetz](./getting-started-kubernetes-windows.md#cluster-subnet-def) auf die Knoten routingfähige verdienen. Wählen Sie eine der folgenden Optionen für Kubernetes heute unter Windows:

1. Verwenden Sie ein CNI-Plug-in z. B. [Flannel](#flannel-in-vxlan-mode) , um ein überlagerungsnetzwerk für Sie einzurichten.
2. Verwenden Sie eine CNI-Plug-in z. B. [Flannel](#flannel-in-host-gateway-mode) Programm Routen für Sie.
3. Konfigurieren Sie einen intelligenten [Top-des-Rack (ToR) wechseln](#configuring-a-tor-switch) um das Subnetz weiterzuleiten.

> [!tip]  
> Es gibt eine vierte Netzwerke Lösung unter Windows die öffnen-vSwitch (OvS) nutzt und öffnen virtuellen Netzwerk (OVN). Dokumentieren dies nicht zum Umfang dieses Dokuments ist, aber lesen Sie [diese Anweisungen,](https://kubernetes.io/docs/getting-started-guides/windows/#for-3-open-vswitch-ovs-open-virtual-network-ovn-with-overlay) um es einzurichten.

## <a name="flannel-in-vxlan-mode"></a>Flannel im Vxlan-Modus

Flannel im Vxlan Modus kann zum Einrichten eines konfigurierbaren virtuellen Überlagerung-Netzwerks verwendet VXLAN zum Weiterleiten von Paketen zwischen Knoten tunneling verwendet werden.

### <a name="prepare-kubernetes-master-for-flannel"></a>Vorbereiten von Kubernetes-Master für Flannel
Einige kleinere Vorbereitung wird auf dem [Kubernetes-Master](./creating-a-linux-master.md) in unserem Cluster empfohlen. Es wird empfohlen, um Überbrückte IPv4-Datenverkehr zu Iptables Ketten zu aktivieren, bei Verwendung von Flannel. Dies kann mit dem folgenden Befehl:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

###  <a name="download--configure-flannel"></a>Herunterladen & Flannel konfigurieren ###
Laden Sie das neueste Flannel Manifest herunter:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Es gibt zwei Bereiche, die Sie ändern sollten, um vom Netzwerk Vxlan-Back-End zu aktivieren:

1. In der `net-conf.json` Teil Ihrer `kube-flannel.yml`, überprüfen Sie noch einmal:
 * Das Clustersubnetz (z. B. "10.244.0.0/16") festgelegt ist wie gewünscht.
 * VNI 4096 ist im Back-End festgelegt.
 * Port 4789 ist im Back-End festgelegt.
2. In der `cni-conf.json` Teil Ihrer `kube-flannel.yml`, ändern Sie den Namen des Netzwerks, `"vxlan0"`.

Nach dem Anwenden der oben aufgeführten Schritte, Ihre `net-conf.json` sollte wie folgt aussehen:
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
> Die VNI muss auf 4096 und -Port 4789 für Flannel unter Linux festgelegt werden, für die Interoperabilität mit Flannel unter Windows. Unterstützung für andere VNIs ist in Kürze verfügbar. Eine Erklärung dieser Felder finden Sie unter [VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) .

Ihre `cni-conf.json` sollte wie folgt aussehen:
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
> Weitere Informationen zu den oben genannten Optionen finden Sie in der offiziellen CNI [Flannel](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#network-configuration-reference) [Portmap](https://github.com/containernetworking/plugins/tree/master/plugins/meta/portmap#port-mapping-plugin)und [Bridge](https://github.com/containernetworking/plugins/tree/master/plugins/main/bridge#network-configuration-reference) -Plug-in-Dokumentation für Linux.

### <a name="launch-flannel--validate"></a>Starten Sie Flannel & überprüfen ###
Starten Sie die Flannel verwenden:

```bash
kubectl apply -f kube-flannel.yml
```

Da die Pods Flannel Linux-basierten sind, weisen Sie dann den Linux- [Selektoren](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/l2bridge/manifests/node-selector-patch.yml) Patch `kube-flannel-ds` DaemonSet nur Linux Ziel (wir startet den Flannel "Flanneld" Host-Agent-Prozess unter Windows später für den Beitritt):

```
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Wenn keine Knoten X86-64-basierte nicht, ersetzen Sie `-amd64` oben mit der Prozessorarchitektur.

Nach ein paar Minuten sollte angezeigt werden alle Pods als ausgeführt, wenn das Flannel Pod-Netzwerk bereitgestellt wurde.

```bash
kubectl get pods --all-namespaces
```

![Text](media/kube-master.png)

Die Flannel DaemonSet sollte auch die Selektoren haben `beta.kubernetes.io/os=linux` angewendet.

```bash
kubectl get ds -n kube-system
```

![Text](media/kube-daemonset.png)

> [!tip]  
> Für die verbleibenden Flannel - ds-* DaemonSets, diese entweder ignoriert oder gelöscht, wenn sie geplant werden wird nicht, wenn keine Übereinstimmung, Prozessorarchitektur Knoten vorhanden sind.

> [!tip]  
> Verwirren. Nachfolgend finden Sie ein vollständiges [Beispiel Kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/overlay/manifests/kube-flannel-example.yml) für Flannel v0.11.0 mit diesen Schritten vorab für Standard-Cluster-Subnetz angewendet `10.244.0.0/16`.

Nachdem erfolgreich war, fahren Sie mit den [nächsten Schritten](#next-steps).

## <a name="flannel-in-host-gateway-mode"></a>Flannel in Host-gatewaymodus

Zusammen mit [Flannel Vxlan](#flannel-in-vxlan-mode)ist eine weitere Möglichkeit für Flannel Netzwerke *hostgateway - Modus* (Host-gw), was die Programmierung von statischen Routen auf jedem Knoten mit anderen Knoten Pod-Subnetzen mit den Zielknoten-Adresse des Hosts als nächsten Hop verbunden ist.

### <a name="prepare-kubernetes-master-for-flannel"></a>Vorbereiten von Kubernetes-Master für Flannel

Einige kleinere Vorbereitung wird auf dem [Kubernetes-Master](./creating-a-linux-master.md) in unserem Cluster empfohlen. Es wird empfohlen, um Überbrückte IPv4-Datenverkehr zu Iptables Ketten zu aktivieren, bei Verwendung von Flannel. Dies kann mit dem folgenden Befehl:

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```


###  <a name="download--configure-flannel"></a>Herunterladen & Flannel konfigurieren ###
Laden Sie das neueste Flannel Manifest herunter:

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Es gibt eine Datei, die Sie zum Aktivieren von Netzwerk über beide Windows/Linux-Host-gw ändern müssen.

In der `net-conf.json` Abschnitt von Ihrer Kube-flannel.yml, überprüfen Sie noch einmal, die:
1. Die Art der Netzwerk-Back-End verwendet wird festgelegt ist `host-gw` anstelle von `vxlan`.
2. Das Clustersubnetz (z. B. "10.244.0.0/16") festgelegt ist wie gewünscht.

Nach dem Anwenden der 2 Schritten Ihre `net-conf.json` sollte wie folgt aussehen:
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
kubectl patch ds/kube-flannel-ds-amd64 --patch "$(cat node-selector-patch.yml)" -n=kube-system
```
> [!tip]  
> Wenn keine Knoten X86-64-basierte nicht, ersetzen Sie `-amd64` oben mit der gewünschten Prozessorarchitektur.

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
> Für die verbleibenden Flannel - ds-* DaemonSets, diese entweder ignoriert oder gelöscht, wenn sie geplant werden wird nicht, wenn keine Übereinstimmung, Prozessorarchitektur Knoten vorhanden sind.

> [!tip]  
> Verwirren. Nachfolgend finden Sie ein vollständiges [Beispiel Kube-flannel.yml](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/manifests/kube-flannel-example.yml) für Flannel v0.11.0 mit 2 folgendermaßen vorab für Standard-Cluster-Subnetz angewendet `10.244.0.0/16`.

Nachdem erfolgreich war, fahren Sie mit den [nächsten Schritten](#next-steps).

## <a name="configuring-a-tor-switch"></a>Konfigurieren einen ToR-switch ##
> [!NOTE]
> Sie können diesen Abschnitt überspringen, wenn Sie [Flannel wie die Projektmappe Netzwerk](#flannel-in-host-gateway-mode)ausgewählt haben.
Konfiguration des Schalters ToR tritt außerhalb Ihrer tatsächlichen Knoten. Weitere Informationen hierzu finden Sie in [offiziellen Kubernetes-Dokumentation](https://kubernetes.io/docs/getting-started-guides/windows/#upstream-l3-routing-topology).


## <a name="next-steps"></a>Nächste Schritte ## 
In diesem Abschnitt behandelt wir zum auswählen und konfigurieren eine Netzwerke-Lösung. Jetzt können Sie unter Schritt 4:

> [!div class="nextstepaction"]
> [Verknüpfen von Windows-Worker](./joining-windows-workers.md)