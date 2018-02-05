---
title: "Problembehandlung für Kubernetes"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: troubleshooting
ms.prod: containers
description: "Lösungen für allgemeine Probleme beim Bereitstellen von Kubernetes und beim Beitritt zu Windows-Knoten."
keywords: Kubernetes, 1.9, Linux, Kompilieren
ms.openlocfilehash: 4fb7ac312b08c63564beb0f40889ff6a050c7166
ms.sourcegitcommit: b0e21468f880a902df63ea6bc589dfcff1530d6e
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/17/2018
---
# <a name="troubleshooting-kubernetes"></a>Problembehandlung für Kubernetes #
Diese Seite führt Sie durch mehrere Probleme beim Setup, Networking oder der Bereitstellung von Kubernetes.

> [!tip]
> Schlagen Sie ein FAQ-Element vor, indem Sie eine Veröffentlichungsanforderung an [unser Dokumentationsrepository](https://github.com/MicrosoftDocs/Virtualization-Documentation/) übermitteln.


## <a name="common-deployment-errors"></a>Allgemeine Bereitstellungsfehler ##
Das Debuggen des Kubernetes Master lässt sich in drei Kategorien einteilen (je nach Wahrscheinlichkeit):

  - Es treten Probleme mit den Kubernetes-Systemcontainern auf.
  - Es treten Probleme mit der Funktionsweise der `kubelet` auf.
  - Es treten Probleme mit dem System auf.


Führen Sie `kubectl get pods -n kube-system` durch, um die durch Kubernetes erstellten Pods anzuzeigen. Dies bietet eventuell Einblicke darin, welche abstürzen oder nicht ordnungsgemäß gestartet werden. Führen Sie dann `docker ps -a` aus, um alle unformatierten Container anzuzeigen, die diese Pods unterstützen. Führen Sie abschließend `docker logs [ID]` auf dem(n) Container(n) aus, die vermutlich das Problem verursachen, um die unformatierte Ausgabe der Prozesse anzuzeigen.


### <a name="permission-denied-errors"></a>_"Zugriff verweigert"_-Fehler ###
Stellen Sie sicher, dass Skripts über ausführbare Berechtigungen verfügen:

```bash
chmod +x [script name]
```

Zudem hinaus müssen bestimmte Skripts mit Super-User-Berechtigungen ausgeführt werden (z.B. `kubelet`) und über das Präfix `sudo` verfügen.


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>Verbindung mit dem API-Server unter `https://[address]:[port]` ist nicht möglich. ###
In den meisten Fällen deutet dieser Fehler auf Probleme mit dem Sicherheitszertifikat hin. Stellen Sie sicher, dass Sie die Konfigurationsdatei ordnungsgemäß generiert haben, dass die darin enthaltenen IP-Adressen denen Ihres Host entsprechen und dass Sie diese in das Verzeichnis kopiert haben, das vom API-Server bereitgestellt wird.

Wenn Sie [unseren Anweisungen](./creating-a-linux-master) folgen, befindet sich dies in `~/kube/kubelet/`. Andernfalls finden Sie die Bereitstellungspunkte in der Manifestdatei des API-Servers.


## <a name="common-networking-errors"></a>Allgemeine Netzwerkfehler ##
Möglicherweise gibt es weitere vorhandene Einschränkungen auf Ihrem Netzwerk oder auf Hosts, die bestimmte Kommunikationsarten zwischen Knoten verhindern. Stellen Sie Folgendes sicher:

  - Datenverkehr, der so aussieht, as ob er von Pods stammt, ist zulässig
  - HTTP-Datenverkehr ist zulässig, wenn Sie Webdienste bereitstellen
  - ICMP-Pakete werden nicht verworfen


<!-- ### My Linux node cannot ping my Windows pods ### -->

## <a name="common-windows-errors"></a>Allgemeine Fehler unter Windows ##

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>Pods stoppen nach einiger Zeit die erfolgreiche Auflösung von DNS-Abfragen. ###
Dies ist ein bekanntes Problem im Netzwerkstapel, das einige Setups betrifft. Es wird von der Windows-Wartung priorisiert.


### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>Meine Kubernetes-Pods bleiben bei „ContainerCreating” hängen. ###
Dieses Problem kann viele Ursachen haben. Eine der häufigsten ist ein falsch konfiguriertes Pause-Image. Dies ist ein High-Level-Symptom des nächsten Problems.


### <a name="when-deploying-docker-containers-keep-restarting"></a>Bei der Bereitstellung werden Docker-Container immer neu gestartet. ###
Überprüfen Sie, ob das Pause-Image mit Ihrer Betriebssystemversion kompatibel ist. In den [Anweisungen](./getting-started-kubernetes-windows.md) wird davon ausgegangen, dass Betriebssystem und Container in der Version 1709 vorliegen. Wenn Sie eine neuere Version von Windows besitzen, z.B. eine Insider-Version, müssen Sie die Images entsprechend anpassen. Weitere Informationen finden Sie im Microsoft [Docker-Repository](https://hub.docker.com/u/microsoft/) für Images. Unabhängig davon – sowohl Pause-Image-Dockerfile als auch der Beispieldienst erwarten, dass das Image als `microsoft/windowsservercore:latest` markiert ist.


### <a name="my-windows-pods-cannot-access-the-linux-master-or-vice-versa"></a>Meine Windows-Pods können nicht auf den Linux-Master zugreifen oder umgekehrt. ###
Wenn Sie einen Hyper-V-Computer verwenden, stellen Sie sicher, dass MAC-Spoofing auf dem Netzwerkadapter aktiviert ist.


### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Mein Windows-Knoten kann nicht mithilfe der Dienst-IP auf meine Dienste zugreifen. ###
Dies ist eine bekannte Einschränkung für den aktuellen Netzwerkstapel unter Windows. Nur Pods können auf die Dienst-IP verweisen.


### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>Beim Start von Kubelet wird kein Netzwerkadapter gefunden. ###
Der Windows-Netzwerkstack benötigt einen virtuellen Adapter, damit das Kubernetes-Netzwerk funktioniert. Wenn die folgenden Befehle keine Ergebnisse (in einer Admin-Shell) zurückgeben, ist die Erstellung eines virtuellen Netzwerks &mdash; eine notwendige Voraussetzung, damit Kubelet funktioniert &mdash; fehlgeschlagen.

```powershell
Get-HnsNetwork | ? Name -Like "l2bridge"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

Überprüfen Sie anhand der Ausgabe des Skripts `start-kubelet.ps1`, ob während der Erstellung des virtuellen Netzwerks Fehler auftreten.

