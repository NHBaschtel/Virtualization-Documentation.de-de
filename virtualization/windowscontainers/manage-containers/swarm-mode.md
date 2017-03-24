---
title: Erste Schritte mit dem Schwarmmodus
description: "Initialisieren einen Schwarmclusters, Erstellen eines Überlagerungsnetzwerks und Zuordnen eines Diensts zum Netzwerk"
keywords: Docker, Container, Schwarm, Orchestrierung
author: kallie-b
ms.date: 02/9/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5ceb9626-7c48-4d42-81f8-9c936595ad85
translationtype: Human Translation
ms.sourcegitcommit: f615c6dd268932a2ff99ac12c4e9ffdcf2cc217e
ms.openlocfilehash: ee6053003b31f226d2cfba8566f274ccc19d97ec
ms.lasthandoff: 03/02/2017

---

# Erste Schritte mit dem Schwarmmodus 

**Wichtiger Hinweis:** *Unterstützung für Schwarmmodus und Überlagerung ist derzeit nur für [Windows-Insider](https://insider.windows.com/) im bevorstehenden Windows 10 Creators Update verfügbar. Unterstützung für weitere Windows-Plattformen folgt in Kürze.*

## Was ist der Schwarmmodus?
Der Schwarmmodus ist ein Docker-Feature, das Funktionen zur Orchestrierung von Containern bereitstellt, beispielsweise das systemeigene Clustering von Docker-Hosts und die Planung der Containerarbeitslasten. Eine Gruppe von Docker-Hosts bildet einen „Schwarmcluster“, wenn ihre Docker-Computer zusammen im „Schwarmmodus“ arbeiten. Zusätzliche Informationen zum Schwarmmodus finden auf der [Website für die Docker-Dokumentation](https://docs.docker.com/engine/swarm/).

## Verwaltungsknoten und Arbeitsknoten
Ein Schwarm besteht aus zwei Arten von Containerhosts: *Verwaltungsknoten* und *Arbeitsknoten*. Jeder Schwarm wird über einen Verwaltungsknoten initialisiert, und alle Docker-CLI-Befehle zur Steuerung und Überwachung eines Schwarms müssen auf einem seiner Verwaltungsknoten ausgeführt werden. Verwaltungsknoten können als „Wächter“ für den Schwarmstatus betrachtet werden. Sie bilden eine Konsensgruppe, die den Zustand der Dienste überwacht, die im Schwarm ausgeführt werden, und die dafür sorgt, dass der tatsächliche Zustand des Schwarms immer mit seinem beabsichtigten, vom Entwickler oder Admin definierten Zustand übereinstimmt. 

>    **Hinweis:** Jeder Schwarm kann mehrere Verwaltungsknoten besitzen, muss aber über *mindestens einen* verfügen. 

Arbeitsknoten werden vom Docker-Schwarm über Verwaltungsknoten orchestriert. Um einem Schwarm beizutreten, muss ein Arbeitsknoten ein „Beitrittstoken“ verwenden, das vom Verwaltungsknoten bei der Initialisierung des Schwarms generiert wurde. Arbeitsknoten erhalten einfach nur Aufgaben von Verwaltungsknoten und führen sie aus, sodass sie keine Kenntnis vom Schwarmzustand besitzen (und benötigen).

## Systemvoraussetzungen für den Schwarmmodus

Mindestens ein physisches oder virtuelles Computersystem (um die volle Funktionalität des Schwarms zu nutzen, werden mindestens zwei Knoten empfohlen), auf dem das **Windows 10 Creators Update** ausgeführt wird (verfügbar für Mitglieder im [Windows-Insider](https://insider.windows.com/)-Programm) und das als Containerhost eingerichtet ist. (Weitere Details über erste Schritte mit Docker-Containern finden Sie im Thema [Windows-Container unter Windows 10](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10)).

**Docker-Modul v1.13.0 oder höher**

Offene Ports: Die folgenden Ports müssen auf jedem Host verfügbar sein. Auf einigen Systemen sind diese Ports standardmäßig geöffnet.
- TCP-Port 2377 für die Kommunikation zur Clusterverwaltung
- TCP- und UDP-Port 7946 für die Kommunikation zwischen Knoten
- TCP- und UDP-Port 4789 für Datenverkehr im Überlagerungsnetzwerk

## Initialisieren eines Schwarmclusters
Um einen Schwarm zu initialisieren, führen Sie einfach den folgenden Befehl auf einem der Containerhosts aus (ersetzen Sie dabei \<HOSTIPADDRESS\> durch die lokale IPv4-Adresse des Hostcomputers):

```none
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```
Wenn dieser Befehl auf einem Containerhost ausgeführt wird, beginnt das Docker-Modul auf diesem Host als Verwaltungsknoten im Schwarmmodus zu arbeiten.

## Hinzufügen von Knoten zu einem Schwarm

> **Hinweis:** Um den Schwarmmodus und die Features des Überlagerungsmodus zu nutzen, sind *nicht* mehrere Knoten erforderlich. Alle Schwarm- und Überlagerungsfeatures können auf einem einzelnen Host verwendet werden, der im Schwarmmodus arbeitet. (Dabei kann es sich beispielsweise um einen Verwaltungsknoten handeln, auf dem der Schwarmmodus mit dem Befehl `docker swarm init` initialisiert wurde.)

### Hinzufügen von Arbeitsknoten zu einem Schwarm
Nachdem ein Schwarm von einem Verwaltungsknoten initialisiert worden ist, können andere Hosts dem Schwarm mit folgendem einfachen Befehl als Arbeitsknoten hinzugefügt werden:

```none
C:\> docker swarm join --token <WORKERJOINTOKEN> <MANAGERIPADDRESS>
```

Hierbei ist \<MANAGERIPADDRESS\> die lokale IP-Adresse eines Verwaltungsknotens im Schwarm und \<WORKERJOINTOKEN\> das Beitrittstoken für den Arbeitsknoten, das als Ausgabe des auf dem Verwaltungsknoten ausgeführten Befehls `docker swarm init` bereitgestellt wird. Das Beitrittstoken kann auch mit einem der folgenden Befehle abgerufen werden, wenn der Befehl auf dem Verwaltungsknoten ausgeführt wird, nachdem der Schwarm initialisiert wurde:

```none
# Get the full command required to join a worker node to the swarm
C:\> docker swarm join-token worker

# Get only the join-token needed to join a worker node to the swarm
C:\> docker swarm join-token worker -q
```

### Hinzufügen von Verwaltungsknoten zu einem Schwarm
Zusätzliche Verwaltungsknoten können einem Schwarmcluster mit dem folgenden Befehl hinzugefügt werden:

```none
C:\> docker swarm join --token <MANAGERJOINTOKEN> <MANAGERIPADDRESS>
```

Auch in diesem Fall ist \<MANAGERIPADDRESS\> die lokale IP-Adresse eines Verwaltungsknotens im Schwarm. Das Beitrittstoken \<MANAGERJOINTOKEN\> ist ein Token eines *Verwaltungsknotens* im Schwarm, das durch die Ausführung eines der folgenden Befehle auf einem Verwaltungsknoten abgerufen werden kann:

```none
# Get the full command required to join a **manager** node to the swarm
C:\> docker swarm join-token manager

# Get only the join-token needed to join a **manager** node to the swarm
C:\> docker swarm join-token manager -q
```

## Erstellen eines Überlagerungsnetzwerks

Nach ein Schwarmcluster konfiguriert wurde, können im Schwarm Überlagerungsnetzwerke erstellt werden. Ein Überlagerungsnetzwerk kann durch Ausführen des folgenden Befehls auf einem Schwarmverwaltungsknoten erstellt werden:

```none
# Create an overlay network 
C:\> docker network create --driver=overlay <NETWORKNAME>
```

Hierbei ist \<NETWORKNAME\> der Name, den Sie Ihrem Netzwerk zuweisen möchten.

## Bereitstellen von Diensten für einen Schwarm
Nachdem ein Überlagerungsnetzwerk eingerichtet wurde, können Dienste erstellt und dem Netzwerk zugewiesen werden. Ein Netzwerk wird mit folgender Syntax erstellt:

```none
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

Hierbei ist \<SERVICENAME\> der Name, den Sie dem Dienst zuweisen möchten. Mit diesem Namen referenzieren Sie den Dienst über die Dienstermittlung (die den systemeigenen DNS-Server von Docker verwendet). \<NETWORKNAME\> ist der Name des Netzwerks (beispielsweise „myOverlayNet“), mit dem Sie diesem Dienst verbinden möchten. \<CONTAINERIMAGE\> ist der Name des Containerimages, das den Dienst definiert.

> **Hinweis:** Das zweite Argument `--endpoint-mode dnsrr` für diesen Befehl ist erforderlich, um das Docker-Modul darüber zu informieren, dass die DNS-Richtlinie Round Robin verwendet wird, um den Netzwerkdatenverkehr gleichmäßig auf die Containerendpunkte zu verteilen. DNS-Round-Robin ist derzeit die einzige Lastenausgleichsstrategie, die unter Windows unterstützt wird.[Routing-Mesh](https://docs.docker.com/engine/swarm/ingress/) wird für Windows-Docker-Hosts noch nicht unterstützt, aber demnächst. Benutzer, die schon jetzt eine alternative Lastenausgleichsstrategie wünschen, können ein externes Lastenausgleichssystem (z. B. NGINX) einrichten und den [publish-Portmodus](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) des Schwarms verwenden, um Containerhostports für den Lastenausgleich verfügbar zu machen.

## Skalieren eines Diensts
Nachdem ein Dienst für einen Schwarmcluster bereitgestellt worden ist, werden die Containerinstanzen, aus denen dieser Dienst gebildet wird, im Cluster bereitgestellt. Standardmäßig beruht ein Dienst auf nur einer Containerinstanz, „Replikat“ oder „Task“ genannt. Jedoch kann ein Dienst mit mehreren Tasks erstellt werden – entweder mit der Option `--replicas` in der `docker service create`-Anweisung oder durch Skalieren des Diensts nach der Erstellung.

Die Skalierbarkeit von Diensten ist ein wichtiger Vorteil eines Docker-Schwarms. Zum Skalieren genügt ein Docker-Befehl:

```none
C:\> docker service scale <SERVICENAME>=<REPLICAS>
```

Hierbei ist \<SERVICENAME\> der Name des zu skalierenden Diensts, und \<REPLICAS\> steht für die Anzahl der Tasks (oder Containerinstanzen), auf die Dienst skaliert wird.

## Anzeigen des Schwarmzustands

Es gibt mehrere Befehle, mit denen der Zustand des Schwarms und der im Schwarm ausgeführten Dienste ermittelt werden kann.

### Liste der Schwarmknoten
Verwenden Sie den folgenden Befehl, um eine Liste der Knoten (einschließlich Statusinformationen) anzuzeigen, die momentan zum Schwarm gehören. Dieser Befehl muss auf einem **Verwaltungsknoten** ausgeführt werden.

```none
C:\ docker node ls
```

In der Ausgabe dieses Befehls ist einer der Knoten mit einem Sternchen (*) markiert. Dies ist der aktuelle Knoten, also derjenige, auf dem der `docker node ls`-Befehl ausgeführt wurde.

### Liste der Netzwerke
Verwenden Sie den folgenden Befehl, um eine Liste der Netzwerke anzuzeigen, die auf einem bestimmten Knoten vorhanden sind. Um Überlagerungsnetzwerke anzuzeigen, muss dieser Befehl auf einem **Verwaltungsknoten** ausgeführt werden, der im Schwarmmodus arbeitet.

```none
C:\ docker network ls
```

### Liste der Dienste
Verwenden Sie den folgenden Befehl, um eine Liste der Dienste (einschließlich Statusinformationen) anzuzeigen, die momentan in einem Schwarm ausgeführt werden.

```none
C:\ docker service ls
```

### Liste der Containerinstanzen, die einen Dienst definieren
Verwenden Sie den folgenden Befehl, um Details zu den Containerinstanzen anzuzeigen, die für einen bestimmten Dienst ausgeführt werden. Die Ausgabe dieses Befehl umfasst die Knoten (und IDs), auf denen jeder Container ausgeführt wird, sowie Informationen über den Zustand der Container.  

```none
C:\ docker service ps <SERVICENAME>
```

## Einschränkungen
Momentan gelten unter Windows noch folgende Einschränkungen für den Schwarmmodus:
- Bekanntes Problem: Überlagerung und Schwarmmodus werden derzeit nur auf Hosts unterstützt, die über Ethernet verbunden sind. aber **nicht auf über WLAN verbunden Hosts.** Wir arbeiten an der Behebung dieses Problems.
- Verschlüsselung auf der Datenschicht wird nicht unterstützt (d. h. Datenverkehr von Container zu Container mit der `--opt encrypted`-Option).
- [Routing-Mesh](https://docs.docker.com/engine/swarm/ingress/) wird für Windows-Docker-Hosts noch nicht unterstützt, aber demnächst. Benutzer, die schon jetzt eine alternative Lastenausgleichsstrategie wünschen, können ein externes Lastenausgleichssystem (z. B. NGINX) einrichten und den [publish-Portmodus](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish) des Schwarms verwenden, um Containerhostports für den Lastenausgleich verfügbar zu machen.  



