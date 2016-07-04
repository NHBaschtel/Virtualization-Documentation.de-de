---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
translationtype: Human Translation
ms.sourcegitcommit: 2b85875eae1dcf1e50162e69c53dbf1ac7463450
ms.openlocfilehash: 8921cbd910bf657ddc4998e4214c1e9f9c3a01e9

---

# Verwaltung von Windows Server-Containern

**Dieser Inhalt ist vorläufig und kann geändert werden.** 

Zum Lebenszyklus von Containern gehören Aktionen wie das Starten, Beenden und Entfernen von Containern. Bei diesen Aktionen müssen Sie möglicherweise auch eine Liste mit Containerimages abrufen, Netzwerkverbindungen von Containern verwalten und Containerressourcen beschränken. Dieses Dokument erläutert grundlegende Aufgaben der Containerverwaltung mit Docker und bietet Links zu detaillierten Artikeln. 

## Verwalten von Containern

### Erstellen eines Containers

Verwenden Sie den Befehl `docker run`, um einen Container mit Docker zu erstellen.

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Weitere Informationen zum Docker-Befehl `run` finden Sie in der [Referenz zu „docker run“]( https://docs.docker.com/engine/reference/run/).

### Beenden eines Containers

Verwenden Sie den Befehl `docker stop`, um einen Container mit Docker zu beenden.

```none
PS C:\> docker stop tender_panini

tender_panini
```

Bei diesem Beispiel werden alle ausgeführten Container mit Docker beendet.

```none
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### Entfernen eines Containers

Verwenden Sie zum Entfernen eines Containers mit Docker den Befehl `docker rm`.

```none
PS C:\> docker rm prickly_pike

prickly_pike
``` 

So entfernen Sie alle Container mit Docker

```none
PS C:\> docker rm $(docker ps -aq)

dc3e282c064d
2230b0433370
```

Weitere Informationen zum Befehl „Docker rm“ finden Sie in der [Referenz zu Docker rm](https://docs.docker.com/engine/reference/commandline/rm/).



<!--HONumber=Jun16_HO4-->


