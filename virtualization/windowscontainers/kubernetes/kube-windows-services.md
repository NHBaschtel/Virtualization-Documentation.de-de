---
title: Ausführen von Kubernetes als Windows-Dienst
author: daschott
ms.author: daschott
ms.date: 02/12/2019
ms.topic: get-started-article
ms.prod: containers
description: Informationen zum Ausführen von Kubernetes-Komponenten als Windows-Dienste.
keywords: Kubernetes, 1.13, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5c18
ms.openlocfilehash: 6c68edda6e2017640b0a490c3c30f063c81698b3
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/26/2019
ms.locfileid: "9578641"
---
# <a name="kubernetes-components-as-windows-services"></a>Kubernetes-Komponenten als Windows-Dienste 

Einige Benutzer möglicherweise so konfigurieren Sie Prozesse wie flanneld.exe kubelet.exe, Kube-proxy.exe oder als Windows-Dienste ausführen möchten. Dies sorgt dafür, dass zusätzliche Fehlertoleranz Vorteile wie z. B. die Prozesse, die automatisch nach einer unerwarteten Absturz Prozess oder Knoten neu gestartet wird.


## <a name="prerequisites"></a>Voraussetzungen
1. [Nssm.exe](https://nssm.cc/download) in heruntergeladen haben die `c:\k` Directory
2. Sie haben den Knoten zum Cluster hinzugefügt, und führen Sie das Skript [install.ps1](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/install.ps1) oder [start.ps1](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1) auf Ihrem Knoten zuvor

## <a name="registering-windows-services"></a>Registrieren von Windows-Diensten
Sie können [ein Beispielskript](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1) welche verwendet nssm.exe, die registriert wird ausführen `kubelet`, `kube-proxy`, und `flanneld.exe` als Windows-Dienste im Hintergrund ausgeführt:

```
C:\k\register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>
```

# [<a name="managementip"></a>ManagementIP](#tab/ManagementIP)
Die IP-Adresse zugewiesen, mit dem Windows-Knoten. Sie können `ipconfig` , diesen zu finden.

|  |  | 
|---------|---------|
|Parameter     | `-ManagementIP`        |
|Standardwert    | entfällt        |


# [<a name="networkmode"></a>NetworkMode](#tab/NetworkMode)
Den Netzwerkmodus `l2bridge` (Flannel Host-gw) oder `overlay` (Flannel Vxlan) als eine [Netzwerk-Lösung](./network-topologies.md)ausgewählt.

> [!Important] 
> `overlay` Netzwerkmodus (Flannel Vxlan) erfordert Kubernetes v1.14-Binärdateien oder höher.

|  |  | 
|---------|---------|
|Parameter     | `-NetworkMode`        |
|Standardwert    | `l2bridge`        |


# [<a name="clustercidr"></a>ClusterCIDR](#tab/ClusterCIDR)
Die [Cluster-Subnetz-Bereich](./getting-started-kubernetes-windows.md#cluster-subnet-def).

|  |  | 
|---------|---------|
|Parameter     | `-ClusterCIDR`        |
|Standardwert    | `10.244.0.0/16`        |


# [<a name="kubednsserviceip"></a>KubeDnsServiceIP](#tab/KubeDnsServiceIP)
[Kubernetes DNS-Dienst-IP](./getting-started-kubernetes-windows.md#kube-dns-def).

|  |  | 
|---------|---------|
|Parameter     | `-KubeDnsServiceIP`        |
|Standardwert    | `10.96.0.10`        |


# [<a name="logdir"></a>LogDir](#tab/LogDir)
Das Verzeichnis, in denen Kubelet und Kube-Proxy-Protokolle in ihren jeweiligen Ausgabedateien umgeleitet wurden.

|  |  | 
|---------|---------|
|Parameter     | `-LogDir`        |
|Standardwert    | `C:\k`        |

---


> [!TIP] 
> Etwas schief gehen, sollten, finden Sie in den [Abschnitt Problembehandlung](./common-problems.md#i-have-problems-running-kubernetes-processes-as-windows-services)

## <a name="manual-approach"></a>Die manuelle
Die [oben referenzierten Skript](#registering-windows-services) nicht Arbeit für Sie in diesem Abschnitt sollte enthält einige *Beispiele für Befehle* kann verwendet werden, um diese Dienste manuell schrittweise registrieren.

> [!TIP] 
> Finden Sie weitere Informationen zum Konfigurieren von [Kubelet und Kube-Proxy können jetzt als Windows-Dienste ausführen](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services) `kubelet` und `kube-proxy` als native Windows-Dienste über ausführen `sc`.

### <a name="register-flanneldexe"></a>Registrieren Sie flanneld.exe
```
nssm install flanneld C:\flannel\flanneld.exe
nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
nssm set flanneld AppDirectory C:\flannel
nssm start flanneld
```

### <a name="register-kubeletexe"></a>Registrieren Sie kubelet.exe
```
nssm install kubelet C:\k\kubelet.exe
nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=kubeletwin/pause --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
nssm set kubelet AppDirectory C:\k
nssm start kubelet
```

### <a name="register-kube-proxyexe-l2bridge--host-gw"></a>Registrieren Sie Kube-proxy.exe (l2bridge / Host-gw)
```
nssm install kube-proxy C:\k\kube-proxy.exe
nssm set kube-proxy AppDirectory c:\k
nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```

### <a name="register-kube-proxyexe-overlay--vxlan"></a>Registrieren Sie Kube-proxy.exe (Overlay / Vxlan)
```
PS C:\k> nssm install kube-proxy C:\k\kube-proxy.exe
PS C:\k> nssm set kube-proxy AppDirectory c:\k
PS C:\k> nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
nssm set kube-proxy DependOnService kubelet
nssm start kube-proxy
```