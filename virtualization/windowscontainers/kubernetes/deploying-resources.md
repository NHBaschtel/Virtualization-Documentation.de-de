---
title: Beitreten zu Linux-Knoten
author: daschott
ms.author: daschott
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Bereitstellen von Kubernetes-Ressourcen auf einem Kubernetes-Cluster mit gemischten Betriebssystemen
keywords: kubernetes, 1,14, Windows, erste Schritte
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: e6c569ae8d5bf50e24ea0fc7a6dd04734b60a863
ms.sourcegitcommit: d252f356a3de98f224e1550536810dfc75345303
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 09/04/2019
ms.locfileid: "10069944"
---
# <a name="deploying-kubernetes-resources"></a>Bereitstellen von Kubernetes-Ressourcen #
Vorausgesetzt, Sie verfügen über einen Kubernetes-Cluster, der aus mindestens 1 Master und 1 Worker besteht, können Sie Kubernetes-Ressourcen bereitstellen.
> [!TIP] 
> Sind Sie neugierig, welche Kubernetes-Ressourcen unter Windows heute unterstützt werden? Weitere Informationen finden Sie unter [offiziell unterstützte Features](https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations) und [Kubernetes in Windows-Roadmap](https://github.com/orgs/kubernetes/projects/8) .


## <a name="running-a-sample-service"></a>Ausführen eines Beispieldiensts ##
Sie werden einen sehr einfachen [PowerShell-basierten Webdienst](https://github.com/Microsoft/SDN/blob/master/Kubernetes/WebServer.yaml) bereitstellen, um sicherzustellen, dass der Cluster erfolgreich hinzugefügt und unser Netzwerk ordnungsgemäß konfiguriert wurde.

Bevor Sie dies tun, ist es immer eine gute Idee, sicherzustellen, dass alle unsere Knoten fehlerfrei sind.
```bash
kubectl get nodes
```

Wenn alles gut aussieht, können Sie den folgenden Dienst herunterladen und ausführen:
> [!Important] 
> Vergewissern `kubectl apply`Sie sich zuvor, dass Sie das `microsoft/windowsservercore` Bild in der Beispieldatei auf [ein Container Bild, das von Ihren Knoten ausführbar ist](https://docs.microsoft.com/virtualization/windowscontainers/deploy-containers/version-compatibility#choosing-container-os-versions), überprüfen/ändern können!

```bash
wget https://raw.githubusercontent.com/Microsoft/SDN/master/Kubernetes/flannel/l2bridge/manifests/simpleweb.yml -O win-webserver.yaml
kubectl apply -f win-webserver.yaml
watch kubectl get pods -o wide
```

Dadurch wird eine Bereitstellung und ein Dienst erstellt. Der letzte Überwachungs Befehl fragt die Pods auf unbestimmte Zeit ab, um deren Status zu verfolgen. Drücken Sie `Ctrl+C` einfach, um `watch` den Befehl zu beenden, wenn Sie die Beobachtung durchgeführt haben.

Wenn alles erfolgreich verlaufen ist:

  - siehe 2 Container pro Pod unter `docker ps` Befehl auf dem Windows-Knoten
  - werden 2 Pods unter einem `kubectl get pods`-Befehl vom Linus-Master angezeigt
  - `curl` für die *Pod*-IDs auf Port 80 des Linux-Masters eine Antwort vom Web-Server erhalten. Dies bestätigt die ordnungsgemäße Kommunikation zwischen Pod und Knoten im Netzwerk.
  - der Ping *zwischen Pods* (einschließlich zwischen Hosts, wenn Sie mehrere Windows-Knoten haben) über `docker exec`. Dies veranschaulicht die ordnungsgemäße Pod-zu-Pod-Kommunikation
  - `curl` die virtuelle *Dienst-IP* (unter `kubectl get services`) vom Linux-Master und von einzelnen Pods; Damit wird der richtige Service für die Pod-Kommunikation veranschaulicht.
  - `curl` der *Dienstname* mit dem Kubernetes [-Standard-DNS-Suffix](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/#services), das die ordnungsgemäße Diensterkennung veranschaulicht.
  - `curl` deport vom Linux-Master oder Computern außerhalb des Clusters; ** Dadurch wird die eingehende Konnektivität veranschaulicht.
  - `curl` externe IPS innerhalb des Pod; Dadurch wird die ausgehende Konnektivität veranschaulicht.

> [!Note]  
> Windows- *Container Hosts* können von den für Sie geplanten Diensten **nicht** auf die Dienst-IP zugreifen. Hierbei handelt es sich um eine [bekannte Platt Form Einschränkung](./common-problems.md#my-windows-node-cannot-access-my-services-using-the-service-ip) , die in zukünftigen Versionen von Windows Server verbessert wird. Windows- *Pods* **** können jedoch auf die Dienst-IP zugreifen.

## <a name="next-steps"></a>Nächste Schritte ##
In diesem Abschnitt wird beschrieben, wie Sie Kubernetes-Ressourcen auf Windows-Knoten planen. Damit wird der Leitfaden beendet. Wenn Probleme aufgetreten sind, lesen Sie den Abschnitt Problembehandlung:

> [!div class="nextstepaction"]
> [Problembehandlung](./common-problems.md)

Andernfalls sind Sie möglicherweise auch daran interessiert, Kubernetes-Komponenten als Windows-Dienste auszuführen:
> [!div class="nextstepaction"]
> [Windows-Dienste](./kube-windows-services.md)
