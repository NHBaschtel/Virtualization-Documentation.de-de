---
title: Linux-Knoten beitreten
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Bereitstellen von Kubernetes-Resoureces auf einem vermischten OS-Kubernetes-Cluster.
keywords: Kubernetes, 1.14, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 6ede914def6c5a94313164ad78eeecf61c4fab4a
ms.sourcegitcommit: aaf115a9de929319cc893c29ba39654a96cf07e1
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 05/10/2019
ms.locfileid: "9622935"
---
# <a name="deploying-kubernetes-resources"></a>Bereitstellen von Kubernetes-Ressourcen #
Vorausgesetzt, dass Sie einen Kubernetes-Cluster mit mindestens 1 Master und 1 Worker haben, können Sie zum Bereitstellen von Kubernetes-Ressourcen.
> [!TIP] 
> Möchten Sie wissen, welche Ressourcen Kubernetes heute unter Windows unterstützt werden? Finden Sie [offiziell unterstützte Funktionen](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features) und [Kubernetes auf Windows-Roadmap](https://trello.com/b/rjTqrwjl/windows-k8s-roadmap) für weitere Details.


## <a name="running-a-sample-service"></a>Ausführen eines beispieldiensts ##
Sie werden einen sehr einfachen [PowerShell-basierten Webdienst](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) bereitstellen, um sicherzustellen, dass der Cluster erfolgreich hinzugefügt und unser Netzwerk ordnungsgemäß konfiguriert wurde.

Bevor Sie dies tun, ist es immer empfiehlt sich, stellen Sie sicher, dass alle unsere Knoten fehlerfrei sind.
```bash
kubectl get nodes
```

Wenn alles gut aussieht, können Sie herunterladen und führen Sie den folgenden Dienst:
> [!Important] 
> Vor dem `kubectl apply`, stellen Sie sicher, dass zu double-check/Ändern der `microsoft/windowsservercore` Bild in der Beispieldatei zu [einem containerimage, die durch Ihre Knoten ausführbar ist](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions)!

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Dadurch entsteht eine Bereitstellung und ein Dienst. Der letzte Befehl überwachen fragt die Pods auf unbestimmte Zeit überwacht, um ihren Status zu verfolgen; Drücken Sie einfach `Ctrl+C` zum Beenden der `watch` Befehl, wenn mit der Beobachtung fertig.

Wenn alles erfolgreich verlaufen ist:

  - finden Sie unter 2 Containern pro Depot unter `docker ps` Befehl auf dem Windows-Knoten
  - werden 2 Pods unter einem `kubectl get pods`-Befehl vom Linus-Master angezeigt
  - `curl` für die *Pod*-IDs auf Port 80 des Linux-Masters eine Antwort vom Web-Server erhalten. Dies bestätigt die ordnungsgemäße Kommunikation zwischen Pod und Knoten im Netzwerk.
  - der Ping *zwischen Pods* (einschließlich zwischen Hosts, wenn Sie mehrere Windows-Knoten haben) über `docker exec`. Dies veranschaulicht die ordnungsgemäße Pod-zu-Pod-Kommunikation
  - `curl` die virtuelle *Dienst-IP* (unter gesehen `kubectl get services`) vom Linux-Master und den einzelnen Pods; Dies veranschaulicht die ordnungsgemäße Pod-Kommunikation-Dienst.
  - `curl` der *Dienstname* mit dem Kubernetes [Standard-DNS-Suffix](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services), zur richtigen Dienstermittlung.
  - `curl` die *NodePort* vom Linux-Master oder Computer außerhalb des Clusters; Dies veranschaulicht die eingehende Konnektivität.
  - `curl` externen IP-Adressen von innerhalb der Pod; Dies veranschaulicht die ausgehende Verbindungen.

> [!Note]  
> Windows- *containerhosts* wird **nicht** mehr auf die Dienst-IP von Diensten, die für diese geplant zugreifen können. Dies ist eine [bekannte Einschränkung der Plattform](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) , die in zukünftigen Versionen zu Windows Server verbessert wird. Windows *Pods* **sind** jedoch die Dienst-IP zugreifen.

### <a name="port-mapping"></a>Port-Zuordnung ### 
Es ist auch möglich, auf Dienste zuzugreifen, die in Pods über ihre jeweiligen Knoten gehostet werden, indem ein Port auf dem Knoten zugeordnet wird. Es ist ein [weiteres YAML-Beispiel verfügbar](https://github.com/Microsoft/SDN/blob/master/Kubernetes/PortMapping.yaml), das dieses Feature zeigt, mit einer Zuordnung von Port 4444 auf dem Knoten zu Port 80 auf dem Pod. Um es bereitzustellen, führen Sie die gleichen Schrittewie zuvor aus:

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/PortMapping.yaml -O win-webserver-port-mapped.yaml
kubectl apply -f win-webserver-port-mapped.yaml
watch kubectl get pods -o wide
```

Es sollte jetzt möglich sein, `curl`auf die *Knoten*-IP auf Port 4444 anzuwenden und eine Antwort des Webservers zu erhalten. Beachten Sie, dass dadurch die Skalierung auf einen einzelnen Pod pro Knoten beschränkt wird, da eine Eins-zu-Eins-Zuordnung erzwungen werden muss.


## <a name="next-steps"></a>Nächste Schritte ##
In diesem Abschnitt behandelt wir Planen von Kubernetes-Ressourcen auf Windows-Knoten. Dies schließt die Anleitung ab. Wenn Probleme aufgetreten ist, überprüfen Sie den Abschnitt Problembehandlung:

> [!div class="nextstepaction"]
> [Problembehandlung](./common-problems.md)

Andernfalls können Sie auch daran interessiert Kubernetes-Komponenten als Windows-Dienste nutzen:
> [!div class="nextstepaction"]
> [Windows-Dienste](./kube-windows-services.md)