---
title: Problembehandlung für Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Lösungen für allgemeine Probleme beim Bereitstellen von Kubernetes und beim Beitritt zu Windows-Knoten.
keywords: kubernetes, 1,14, Linux, kompilieren
ms.openlocfilehash: 8bebc83e03fe919f6af3968b0e0463ab3c6bb987
ms.sourcegitcommit: 6b925368d122ba600d7d4c73bd240cdcb915cccd
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/22/2019
ms.locfileid: "10305724"
---
# <a name="troubleshooting-kubernetes"></a>Problembehandlung für Kubernetes #
Diese Seite führt Sie durch mehrere Probleme beim Setup, Networking oder der Bereitstellung von Kubernetes.

> [!tip]
> Schlagen Sie ein FAQ-Element vor, indem Sie eine Veröffentlichungsanforderung an [unser Dokumentationsrepository](https://github.com/MicrosoftDocs/Virtualization-Documentation/) übermitteln.

Diese Seite ist in die folgenden Kategorien unterteilt:
1. [Allgemeine Fragen](#general-questions)
2. [Häufige Netzwerkfehler](#common-networking-errors)
3. [Häufige Windows-Fehler](#common-windows-errors)
4. [Häufige Kubernetes-Master Fehler](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Allgemeine Fragen ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Wie kann ich feststellen, dass Start. ps1 unter Windows erfolgreich abgeschlossen wurde? ###
Sie sollten kubelet, Kuben-Proxy und (wenn Sie "Flanell" als ihre Netzwerklösung ausgewählt haben) Flanell-Host-Agent-Prozesse sehen, die auf Ihrem Knoten ausgeführt werden, wobei die ausgeführten Protokolle in separaten PoSh-Fenstern angezeigt werden. Darüber hinaus sollte Ihr Windows-Knoten in Ihrem Kubernetes-Cluster als "bereit" aufgeführt werden.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>Kann ich so konfigurieren, dass all dies im Hintergrund statt in PoSh Windows ausgeführt wird? ###
Beginnend mit Kubernetes Version 1,11 kann kubelet #a0 Kuben-Proxy als systemeigene Windows- [Dienste](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)ausgeführt werden. Sie können auch immer Alternative Service Manager wie [nssm. exe](https://nssm.cc/) verwenden, um diese Prozesse (Flanell, kubelet #a0 Kuben-Proxy) im Hintergrund für Sie immer auszuführen. Siehe [Windows-Dienste auf Kubernetes](./kube-windows-services.md) , beispielsweise Schritte.

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>Ich habe Probleme beim Ausführen von Kubernetes-Prozessen als Windows-Dienste ###
Zur ersten Problembehandlung können Sie die folgenden Flags in [nssm. exe](https://nssm.cc/) verwenden, um stdout und stderr in eine Ausgabedatei umzuleiten:
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
Weitere Informationen finden Sie unter offizielle [nssm-Verwendungs](https://nssm.cc/usage) Dokumente.

## <a name="common-networking-errors"></a>Häufige Netzwerkfehler ##

### <a name="load-balancers-are-plumbed-inconsistently-across-the-cluster-nodes"></a>Lastenausgleichsgeräte werden inkonsistent über die Clusterknoten verteilt ###
In der Konfiguration des (Standard-) Kuben-Proxys können Cluster, die 100 + Load-Balancer enthalten, die verfügbaren ephemeren (dynamischen) Ports aufgrund der großen Anzahl von Ports, die auf jedem Knoten für jeden (nicht-DSR-) Lastenausgleichsmodul reserviert sind, auslassen. Dies kann sich durch Fehler in einem Kuben-Proxy wie folgt manifestieren:
```
Policy creation failed: hcnCreateLoadBalancer failed in Win32: The specified port already exists.
```

Benutzer können dieses Problem identifizieren, indem Sie das [CollectLogs. ps1](https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/debug/collectlogs.ps1) -Skript `*portrange.txt` ausführen und die Dateien konsultieren. In `reservedports.txt`wird auch eine heuristische Zusammenfassung generiert.

Um dieses Problem zu beheben, können einige Schritte ausgeführt werden:
1.  Für eine dauerhafte Lösung sollte der Lade Ausgleich für den Kuben-Proxy auf den [DSR-Modus](https://techcommunity.microsoft.com/t5/Networking-Blog/Direct-Server-Return-DSR-in-a-nutshell/ba-p/693710)eingestellt werden. Leider ist der DSR-Modus nur für neuere [Windows Server Insider Build 18945](https://blogs.windows.com/windowsexperience/2019/07/30/announcing-windows-server-vnext-insider-preview-build-18945/#o1bs7T2DGPFpf7HM.97) (oder höher) vollständig implementiert.
2. Um dieses Problem zu umgehen, können Benutzer auch die standardmäßige Windows-Konfiguration von ephemeren Ports mithilfe `netsh int ipv4 dynamicportrange TCP <start_range> <end_range>`eines Befehls wie. *Warnung:* Das Überschreiben des standardmäßigen dynamischen Portbereichs kann Auswirkungen auf andere Prozesse/Dienste auf dem Host haben, die auf verfügbare TCP-Ports aus dem nicht ephemeren Bereich angewiesen sind, damit dieser Bereich sorgfältig ausgewählt werden sollte.
3. Darüber hinaus arbeiten wir an einer skalierbaren Erweiterung für Lastenausgleichsfunktionen für nicht-DSR-Modi mithilfe der intelligenten Port Pool Freigabe, die über ein kumulatives Update in Q1 2020 freigegeben werden soll.

### <a name="hostport-publishing-is-not-working"></a>HostPort Publishing funktioniert nicht ###
Es ist derzeit nicht möglich, Ports mithilfe des Kubernetes `containers.ports.hostPort` -Felds zu veröffentlichen, da dieses Feld nicht von Windows cni-Plugins berücksichtigt wird. Bitte verwenden Sie DEPORT Publishing zurzeit, um Ports auf dem Knoten zu veröffentlichen.

### <a name="i-am-seeing-errors-such-as-hnscall-failed-in-win32-the-wrong-diskette-is-in-the-drive"></a>Ich sehe Fehler wie "hnsCall Fehler in Win32: die falsche Diskette befindet sich auf dem Laufwerk." ###
Dieser Fehler kann auftreten, wenn Sie benutzerdefinierte Änderungen an HNS-Objekten vornehmen oder ein neues Windows Update installieren, das Änderungen an HNS einführt, ohne alte HNS-Objekte zu zerstören. Es gibt an, dass ein HNS-Objekt, das zuvor erstellt wurde, bevor ein Update mit der aktuell installierten HNS-Version nicht kompatibel ist.

Unter Windows Server 2019 (und darunter) können Benutzer HNS-Objekte löschen, indem Sie die HNS. Data-Datei löschen. 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

Benutzer sollten in der Lage sein, alle inkompatiblen HNS-Endpunkte oder Netzwerke direkt zu löschen:
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Benutzer unter Windows Server, Version 1903, können an den folgenden Registrierungsspeicherort wechseln und alle NICs löschen, beginnend mit dem Netzwerknamen ( `vxlan0` Beispiels `cbr0`Weise oder):
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```

### <a name="containers-on-my-flannel-host-gw-deployment-on-azure-cannot-reach-the-internet"></a>Container auf meinem Flanell-Host – GW-Bereitstellung auf Azure kann das Internet nicht erreichen ###
Beim Bereitstellen von Flanell im Host-GW-Modus auf Azure müssen Pakete den Azure Physical Host Vswitch durchlaufen. Benutzer sollten für jedes Subnetz, das einem Knoten zugewiesen ist, [benutzerdefinierte Routen](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined) des Typs "Virtual Appliance" programmieren. Dies kann über das Azure-Portal erfolgen (siehe [hier](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)ein Beispiel) oder über `az` Azure CLI. Hier ist ein Beispiel UDR mit dem Namen "MyRoute" mit AZ-Befehlen für einen Knoten mit IP-10.0.0.4 und entsprechendem Pod-Subnetz 10.244.0.0/24:
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

### <a name="my-windows-pods-cannot-ping-external-resources"></a>Meine Windows-Pods können externe Ressourcen nicht anpingen ###
Für Windows-Pods sind heute keine ausgehenden Regeln für das ICMP-Protokoll programmiert. TCP/UDP wird jedoch unterstützt. Wenn Sie versuchen, die Verbindung zu Ressourcen außerhalb des Clusters zu demonstrieren `ping <IP>` , `curl <IP>` ersetzen Sie die entsprechenden Befehle.

Wenn Sie nach wie vor Probleme haben, ist es wahrscheinlich, dass Ihre Netzwerkkonfiguration in [cni. conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) etwas mehr Aufmerksamkeit verdient. Sie können diese statische Datei immer bearbeiten, die Konfiguration wird auf alle neu erstellten Kubernetes-Ressourcen angewendet.

Woran liegt das?
Eine der Kubernetes-Netzwerkanforderungen (siehe [Kubernetes-Modell](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) besteht darin, dass die Clusterkommunikation intern ohne NAT erfolgen kann. Um diese Anforderung zu erfüllen, haben wir eine [Ausnahme](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) für die gesamte Kommunikation, in der keine ausgehende NAT erfolgen soll. Dies bedeutet aber auch, dass Sie die externe IP-Adresse, die Sie Abfragen möchten, aus der exceptionliste ausschließen müssen. Nur dann wird der von Ihren Windows-Pods stammende Datenverkehr richtig SNAT'ed, um eine Antwort von der Außenwelt zu erhalten. In diesem Zusammenhangsollte ihre ausnahmelist `cni.conf` in wie folgt aussehen:
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>Mein Windows-Knoten kann nicht auf einen DEPORT-Dienst zugreifen ###
Der Zugriff auf den lokalen DEPORT vom Knoten selbst führt zu einem Fehler. Dies ist eine bekannte Einschränkung. Der Zugriff auf DEPORT kann von anderen Knoten oder externen Clients aus funktionieren.

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>Nach einiger Zeit werden vNICs-und HNS-Endpunkte von Containern gelöscht. ###
Dieses Problem kann auftreten, wenn der `hostname-override` Parameter nicht an den [Kuben-Proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)übergeben wird. Um das Problem zu beheben, müssen die Benutzer den Hostnamen wie folgt an den Kuben-Proxy übergeben:
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>Im Flanell-Modus (vxlan) haben meine Pods Verbindungsprobleme, nachdem Sie dem Knoten erneut beigetreten sind. ###
Wenn ein zuvor gelöschter Knoten wieder mit dem Cluster verknüpft wird, versucht flanelled, dem Knoten ein neues Pod-Subnetz zuzuweisen. Benutzer sollten die alten Konfigurationsdateien des Pod-Subnetzes in den folgenden Pfaden entfernen:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Nach dem Start von Start. ps1 bleibt flanelld in "warten auf das zu erstellende Netzwerk" hängen. ###
Es gibt zahlreiche Berichte über dieses Problem, die untersucht werden. höchstwahrscheinlich handelt es sich um ein Problem mit der Zeitmessung, wenn die Management-IP des Flanell-Netzwerks festgesetzt ist. Eine Problemumgehung besteht darin, Start. ps1 einfach neu zu starten oder manuell wie folgt erneut zu starten:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

Es gibt auch eine [PR](https://github.com/coreos/flannel/pull/1042) , die dieses Problem im Rahmen der Überprüfung derzeit behebt.


### <a name="on-flannel-host-gw-my-windows-pods-do-not-have-network-connectivity"></a>Auf Flanell (Host-GW) haben meine Windows-Pods keine Netzwerkkonnektivität ###
Wenn Sie l2bridge für Netzwerke (auch als [Flanell-Host-Gateway](./network-topologies.md#flannel-in-host-gateway-mode)) verwenden möchten, sollten Sie sicherstellen, dass das Spoofing von Mac-Adressen für die Windows-Container Host-VMS (Gäste) aktiviert ist. Um dies zu erreichen, sollten Sie Folgendes als Administrator auf dem Computer ausführen, auf dem die VMs gehostet werden (Beispiel für Hyper-V):

```powershell
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> [!TIP]
> Wenn Sie ein VMware-basiertes Produkt verwenden, um Ihre Anforderungen an die Virtualisierung zu erfüllen, schauen Sie unter Aktivieren des [Promiscuous-Modus](https://kb.vmware.com/s/article/1004099) für die MAC-Spoofing-Anforderung nach.

>[!TIP]
> Wenn Sie Kubernetes auf Azure-oder IaaS-VMS von anderen Cloud-Anbietern selbst bereitstellen, können Sie stattdessen auch [Overlay-Netzwerke](./network-topologies.md#flannel-in-vxlan-mode) verwenden.

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Meine Windows-Pods können aufgrund fehlender/Run/Flannel/Subnet.env nicht gestartet werden ###
Dies zeigt an, dass Flanell nicht richtig gestartet wurde. Sie können entweder versuchen, flanelld. exe neu zu starten, oder Sie können die Dateien manuell `/run/flannel/subnet.env` aus dem Kubernetes-Master `C:\run\flannel\subnet.env` in den Windows-Worker-Knoten kopieren `FLANNEL_SUBNET` und die Zeile in das zugewiesene Subnetz ändern. Wenn beispielsweise Knoten-Subnetz 10.244.4.1/24 zugewiesen wurde:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
Es ist sicherer, flanelld. exe diese Datei für Sie zu generieren.

### <a name="pod-to-pod-connectivity-between-hosts-is-broken-on-my-kubernetes-cluster-running-on-vsphere"></a>Die Pod-zu-Pod-Konnektivität zwischen Hosts ist auf meinem Kubernetes-Cluster auf vSphere unterbrochen 
Da sowohl vSphere als auch Flanell den Port 4789 (standardmäßigen VXLAN-Port) für Overlay-Netzwerke reserviert, können Pakete am Ende abgefangen werden. Wenn vSphere für Overlay-Netzwerke verwendet wird, sollte es so konfiguriert werden, dass ein anderer Port verwendet wird, um 4789 freizugeben.  


### <a name="my-endpointsips-are-leaking"></a>Meine Endpunkte/IPS sind undicht ###
Es gibt zwei zurzeit bekannte Probleme, die zu Endpunkten führen können. 
1.  Das erste [bekannte Problem](https://github.com/kubernetes/kubernetes/issues/68511) ist ein Problem in Kubernetes Version 1,11. Vermeiden Sie es, Kubernetes Version 1.11.0-1.11.2 zu verwenden.
2. Das zweite [bekannte Problem](https://github.com/docker/libnetwork/issues/1950) , bei dem Endpunkte auslaufen können, ist ein Parallelitäts Problem beim Speichern von Endpunkten. Um das Update zu erhalten, müssen Sie docker EE 18,09 oder höher verwenden.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>Meine Pods können aufgrund der Fehler "Netzwerk: Fehler beim Zuweisen für Bereichsfehler" nicht gestartet werden ###
Dies zeigt an, dass der IP-Adressraum auf dem Knoten aufgebraucht ist. Wenn Sie alle durch [gesickerten Endpunkte](#my-endpointsips-are-leaking)bereinigen möchten, migrieren Sie alle Ressourcen auf betroffenen Knoten, #a0 führen Sie die folgenden Befehle aus:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Mein Windows-Knoten kann nicht mithilfe der Dienst-IP auf meine Dienste zugreifen. ###
Dies ist eine bekannte Einschränkung für den aktuellen Netzwerkstapel unter Windows. Windows- *Pods* **können jedoch** auf die Dienst-IP zugreifen.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>Beim Start von Kubelet wird kein Netzwerkadapter gefunden. ###
Der Windows-Netzwerkstack benötigt einen virtuellen Adapter, damit das Kubernetes-Netzwerk funktioniert. Wenn die folgenden Befehle keine Ergebnisse (in einer Admin-Shell) zurückgeben, ist die Erstellung eines virtuellen Netzwerks &mdash; eine notwendige Voraussetzung, damit Kubelet funktioniert &mdash; fehlgeschlagen.

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

Häufig lohnt es sich, den Parameter [InterfaceName](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) des Skripts Start. ps1 zu ändern, wenn der Netzwerkadapter des Hosts nicht "Ethernet" ist. Konsultieren Sie andernfalls die Ausgabe des `start-kubelet.ps1` Skripts, um festzustellen, ob während der Erstellung des virtuellen Netzwerks Fehler auftreten. 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Pods stoppen nach einiger Zeit die erfolgreiche Auflösung von DNS-Abfragen. ###
Es gibt ein bekanntes Problem mit der DNS-Zwischenspeicherung im Netzwerkstapel von Windows Server, Version 1803 und darunter, die möglicherweise dazu führen, dass DNS-Anforderungen fehlschlagen. Um dieses Problem zu umgehen, können Sie mit den folgenden Registrierungsschlüsseln die Werte für den Max-TTL-Cache auf Null setzen:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>Ich sehe immer noch Probleme. Was soll ich tun? ### 
Möglicherweise gibt es weitere vorhandene Einschränkungen auf Ihrem Netzwerk oder auf Hosts, die bestimmte Kommunikationsarten zwischen Knoten verhindern. Stellen Sie Folgendes sicher:
  - Sie haben die ausgewählte [Netzwerktopologie](./network-topologies.md) ordnungsgemäß konfiguriert.
  - Datenverkehr, der offensichtlich von Pods stammt, ist zulässig.
  - HTTP-Datenverkehr ist zulässig, wenn Sie Webdienste bereitstellen
  - Pakete aus verschiedenen Protokollen (IE ICMP vs. TCP/UDP) werden nicht gelöscht


## <a name="common-windows-errors"></a>Häufige Windows-Fehler ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Meine Kubernetes-Pods bleiben bei „ContainerCreating” hängen. ###
Dieses Problem kann viele Ursachen haben. Eine der häufigsten ist ein falsch konfiguriertes Pause-Image. Dies ist ein High-Level-Symptom des nächsten Problems.


### <a name="when-deploying-docker-containers-keep-restarting"></a>Bei der Bereitstellung werden Docker-Container immer neu gestartet. ###
Überprüfen Sie, ob das Pause-Image mit Ihrer Betriebssystemversion kompatibel ist. In den [Anweisungen](./deploying-resources.md) wird davon ausgegangen, dass das Betriebssystem und die Container Version 1803 sind. Wenn Sie eine neuere Version von Windows besitzen, z.B. eine Insider-Version, müssen Sie die Images entsprechend anpassen. Weitere Informationen finden Sie im Microsoft [Docker-Repository](https://hub.docker.com/u/microsoft/) für Images. Unabhängig davon – sowohl Pause-Image-Dockerfile als auch der Beispieldienst erwarten, dass das Image als `:latest` markiert ist.


## <a name="common-kubernetes-master-errors"></a>Häufige Kubernetes-Master Fehler ##
Das Debuggen des Kubernetes Master lässt sich in drei Kategorien einteilen (je nach Wahrscheinlichkeit):

  - Es treten Probleme mit den Kubernetes-Systemcontainern auf.
  - Es treten Probleme mit der Funktionsweise der `kubelet` auf.
  - Es treten Probleme mit dem System auf.

Führen Sie `kubectl get pods -n kube-system` durch, um die durch Kubernetes erstellten Pods anzuzeigen. Dies bietet eventuell Einblicke darin, welche abstürzen oder nicht ordnungsgemäß gestartet werden. Führen Sie dann `docker ps -a` aus, um alle unformatierten Container anzuzeigen, die diese Pods unterstützen. Führen Sie abschließend `docker logs [ID]` auf dem(n) Container(n) aus, die vermutlich das Problem verursachen, um die unformatierte Ausgabe der Prozesse anzuzeigen.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>Verbindung mit dem API-Server unter `https://[address]:[port]` ist nicht möglich. ###
In den meisten Fällen deutet dieser Fehler auf Probleme mit dem Sicherheitszertifikat hin. Stellen Sie sicher, dass Sie die Konfigurationsdatei ordnungsgemäß generiert haben, dass die darin enthaltenen IP-Adressen denen Ihres Host entsprechen und dass Sie diese in das Verzeichnis kopiert haben, das vom API-Server bereitgestellt wird.

Wenn Sie [unseren Anweisungen](./creating-a-linux-master.md)folgen, finden Sie hier die folgenden Möglichkeiten:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 Andernfalls lesen Sie die Manifestdatei des API-Servers, um die Bereitstellungspunkte zu überprüfen.
