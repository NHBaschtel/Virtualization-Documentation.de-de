---
title: "Problembehandlung für Kubernetes"
author: gkudra-msft
ms.author: gekudray
ms.date: 11/16/2017
ms.topic: troubleshooting
ms.prod: containers
description: "Lösungen für allgemeine Probleme beim Bereitstellen von Kubernetes und beim Beitritt zu Windows-Knoten."
keywords: Kubernetes, 1.9, Linux, Kompilieren
ms.openlocfilehash: 73b44ffd12fba58ac4ef38352c012061a6817945
ms.sourcegitcommit: ad5f6344230c7c4977adf3769fb7b01a5eca7bb9
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 12/05/2017
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

Darüber hinaus müssen bestimmte Skripts mit Administratorberechtigungen ausgeführt werden (z.B. `kubelet`) und über das Präfix `sudo` verfügen.


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


### <a name="my-windows-pods-cannot-access-the-linux-master-or-vice-versa"></a>Meine Windows-Pods können nicht auf den Linux-Master oder umgekehrt zugreifen. ###
Wenn Sie einen Hyper-V-Computer verwenden, stellen Sie sicher, dass MAC-Spoofing auf dem Netzwerkadapter aktiviert ist.


### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Mein Windows-Knoten kann nicht mithilfe der Dienst-IP auf meine Dienste zugreifen. ###
Dies ist eine bekannte Einschränkung für den aktuellen Netzwerkstapel unter Windows.
