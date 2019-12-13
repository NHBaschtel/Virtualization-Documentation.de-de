---
title: Beitreten zu Linux-Knoten
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Bereitstellen von Kubernetes-Ressourcen auf einem Kubernetes-Cluster mit gemischtem Betriebssystem.
keywords: kubernetes, 1,14, Windows, Getting Started
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: e6c569ae8d5bf50e24ea0fc7a6dd04734b60a863
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74909960"
---
# <a name="deploying-kubernetes-resources"></a>Bereitstellen von Kubernetes Ressourcen #
Angenommen, Sie verfügen über einen Kubernetes-Cluster, der aus mindestens einem Master-und 1-Worker besteht, können Sie Kubernetes-Ressourcen bereitstellen.
> [!TIP] 
> Seien Sie neugierig, welche Kubernetes-Ressourcen heute unter Windows unterstützt werden? Weitere Informationen finden Sie unter [offiziell unterstützte Features](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) und [Kubernetes in der Roadmap für Windows](https://github.com/orgs/kubernetes/projects/8) .


## <a name="running-a-sample-service"></a>Ausführen eines Beispiel Dienst ##
Sie werden einen sehr einfachen [PowerShell-basierten Webdienst](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) bereitstellen, um sicherzustellen, dass der Cluster erfolgreich hinzugefügt und unser Netzwerk ordnungsgemäß konfiguriert wurde.

Bevor Sie dies tun, ist es immer eine gute Idee, sicherzustellen, dass alle unsere Knoten fehlerfrei sind.
```bash
kubectl get nodes
```

Wenn alles gut aussieht, können Sie den folgenden Dienst herunterladen und ausführen:
> [!Important] 
> Vergewissern Sie sich vor der `kubectl apply`, dass Sie das `microsoft/windowsservercore` Abbild in der Beispieldatei auf [ein Container Image überprüfen, das von Ihren Knoten ausführbare Dateien ist](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions).

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Dadurch wird eine Bereitstellung und ein Dienst erstellt. Mit dem Befehl Letzte Überwachung werden die Pods unbegrenzt abgefragt, um Ihren Status zu verfolgen. Drücken Sie einfach `Ctrl+C`, um den `watch` Befehl zu beenden, wenn Sie den Vorgang beobachten.

Wenn alles erfolgreich verlaufen ist:

  - siehe 2 Container pro Pod unter `docker ps` Befehl auf dem Windows-Knoten
  - werden 2 Pods unter einem `kubectl get pods`-Befehl vom Linus-Master angezeigt
  - `curl` auf den *Pod* -IPS auf Port 80 des Linux-Masters erhält eine Webserver Antwort. Dies veranschaulicht die richtige Knoten-zu-Pod-Kommunikation über das Netzwerk.
  - der Ping *zwischen Pods* (einschließlich zwischen Hosts, wenn Sie mehrere Windows-Knoten haben) über `docker exec`. Dies veranschaulicht die ordnungsgemäße Pod-zu-Pod-Kommunikation
  - `curl` die *IP-Adresse des virtuellen Dienstanbieter* (unter `kubectl get services`) vom Linux-Master und von einzelnen Pods. Dies veranschaulicht die richtige Kommunikation zwischen Diensten.
  - `curl` Sie den *Dienstnamen* mit dem [DNS-Suffix](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services)Kubernetes, um die richtige Dienst Ermittlung zu demonstrieren.
  - `curl` Sie den *Knoten "nodeport* " aus dem Linux-Master oder den Computern außerhalb des Clusters. Dadurch wird die eingehende Konnektivität veranschaulicht.
  - `curl` externe IPS innerhalb des Pod. Dadurch wird die ausgehende Konnektivität veranschaulicht.

> [!Note]  
> Windows- *Container Hosts* sind **nicht** in der Lage, auf die Dienst-IP von den für Sie geplanten Diensten zuzugreifen. Dies ist eine [bekannte Platt Form Einschränkung](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) , die in zukünftigen Versionen von Windows Server verbessert wird. Windows- *Pods* **sind** jedoch in der Lage, auf die Dienst-IP zuzugreifen.

## <a name="next-steps"></a>Nächste Schritte ##
In diesem Abschnitt wird beschrieben, wie Sie Kubernetes-Ressourcen auf Windows-Knoten planen. Dies schließt die Anleitung ab. Wenn Probleme aufgetreten sind, lesen Sie den Abschnitt zur Problembehandlung:

> [!div class="nextstepaction"]
> [Problembehandlung](./common-problems.md)

Andernfalls sind Sie möglicherweise auch daran interessiert, Kubernetes-Komponenten als Windows-Dienste zu ausführen:
> [!div class="nextstepaction"]
> [Windows-Dienste](./kube-windows-services.md)
