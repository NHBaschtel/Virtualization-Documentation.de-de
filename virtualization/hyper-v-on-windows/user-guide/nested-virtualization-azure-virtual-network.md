---
title: Konfigurieren von geschachtelten VMs für die direkte Kommunikation mit Ressourcen in einem virtuellen Azure-Netzwerk
description: Geschachtelte Virtualisierung
keywords: Windows 10, Hyper-v, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: efd180c458457da1cea6b379e21ba3a37083d15a
ms.sourcegitcommit: c4a3f88d1663dd19336bfd4ede0368cb18550ac7
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 07/31/2019
ms.locfileid: "9883263"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Konfigurieren von geschachtelten VMs für die Kommunikation mit Ressourcen in einem virtuellen Azure-Netzwerk

Die ursprüngliche Anleitung zum Bereitstellen und Konfigurieren von geschachtelten virtuellen Computern in Azure erfordert, dass Sie über einen NAT-Schalter auf diese VMS zugreifen. Dies stellt mehrere Einschränkungen dar:

1. Geschachtelte VMs können nicht auf Ressourcen lokal oder in einem virtuellen Azure-Netzwerk zugreifen.
2. Lokale Ressourcen oder Ressourcen in Azure können nur über eine NAT auf die geschachtelten VMS zugreifen, was bedeutet, dass mehrere Gäste nicht denselben Port verwenden können.

In diesem Dokument wird eine Bereitstellung durchlaufen, bei der wir RRAS, benutzerdefinierte Routen, ein Subnetz für ausgehende NAT verwenden, um Gast-Internetzugriff zu ermöglichen, und einen "unverankerten" Adressraum, damit verschachtelte VMS sich wie alle anderen virtuellen Computer Verhalten und kommunizieren können. wird direkt in einem VNet in Azure bereitgestellt.

Bevor Sie mit diesem Leitfaden beginnen, gehen Sie bitte wie folgt vor:

1. Lesen Sie die [Anleitungen](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization) zur verschachtelten Virtualisierung.
2. Lesen Sie diesen gesamten Artikel vor der Implementierung.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Übersicht über das, was wir gerade tun und warum
* Wir erstellen eine Nesting-fähige VM mit zwei NICs. 
* Eine NIC wird verwendet, um unsere verschachtelten VMS mit Internetzugriff über NAT bereitzustellen, und die andere NIC wird verwendet, um den Datenverkehr von unserem internen Switch an Ressourcen außerhalb des Hypervisors zu leiten. Jede NIC muss sich in einer anderen Routingdomäne befinden, also in einem anderen Subnetz.
* Das bedeutet, dass wir ein virtuelles Netzwerk mit mindestens drei Subnetzen benötigen. Eine für NAT, eine für das LAN-Routing und eine, die nicht verwendet wird, aber für unsere geschachtelten VMS "reserviert" ist. Die Namen, die wir für die Subnetze in diesem Dokument verwenden, sind "NAT", "Hyper-V-LAN" und "ghosted".
* Die Größe dieser Subnetze liegt nach ihrem Ermessen, doch es gibt einige Überlegungen. Die Größe der "ghosted"-Subnetze legt fest, wie viele IPS für Ihre geschachtelten VMS vorhanden sind. Die Größe der Subnets "NAT" und "Hyper-V-LAN" bestimmt auch, wie viele IPS für Hypervisoren vorhanden sind. Sie könnten also wirklich kleine Subnets erstellen, wenn Sie nur ein oder zwei Hypervisoren planen.
* Hintergrund: geschachtelte VMS empfangen DHCP nicht von der VNet, mit der der Host verbunden ist, auch wenn Sie einen internen oder externen Schalter konfigurieren. 
  * Das bedeutet, dass der Hyper-V-Host DHCP bereitstellen muss.
* Der Hyper-v-Host ist sich der aktuell zugewiesenen Leases auf dem VNet nicht bewusst, daher müssen Sie, um eine Situation zu vermeiden, in der der Host eine bereits vorhandene IP-Adresse zuordnet, einen IPS-Block für die Verwendung durch den Hyper-V-Host zuweisen. Auf diese Weise können wir ein doppeltes IP-Szenario vermeiden.
  * Der von uns ausgewählte IPS-Block entspricht einem Subnetz innerhalb desselben VNet, in dem sich Ihr Hyper-V befindet.
  * Der Grund dafür, dass dies einem vorhandenen Subnetz entspricht, besteht darin, BGP-Ankündigungen wieder über eine Express Route zu verarbeiten. Wenn wir gerade einen IP-Bereich für den Hyper-V-Host erstellt haben, müssten wir eine Reihe von statischen Routen erstellen, damit Clients mit den geschachtelten VMS kommunizieren können. Dies bedeutet, dass es sich nicht um eine schwierige Anforderung handelt, da Sie einen IP-Bereich für die geschachtelten VMS bilden und dann alle erforderlichen Routen erstellen können, um Clients an den Hyper-V-Host für diesen Bereich zu verweisen.
* Wir werden innerhalb von Hyper-V einen internen Schalter erstellen und dann der neu erstellten Schnittstelleeine IP-Adresse in einem Bereich zuweisen, den wir für DHCP reserviert haben. Diese IP-Adresse wird zum Standardgateway für unsere geschachtelten VMS und dient zur Weiterleitung zwischen dem internen Switch und der NIC des Hosts, der mit unserem VNet verbunden ist.
* Wir werden die Routing-und RAS-Rolle auf dem Host installieren, wodurch unser Host zu einem Router wird.  Dies ist erforderlich, um die Kommunikation zwischen Ressourcen außerhalb des Hosts und unseren geschachtelten VMS zu ermöglichen.
* Wir werden anderen Ressourcen sagen, wie Sie auf diese geschachtelten VMS zugreifen können. Dies erfordert, dass wir eine benutzerdefinierte Routentabelle erstellen, die eine statische Route für den IP-Bereich enthält, in dem sich die geschachtelten VMs befinden. Diese statische Route zeigt auf die IP-Adresse für Hyper-V.
* Sie legen diese UDR dann auf dem Gateway-Subnetz ab, damit Clients, die von einem lokalen Standort kommen, wissen, wie Sie unsere geschachtelten VMS erreichen.
* Sie können dieses UDR auch in einem anderen Subnetz in Azure platzieren, das eine Verbindung mit den geschachtelten VMS erfordert.
* Für mehrere Hyper-V-Hosts würden Sie zusätzliche "unverankerte" Subnetze erstellen und der UDR eine zusätzliche statische Route hinzufügen.
* Wenn Sie einen Hyper-v-Host außer Betrieb setzen, werden Sie unser "Unverankertes" Subnetz löschen/neu verwenden und diese statische Route aus unserem UDR entfernen, oder wenn es sich um den letzten Hyper-v-Host handelt, entfernen Sie den UDR insgesamt.

## <a name="creating-the-host"></a>Erstellen des Hosts

Ich vertusche alle Konfigurationswerte, die persönlichen Vorlieben entsprechen, wie VM-Name, Ressourcengruppe usw.

1. Navigieren zu Portal.Azure.com
2. Klicken Sie in der oberen linken Ecke auf "Ressource erstellen".
3. Wählen Sie "Window Server 2016 VM" aus der beliebten Spalte aus.
4. Wählen Sie auf der Registerkarte "Grundlagen" eine VM-Größe aus, die in der Lage ist, Virtualisierung zu verschachteln.
5. Wechseln zur Registerkarte "Netzwerk"
6. Erstellen eines neuen virtuellen Netzwerks mit der folgenden Konfiguration
    * VNet-Adressraum: 10.0.0.0/22
    * Subnetz 1
        * Name: NAT
        * Adressraum: 10.0.0.0/24
    * Subnetz 2
        * Name: Hyper-V-LAN
        * Adressraum: 10.0.1.0/24
    * Subnetz 3
        * Name: ghosted
        * Adressraum: 10.0.2.0/24
    * Subnetz 4
        * Name: Azure-VMS
        * Adressraum: 10.0.3.0/24
7. Sicherstellen, dass Sie das NAT-Subnetz für den virtuellen Computer ausgewählt haben
8. Wechseln Sie zu "überprüfen + erstellen", und wählen Sie "erstellen" aus.

## <a name="create-the-second-network-interface"></a>Erstellen der zweiten Netzwerkschnittstelle
1. Nachdem die Bereitstellung des virtuellen Computers durchgeführt wurde, navigieren Sie im Azure-Portal zu ihm.
2. Beenden des VM
3. Einmal angehalten, gehen Sie zu "Netzwerk" unter Einstellungen.
4. "Netzwerkschnittstelle anfügen"
5. "Netzwerkschnittstelle erstellen"
6. Geben Sie ihm einen Namen (es spielt keine Rolle, was Sie ihm nennen, aber denken Sie daran, es zu merken).
7. "Hyper-V-LAN" für das Subnetz auswählen
8. Sicherstellen, dass Sie dieselbe Ressourcengruppe auswählen, in der sich der Host befindet
9. Erstellen
10. Damit kehren Sie zum vorherigen Bildschirm zurück, stellen Sie sicher, dass Sie die neu erstellte Netzwerkschnittstelle auswählen und "OK" auswählen.
11. Kehren Sie zum Bereich "Übersicht" zurück, und starten Sie den virtuellen Computer erneut, sobald die vorherige Aktion abgeschlossen ist.
12. Navigieren Sie zu der zweiten soeben erstellten NIC, die Sie in der zuvor ausgewählten Ressourcengruppe finden können.
13. Wechseln Sie zu "IP-Konfigurationen", und aktivieren Sie "IP-Weiterleitung" auf "aktiviert", und speichern Sie dann die Änderung.

## <a name="setting-up-hyper-v"></a>Einrichten von Hyper-V
1. Remote in Ihren Host
2. Öffnen einer erhöhten PowerShell-Eingabeaufforderung
3. Führen Sie den folgenden Befehl aus. `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. Dadurch wird der Host neu gestartet.
5. Wiederherstellen der Verbindung mit dem Host, um den restlichen Setupvorgang fortzusetzen

## <a name="creating-our-virtual-switch"></a>Erstellen unseres virtuellen Switches

1. Öffnen Sie PowerShell im Verwaltungsmodus.
2. Erstellen eines internen Schalters: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Zuweisen der neu erstellten Schnittstelle zu einer IP-Adresse: `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>Installieren und Konfigurieren von DHCP

*Viele Personen vermissen diese Komponente, wenn Sie zuerst versuchen, die verschachtelte Virtualisierung zu verwenden. Im Gegensatz zu lokalen Computern, auf denen Ihre Gast-VMS DHCP aus dem Netzwerk empfangen, auf dem sich Ihr Host befindet, müssen geschachtelte VMs in Azure über den Host, auf dem Sie ausgeführt werden, DHCP bereitgestellt werden. Oder Sie müssen jeder geschachtelten VM, die nicht skalierbar ist, statisch eine IP-Adresse zuweisen.*

1. Installieren Sie die DHCP-Rolle: `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Erstellen Sie den DHCP-Bereich: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. Konfigurieren Sie die DNS-und Standard Gateway-Optionen für den Bereich: `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * Achten Sie darauf, einen gültigen DNS-Server einzugeben, wenn die Namensauflösung funktionieren soll. In diesem Fall verwende ich [den rekursiven DNS von Azure](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances).

## <a name="installing-remote-access"></a>Installieren des Remote Zugriffs

1. Öffnen Sie den Server-Manager, und wählen Sie "Rollen und Features hinzufügen" aus.
2. Wählen Sie "weiter" aus, bis Sie "Server Rollen" erreichen.
3. Aktivieren Sie "Fernzugriff" und klicken Sie auf "weiter", bis Sie zu "Rollendienste" gelangen.
4. Aktivieren Sie "Routing", wählen Sie "Features hinzufügen" und dann "weiter" und dann "installieren" aus. Schließen Sie den Assistenten ab, und warten Sie, bis die Installation abgeschlossen ist.

## <a name="configuring-remote-access"></a>Konfigurieren des Remote Zugriffs

1. Öffnen Sie den Server-Manager, wählen Sie "Extras" und dann "Routing und Remote Zugriff" aus.
2. Auf der linken Seite des Verwaltungs Panels für Routing und RAS sehen Sie ein Symbol mit dem Namen Ihres Servers daneben, klicken Sie mit der rechten Maustaste darauf, und wählen Sie "Routing und Remote-Zugriff konfigurieren und aktivieren" aus.
3. Klicken Sie im Assistenten auf "weiter", aktivieren Sie die Radial-Schaltfläche für "benutzerdefinierte Konfiguration", und wählen Sie dann "weiter" aus.
4. Aktivieren Sie "NAT" und "LAN-Routing", und wählen Sie dann "weiter" und dann "Fertig stellen" aus. Wenn Sie aufgefordert werden, den Dienst zu starten, müssen Sie dies tun.
5. Navigieren Sie nun zum Knoten "IPv4", und erweitern Sie ihn so, dass der "NAT"-Knoten zur Verfügung gestellt wird.
6. Klicken Sie mit der rechten Maustaste auf "NAT", wählen Sie "neue Schnittstelle...". und wählen Sie "Ethernet", dies sollte Ihre erste NIC mit der IP-Adresse "10.0.0.4" sein.
7. Nun müssen wir einige statische Routen erstellen, um den LAN-Datenverkehr auf der zweiten NIC zu erzwingen. Gehen Sie dazu zum Knoten "statische Routen" unter "IPv4".
8. Anschließend erstellen wir die folgenden Routen.
    * Route 1
        * Schnittstelle: Ethernet
        * Ziel: 10.0.0.0
        * Netzwerkmaske: 255.255.255.0
        * Gateway: 10.0.0.1
        * Metrik: 256
        * Hinweis: Wir setzen dies hier ein, damit die primäre NIC auf den Verkehr reagieren kann, der für Sie eine eigene Schnittstelle bestimmt. Wenn dies nicht der Fall ist, würde die folgende Route dazu führen, dass der Datenverkehr für NIC 1 zu NIC 2 führt. Dadurch wird eine asymmetrische Route erstellt. 10.0.0.1 ist die IP-Adresse, die Azure dem NAT-Subnetz zuweist. Azure verwendet die erste verfügbare IP in einem Bereich als Standardgateway. Wenn Sie also 192.168.0.0/24 für Ihr NAT-Subnetz verwendet haben, wäre das Gateway 192.168.0.1. Bei der Weiterleitung gewinnt die spezifischere Route, was bedeutet, dass diese Route die untere Route überschreitet.

    * Route 2
        * Schnittstelle: Ethernet 2
        * Ziel: 10.0.0.0
        * Netzwerkmaske: 255.255.252.0
        * Gateway: 10.0.1.1
        * Metrik: 256
        * Hinweis: Hierbei handelt es sich um eine catch all-Route für den Datenverkehr, der für unsere Azure VNet vorgesehen ist. Dadurch wird der Datenverkehr aus der zweiten NIC erzwungen. Sie müssen weitere Routen für andere Bereiche hinzufügen, auf die ihre geschachtelten VMS zugreifen sollen. Wenn Sie also auf dem Prem-Netzwerk 172.16.0.0/22 sind, möchten Sie eine andere Route haben, um den Datenverkehr über die zweite NIC unseres Hypervisors zu senden.

## <a name="creating-a-route-table-within-azure"></a>Erstellen einer Routentabelle in Azure

In [diesem Artikel](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal) finden Sie ausführlichere Informationen zum Erstellen und Verwalten von Routen in Azure.

1. Navigieren Sie https://portal.azure.comzu.
2. Wählen Sie in der oberen linken Ecke "Ressource erstellen" aus.
3. Geben Sie im Suchfeld "Route-Tabelle" ein, und drücken Sie die EINGABETASTE.
4. Das oberste Ergebnis ist Route-Tabelle, wählen Sie diese aus, und wählen Sie dann "erstellen" aus.
5. Benennen Sie die Route-Tabelle, in meinem Fall nannte ich Sie "Routen-für-geschachtelt-VMS".
6. Stellen Sie sicher, dass Sie das gleiche Abonnement auswählen, in dem sich Ihre Hyper-V-Hosts befinden.
7. Erstellen Sie entweder eine neue Ressourcengruppe, oder wählen Sie eine vorhandene aus, und stellen Sie sicher, dass der Bereich, in dem Sie die Arbeitsplan Tabelle erstellen, der gleiche Bereich ist, in dem sich der Hyper-V-Host befindet.
8. Wählen Sie "erstellen" aus.

## <a name="configuring-the-route-table"></a>Konfigurieren der Route-Tabelle

1. Navigieren Sie zu der soeben erstellten Route-Tabelle. Sie können dies tun, indem Sie in der Suchleiste oben in der Mitte des Portals nach dem Namen der Route-Tabelle suchen.
2. Nachdem Sie die Route-Tabelle ausgewählt haben, wechseln Sie zu "Routen" innerhalb des Blades.
3. Wählen Sie "hinzufügen" aus.
4. Geben Sie Ihrer Route einen Namen, ich habe "geschachtelte VMS" besucht.
5. Für Adresspräfix geben Sie den IP-Bereich für unser "Floating"-Subnetz ein. In diesem Fall wäre es 10.0.2.0/24.
6. Wählen Sie für "Nächster Hop-Typ" die Option "Virtual Appliance" aus, und geben Sie dann die IP-Adresse für die zweite NIC des Hyper-V-Hosts ein, die 10.0.1.4 werden soll, und wählen Sie dann "OK" aus.
7. Wählen Sie nun innerhalb des Blades die Option "Subnets" aus, die sich direkt unterhalb von "Routen" befindet.
8. Wählen Sie "zuordnen" aus, wählen Sie dann unsere "verschachtelte-Fun"-VNet und dann das Subnetz "Azure-VMS" aus, und wählen Sie dann "OK" aus.
9. Führen Sie denselben Vorgang für das Subnetz durch, auf dem sich der Hyper-V-Host befindet, sowie für alle anderen Subnetze, die auf die geschachtelten VMS zugreifen müssen. Wenn verbunden 

# <a name="end-state-configuration-reference"></a>Endstatus-Konfigurationsreferenz
Die Umgebung in diesem Leitfaden enthält die folgenden Konfigurationen. Dieser Abschnitt ist als Referenz inteded zu verwenden.

1. Azure Virtual Network-Informationen.
    * VNet-Konfiguration auf hoher Ebene
        * Name: geschachtelt – Spaß
        * Adressraum: 10.0.0.0/22
        * Hinweis: Diese wird aus vier Subnetzen bestehen. Außerdem sind diese Bereiche nicht in Stein gemeißelt. Wenden Sie sich an Ihre Umgebung, wie Sie möchten. 

    * Konfiguration des ersten Subnetzes auf hoher Ebene
        * Name: NAT
        * Adressraum: 10.0.0.0/24
        * Hinweis: Hier befindet sich die primäre NIC des Hyper-V-Hosts. Diese wird verwendet, um ausgehende NAT für die geschachtelten VMS zu behandeln. Es ist das Gateway zum Internet für Ihre geschachtelten VMS.

    * Konfiguration des zweiten Subnetzes auf hoher Ebene
        * Name: Hyper-V-LAN
        * Adressraum: 10.0.1.0/24
        * Hinweis: Unser Hyper-v-Host verfügt über eine zweite NIC, die zur Behandlung des Routings zwischen den geschachtelten VMS-und nicht-Internet-Ressourcen außerhalb des Hyper-v-Hosts verwendet wird.

    * Konfiguration des dritten Subnets auf hoher Ebene
        * Name: ghosted
        * Adressraum: 10.0.2.0/24
        * Hinweis: Hierbei handelt es sich um ein "Unverankertes" Subnetz. Der Adressraum wird von unseren geschachtelten VMS verbraucht und ist für die Behandlung von Routenankündigungen wieder lokal verfügbar. In diesem Subnetz werden keine VMs bereitgestellt.

    * Konfiguration des vierten Subnets auf hoher Ebene
        * Name: Azure-VMS
        * Adressraum: 10.0.3.0/24
        * Hinweis: Subnetz mit Azure VMS.

1. Unser Hyper-V-Host verfügt über die folgenden NIC-Konfigurationen.
    * Primäre NIC 
        * IP-Adresse: 10.0.0.4
        * Subnetzmaske: 255.255.255.0
        * Standard Gateway: 10.0.0.1
        * DNS: konfiguriert für DHCP
        * IP-Weiterleitung aktiviert: Nein

    * Sekundäre NIC
        * IP-Adresse: 10.0.1.4
        * Subnetzmaske: 255.255.255.0
        * Standard Gateway: leer
        * DNS: konfiguriert für DHCP
        * IP-Weiterleitung aktiviert: Ja

    * Von Hyper-V erstellte NIC für internen virtuellen Switch
        * IP-Adresse: 10.0.2.1
        * Subnetzmaske: 255.255.255.0
        * Standard Gateway: leer

3. Unsere Route-Tabelle hat eine einzige Regel.
    * Regel 1
        * Name: geschachtelte VMS
        * Ziel: 10.0.2.0/24
        * Nächster Hop: Virtual Appliance-10.0.1.4

## <a name="conclusion"></a>Fazit

Sie sollten jetzt in der Lage sein, einen virtuellen Computer (sogar einen 32-Bit-VM!) für Ihren Hyper-V-Host bereitzustellen, auf den Sie von lokal und in Azure aus zugreifen können.
