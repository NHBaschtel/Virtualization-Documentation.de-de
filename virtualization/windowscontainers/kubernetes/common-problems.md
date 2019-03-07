---
title: Problembehandlung für Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Lösungen für allgemeine Probleme beim Bereitstellen von Kubernetes und beim Beitritt zu Windows-Knoten.
keywords: Kubernetes, 1.12, Linux, kompilieren
ms.openlocfilehash: 30bb0c064c96ff4bd0b6e1c078221b2d9170d4e7
ms.sourcegitcommit: 817a629f762a4a5d4bcff58302f2bc2408bf8be1
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/07/2019
ms.locfileid: "9149920"
---
# <a name="troubleshooting-kubernetes"></a>Problembehandlung für Kubernetes #
Diese Seite führt Sie durch mehrere Probleme beim Setup, Networking oder der Bereitstellung von Kubernetes.

> [!tip]
> Schlagen Sie ein FAQ-Element vor, indem Sie eine Veröffentlichungsanforderung an [unser Dokumentationsrepository](https://github.com/MicrosoftDocs/Virtualization-Documentation/) übermitteln.

Diese Seite wird in den folgenden Kategorien unterteilt:
1. [Allgemeine Fragen](#general-questions)
2. [Allgemeine Netzwerkfehlern](#common-networking-errors)
3. [Allgemeine Fehler unter Windows](#common-windows-errors)
4. [Allgemeine Kubernetes-master-Fehler](#common-kubernetes-master-errors)

## <a name="general-questions"></a>Allgemeine Fragen ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Wie weiß ich start.ps1 unter Windows erfolgreich abgeschlossen wurde? ###
Sie sollten Kubelet, finden Sie unter Kube-Proxy, und (Wenn Sie Flannel wie die Netzwerk-Projektmappe) Flanneld Host-Agent-Prozesse auf den Knoten, mit der Ausführung der Protokolle, die im angezeigt werden, trennen PoSh Windows. Darüber hinaus sollten Ihre Windows-Knoten als "Bereit" im Kubernetes-Cluster aufgeführt werden.

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>Kann ich konfigurieren, um diese im Hintergrund anstelle von PoSh Windows ausgeführt werden? ###
Ab Version 1.11 Kubernetes, kann Kubelet & Kube-Proxy als systemeigene [Windows-Dienste](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)ausgeführt werden. Sie können auch immer alternative Service-Manager wie [nssm.exe](https://nssm.cc/) verwenden, diese Prozesse (Flanneld, Kubelet & Kube-Proxy) immer im Hintergrund für Sie ausgeführt. Sehen Sie, dass [Windows-Bereitstellungsdienste auf Kubernetes](./kube-windows-services.md) z. B. die Schritte.

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>Ich habe Probleme beim Ausführen von Kubernetes-Prozesse als Windows-Dienste ###
Anfängliche Problembehandlung, können Sie für die folgenden Flags in [nssm.exe](https://nssm.cc/) Stdout und Stderr an eine Ausgabedatei umgeleitet:
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
Weitere Informationen finden Sie unter offiziellen [Nssm Nutzung](https://nssm.cc/usage) Dokumente.

## <a name="common-networking-errors"></a>Allgemeine Netzwerkfehlern ##

### <a name="my-windows-pods-do-not-have-network-connectivity"></a>Meine Windows-Pods haben keine Verbindung zum Netzwerk ###
Wenn Sie alle virtuellen Computer verwenden, stellen Sie sicher, dass MAC-spoofing auf dem VM-Netzwerkadapter aktiviert ist. Finden Sie unter [Anti-spoofing-Schutz](./getting-started-kubernetes-windows.md#disable-anti-spoofing-protection) für Weitere Informationen.


### <a name="my-windows-pods-cannot-ping-external-resources"></a>Meine Windows-Pods können nicht externe Ressourcen ping. ###
Windows-Pods haben keine ausgehende Regeln für das Protokoll ICMP heute programmiert. TCP/UDP wird jedoch unterstützt. Bei dem Versuch, die Verbindung auf Ressourcen außerhalb des Clusters zu veranschaulichen, ersetzen Sie bitte `ping <IP>` mit entsprechenden `curl <IP>` Befehle.

Wenn Sie weiterhin Probleme gegenüberstehen, wahrscheinlich verdient Netzwerkkonfiguration in [cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) einige erfordern zusätzliche Aufmerksamkeit. Sie können diese statischen Datei immer bearbeiten, wird die Konfiguration auf alle neu erstellten Kubernetes-Ressourcen angewendet.

Woran liegt das?
Eines der Kubernetes-networking-Anforderungen ist (siehe [Kubernetes Model](https://kubernetes.io/docs/concepts/cluster-administration/networking/)) für Cluster-Kommunikation ohne NAT intern durchgeführt. Um diese Anforderung zu berücksichtigen, haben wir ein [ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) für die gesamte Kommunikation, in denen wir nicht ausgehende NAT erfolgen soll. Dies bedeutet jedoch auch, dass Sie die externe IP-Adresse ausschließen, Sie versuchen, die ExceptionList abgefragt werden, müssen. Nur dann erfolgt der Datenverkehr von Ihrem Windows-Pods stammen, die SNAT'ed ordnungsgemäß auf eine Antwort von außen zu erhalten. In dieser Hinsicht Ihre ExceptionList in `cni.conf` sollte wie folgt aussehen:
```
                "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>Mein Windows-Knoten kann nicht auf einen NodePort-Dienst zugreifen. ###
Lokaler NodePort Zugriff von den Knoten selbst schlägt fehl. Dies ist eine bekannte Einschränkung. NodePort Zugriff funktioniert von anderen Knoten oder externe Clients.

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>Nach einiger Zeit werden vNICs und HNS-Endpunkte von Containern gelöscht ###
Dieses Problem kann verursacht werden, wenn die `hostname-override` Parameter nicht an [Kube-Proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)übergeben wird. Um dies zu beheben, müssen die Benutzer den Hostnamen an Kube-Proxy wie folgt übergeben:
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>Im Modus "Flannel (Vxlan)" sind meine Pods Konnektivitätsprobleme mit der nach dem ein Knoten "" ###
Wenn ein zuvor gelöschter Knoten zum Cluster wieder aufgenommen wird ist, wird FlannelD versuchen, ein neues Pod-Subnetz auf den Knoten zuweisen. Benutzer sollten die alten Pod-Subnetz-Konfigurationsdateien in die folgenden Pfade entfernen:
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Nach dem Starten von start.ps1, ist in "Warten auf das Netzwerk zu erstellenden" Flanneld Probleme haben. ###
Es gibt zahlreiche Berichte über dieses Problem die untersucht werden sollen. Es ist wahrscheinlich ein Zeitsteuerungsproblem für, wenn die Management-IP des Netzwerks Flannel festgelegt ist. Dieses Problem zu umgehen, besteht darin, einfach start.ps1 starten oder starten es manuell wie folgt:
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

Es gibt auch eine [Veröffentlichungsanforderung](https://github.com/coreos/flannel/pull/1042) , die dieses Problem prüfenden derzeit bedient.

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>Meine Windows-Pods können nicht gestartet werden aufgrund fehlender /run/flannel/subnet.env ###
Dies bedeutet, dass Flannel nicht ordnungsgemäß gestartet wurde. Sie können entweder versuchen, flanneld.exe neu zu starten, oder Sie können kopieren Sie die Dateien manuell aus `/run/flannel/subnet.env` auf dem Kubernetes-Master auf `C:\run\flannel\subnet.env` auf dem Windows-Worker-Knoten und ändern Sie die `FLANNEL_SUBNET` Zeile an eine andere Nummer. Wenn beispielsweise Knoten Subnetz 10.244.4.1/24 gewünscht wird:
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```

### <a name="my-endpointsips-are-leaking"></a>Meine Endpunkte/IP-Adressen werden offengelegt werden. ###
Es vorhanden 2 derzeit bekannten Probleme bei der Endpunkte Speicherverluste verursachen können. 
1.  Die erste [bekanntes Problem](https://github.com/kubernetes/kubernetes/issues/68511) ist ein Problem in Kubernetes Version 1.11. Vermeiden Sie die Verwendung von Kubernetes Version 1.11.0 - 1.11.2.
2. Die zweite [bekanntes Problem](https://github.com/docker/libnetwork/issues/1950) , die Endpunkte Speicherverluste verursachen können, ist ein Concurrency-Problem im Speicher von Endpunkten. Um die Korrektur zu erhalten, verwenden Sie Docker EE 18.09 oder höher.

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>Meine-Pods können nicht gestartet werden aufgrund von "Netzwerk: Fehler beim Zuordnen für Bereich" Fehler ###
Dies bedeutet, dass der IP-Adresse Speicherplatz auf Ihrem Knoten, einrichten verwendet wird. Um alle [Endpunkte Speicherblock](#my-endpointsips-are-leaking)zu bereinigen, migrieren Sie alle Ressourcen auf betroffene Knoten & führen Sie die folgenden Befehle:
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Mein Windows-Knoten kann nicht mithilfe der Dienst-IP auf meine Dienste zugreifen. ###
Dies ist eine bekannte Einschränkung für den aktuellen Netzwerkstapel unter Windows. Windows *Pods* **sind** jedoch die Dienst-IP zugreifen.

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>Beim Start von Kubelet wird kein Netzwerkadapter gefunden. ###
Der Windows-Netzwerkstack benötigt einen virtuellen Adapter, damit das Kubernetes-Netzwerk funktioniert. Wenn die folgenden Befehle keine Ergebnisse (in einer Admin-Shell) zurückgeben, ist die Erstellung eines virtuellen Netzwerks &mdash; eine notwendige Voraussetzung, damit Kubelet funktioniert &mdash; fehlgeschlagen.

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

Oft ist es sinnvoll sein, um der [Schnittstellenname](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6) Parameter für das Skript start.ps1 in Fällen, in dem Netzwerkadapter des Hosts "Ethernet" nicht, zu ändern. Andernfalls erhalten Sie die Ausgabe der `start-kubelet.ps1` Skript, um festzustellen, ob während der Erstellung des virtuellen Netzwerks Fehler auftreten. 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Pods stoppen nach einiger Zeit die erfolgreiche Auflösung von DNS-Abfragen. ###
Es gibt eine bekannte Zwischenspeichern auf DNS-Problem im Netzwerkstapel von Windows Server, Version 1803 und darunter kann manchmal dazu führen, dass DNS-Anfragen fehlschlägt. Um dieses Problem zu umgehen, können Sie die maximale TTL-Cache-Werte mithilfe der folgenden Registrierungsschlüssel festlegen:

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>Ich kann weiterhin Probleme sehen. Was soll ich tun? ### 
Möglicherweise gibt es weitere vorhandene Einschränkungen auf Ihrem Netzwerk oder auf Hosts, die bestimmte Kommunikationsarten zwischen Knoten verhindern. Stellen Sie Folgendes sicher:
  - Sie haben die ausgewählte [Netzwerktopologie](./network-topologies.md) ordnungsgemäß konfiguriert.
  - Datenverkehr, der offensichtlich von Pods stammt, ist zulässig.
  - HTTP-Datenverkehr ist zulässig, wenn Sie Webdienste bereitstellen
  - Pakete über verschiedene Protokolle (ie ICMP im Vergleich zu TCP/UDP) werden nicht gelöscht.


## <a name="common-windows-errors"></a>Allgemeine Fehler unter Windows ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Meine Kubernetes-Pods bleiben bei „ContainerCreating” hängen. ###
Dieses Problem kann viele Ursachen haben. Eine der häufigsten ist ein falsch konfiguriertes Pause-Image. Dies ist ein High-Level-Symptom des nächsten Problems.


### <a name="when-deploying-docker-containers-keep-restarting"></a>Bei der Bereitstellung werden Docker-Container immer neu gestartet. ###
Überprüfen Sie, ob das Pause-Image mit Ihrer Betriebssystemversion kompatibel ist. Die [Anweisungen](./deploying-resources.md) wird davon ausgegangen, dass sowohl das Betriebssystem und die Container, Version 1803 sind. Wenn Sie eine neuere Version von Windows besitzen, z.B. eine Insider-Version, müssen Sie die Images entsprechend anpassen. Weitere Informationen finden Sie im Microsoft [Docker-Repository](https://hub.docker.com/u/microsoft/) für Images. Unabhängig davon – sowohl Pause-Image-Dockerfile als auch der Beispieldienst erwarten, dass das Image als `:latest` markiert ist.


## <a name="common-kubernetes-master-errors"></a>Allgemeine Kubernetes-master-Fehler ##
Das Debuggen des Kubernetes Master lässt sich in drei Kategorien einteilen (je nach Wahrscheinlichkeit):

  - Es treten Probleme mit den Kubernetes-Systemcontainern auf.
  - Es treten Probleme mit der Funktionsweise der `kubelet` auf.
  - Es treten Probleme mit dem System auf.

Führen Sie `kubectl get pods -n kube-system` durch, um die durch Kubernetes erstellten Pods anzuzeigen. Dies bietet eventuell Einblicke darin, welche abstürzen oder nicht ordnungsgemäß gestartet werden. Führen Sie dann `docker ps -a` aus, um alle unformatierten Container anzuzeigen, die diese Pods unterstützen. Führen Sie abschließend `docker logs [ID]` auf dem(n) Container(n) aus, die vermutlich das Problem verursachen, um die unformatierte Ausgabe der Prozesse anzuzeigen.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>Verbindung mit dem API-Server unter `https://[address]:[port]` ist nicht möglich. ###
In den meisten Fällen deutet dieser Fehler auf Probleme mit dem Sicherheitszertifikat hin. Stellen Sie sicher, dass Sie die Konfigurationsdatei ordnungsgemäß generiert haben, dass die darin enthaltenen IP-Adressen denen Ihres Host entsprechen und dass Sie diese in das Verzeichnis kopiert haben, das vom API-Server bereitgestellt wird.

Wenn [unsere Anweisungen](./creating-a-linux-master.md)befolgt haben, werden die gut zum erkennen, dass dies ist:   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 Lesen Sie andernfalls die Bereitstellungspunkte Manifestdatei des API-Servers.