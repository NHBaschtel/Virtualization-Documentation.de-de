---
author: neilpeterson
redirect_url: ../quick_start/manage_docker
---

# Verwaltung von Windows Server-Containern

<g id="1" ctype="x-strong">Dieser Inhalt ist vorläufig und kann geändert werden.</g>

Zum Lebenszyklus von Containern gehören Aktionen wie das Starten, Beenden und Entfernen von Containern. Bei diesen Aktionen müssen Sie möglicherweise auch eine Liste mit Containerimages abrufen, Netzwerkverbindungen von Containern verwalten und Containerressourcen beschränken. Dieses Dokument erläutert grundlegende Aufgaben der Containerverwaltung mit Docker und bietet Links zu detaillierten Artikeln.

## Verwalten von Containern

### Erstellen eines Containers

Verwenden Sie den Befehl <g id="2" ctype="x-code">docker run</g>, um einen Container mit Docker zu erstellen.

```none
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Weitere Informationen zum Docker-Befehl <g id="2" ctype="x-code">run</g> finden Sie in der <g id="4CapsExtId1" ctype="x-link"><g id="4CapsExtId2" ctype="x-linkText">Referenz zu docker run</g><g id="4CapsExtId3" ctype="x-title"></g></g>.

### Beenden eines Containers

Verwenden Sie den Befehl <g id="2" ctype="x-code">docker stop</g>, um einen Container mit Docker zu beenden.

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

Verwenden Sie zum Entfernen eines Containers mit Docker den Befehl <g id="2" ctype="x-code">docker rm</g>.

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

Weitere Informationen zum Befehl „Docker rm“ finden Sie in der <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Referenz zu Docker rm</g><g id="2CapsExtId3" ctype="x-title"></g></g>.






<!--HONumber=Apr16_HO5-->


