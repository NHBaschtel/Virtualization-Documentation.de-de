---
title: Remoteverwaltung eines Windows-Docker-Hosts
description: "Sichere Remoteverwaltung eines Docker-Hosts, auf dem Windows Server ausgeführt wird"
keywords: Docker, Container
author: taylorb-microsoft
ms.date: 02/14/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 0cc1b621-1a92-4512-8716-956d7a8fe495
translationtype: Human Translation
ms.sourcegitcommit: ac6ee9737fb779debc02eb11bb44fcdb7a8e28d4
ms.openlocfilehash: f837ef05dec3bd610409389cbf1bfbf9804d9d49
ms.lasthandoff: 02/14/2017

---
# Remoteverwaltung eines Windows-Docker-Hosts

Auch ohne `docker-machine` kann auf einem virtuellen Windows Server 2016-Computer ein Docker-Host mit Remoteverwaltung erstellt werden.

Die Schritte dazu sind sehr einfach:

* Erstellen Sie die Zertifikate auf dem Server mit [Dockertls](https://hub.docker.com/r/stefanscherer/dockertls-windows/). Wenn Sie Zertifikate mit einer IP-Adresse erstellen, sollten Sie eine statische IP-Adresse verwenden, um zu verhindern, dass die Zertifikate neu erstellt werden müssen, wenn sich die IP-Adresse ändert.

* Starten Sie den Docker-Dienst neu: `Restart-Service Docker`
* Machen Sie die Docker-TLS-Ports 2375 und 2376 durch Erstellen einer NSG Regel verfügbar, die eingehenden Datenverkehr zulässt. Beachten Sie, dass für sichere Verbindungen nur 2376 erforderlich ist. Das Portal sollte eine NSG-Konfiguration wie folgt anzeigen: ![NGSs](images/nsg.png)
* Lassen Sie eingehende Verbindungen über die Windows-Firewall zu. 
```
New-NetFirewallRule -DisplayName 'Docker SSL Inbound' -Profile @('Domain', 'Public', 'Private') -Direction Inbound -Action Allow -Protocol TCP -LocalPort 2376
```
* Kopieren Sie die Dateien `ca.pem`, „cert.pem“ und „key.pem“ aus dem Benutzer-Docker-Ordner auf Ihrem Computer (z. B. `c:\users\chris\.docker`) auf den lokalen Computer. Sie können z. B. mit STRG+C und STRG+V die Dateien einer RDP-Sitzung verwenden. 
* Stellen Sie sicher, dass Sie eine Verbindung mit dem Remote-Docker-Host herstellen können. Führen Sie Folgendes aus:
```
docker -D -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify --tlscacert=c:\
users\foo\.docker\client\ca.pem --tlscert=c:\users\foo\.docker\client\cert.pem --tlskey=c:\users\foo\.doc
ker\client\key.pem ps
```


## Problembehandlung
### Versuchen Sie die Verbindung ohne TLS herzustellen, um festzustellen, ob Ihre NSG-Firewall-Einstellungen korrekt sind.
Konnektivitätsfehler manifestieren sich in der Regel in Fehlern wie:
```
error during connect: Get https://wsdockerhost.southcentralus.cloudapp.azure.com:2376/v1.25/version: dial tcp 13.85.27.177:2376: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.
```

Erlauben Sie unverschlüsselte Verbindungen durch Hinzufügen von 
```
{
    "tlsverify":  false,
}
```
zu `c"\programdata\docker\config\daemon.json`, und starten Sie den Dienst dann neu.

Stellen Sie eine Verbindung mit dem Remotehost mit folgender Befehlszeile her:
```
docker -H tcp://wsdockerhost.southcentralus.cloudapp.azure.com:2376 --tlsverify=0 version
```

### Probleme mit Zertifikaten
Ein Zugriff auf den Docker-Host mit einem Zertifikat, das nicht für die IP-Adresse oder den DNS-Namen erstellt wurde, führt zu folgendem Fehler:
```
error during connect: Get https://w.x.y.c.z:2376/v1.25/containers/json: x509: certificate is valid for 127.0.0.1, a.b.c.d, not w.x.y.z
```
Stellen Sie sicher, dass w.x.y.z der DNS-Name für die öffentliche IP-Adresse des Hosts ist. Zudem muss der DNS-Name dem [allgemeinen Namen](https://www.ssl.com/faqs/common-name/) des Zertifikats entsprechen, also der `SERVER_NAME`-Umgebungsvariablen oder einer der IP-Adressen in der `IP_ADDRESSES`-Variablen, die für Dockertls bereitgestellt wurde.

### crypto/x509-Warnung
Möglicherweise werden Sie folgende Warnung erhalten: 
```
level=warning msg="Unable to use system certificate pool: crypto/x509: system root pool is not available on Windows"
```
Die Warnung hat keine Auswirkung.

