---
title: Konfigurieren von geschachtelten virtuellen Computer direkt mit Ressourcen in einem virtuellen Azure-Netzwerk kommunizieren
description: Geschachtelte Virtualisierung
keywords: Windows 10, hyper-V, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: 18ab4d1d87c22f70fe09aae5222a7d125ac9c974
ms.sourcegitcommit: 914e0dd1168daf1d2b0f22bd011035016cc08baf
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 02/23/2019
ms.locfileid: "9099388"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Konfigurieren von geschachtelten virtuellen Computern für die Kommunikation mit Ressourcen in einem virtuellen Azure-Netzwerk

Die ursprüngliche Anleitung zum Bereitstellen und Konfigurieren von geschachtelten virtuellen Computern in Azure ist erforderlich, dass Sie diese VMs über ein NAT-Switch zugreifen. Dadurch ergibt sich einige Einschränkungen:

1. Geschachtelte VMs können nicht auf lokale Ressourcen zugreifen oder in einem virtuellen Azure-Netzwerk.
2. Lokale Ressourcen oder Ressourcen in Azure können nur der geschachtelten virtuellen Computer einem NAT zugreifen, was bedeutet, dass mehrere Gäste denselben Port verwenden können nicht.

In diesem Dokument werden durch eine Bereitstellung geführt, wir von RRAS, Benutzer definiert Routen, eine dedizierte ausgehende NAT Gast Zugriff auf das Internet und einen "schwebenden" Adressraum Subnetz nutzen geschachtelte VMs Verhalten und wie die anderen virtuellen Computern kommunizieren zulassen direkt in eine vnet angefügt wird in Azure bereitgestellt.

Bevor Sie diese Anleitung beginnen, bitte:

1. Lesen Sie die [Anleitungen finden Sie hier](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/nested-virtualization) auf die geschachtelte Virtualisierung.
2. Lesen Sie diesen Artikel vor der Implementierung.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Hohe Überblick Ausblick und warum
* Wir erstellen eine Schachtelung können VM, die über zwei NICs verfügt. 
* Eine Netzwerkkarte verwendet wird, um unsere geschachtelten virtuellen Computer mit Internetzugriff über NAT und die anderen NIC bieten wird zum Weiterleiten von Datenverkehr von unseren internen Switch zu externen Ressourcen an den Hypervisor verwendet werden. Jede NIC müssen in einer anderen Domäne routing, d. h. ein anderen Subnetz.
* Dies bedeutet, dass wir ein virtuelles Netzwerk mit an eine minimale drei Subnetzen benötigen. Eine für NAT, eine für LAN-routing, und eine, wird nicht verwendet, aber ist für unsere geschachtelten virtuellen Computern "reserviert". Die Namen, die wir für die Subnetze in diesem Dokument verwenden sind, "NAT", "Hyper-V-LAN" und "Ghosted".
* Die Größe des diese Subnetze nach eigenem Ermessen ist, aber es gibt einige Punkte. Die Größe der Subnetze "Ghosted" bestimmt, wie viele IP-Adressen für Ihre geschachtelten virtuellen Computer besitzen. Wird auch die Größe der Subnetze "NAT" und "Hyper-V-LAN" bestimmt, wie viele IP-Adressen für Hypervisoren besitzen. Technisch wirklich kleinere Subnetze hier treffen zu können, wenn Sie nur auf ein oder zwei Hypervisoren planen wurden.
* Hintergrund: Erhalten DHCP geschachtelten virtuellen Computer wird nicht von der vnet angefügt wird, die mit ihrem Host verbunden ist, auch wenn Sie einem internen oder externen Switch konfigurieren. 
  * Dies bedeutet, dass der Hyper-V-Host DHCP angeben muss.
* Hyper-V-Host ist nicht im Hinblick auf die vnet angefügt wird, die derzeit zugewiesene Leases, damit um Situationen zu vermeiden, in dem der Host eine IP-Adresse bereits vorhanden ist, weist, einen Block von IP-Adressen für die Verwendung durch Hyper-V-Host zugeordnet werden muss. Dadurch können wir eine doppelte IP-Szenario zu vermeiden.
  * Der IP-Adressen wir wählen-Block entspricht mit einem Subnetz innerhalb der gleichen vnet angefügt wird, die in der Hyper-V ist.
  * Der Grund, dass dies zu einem vorhandenen Subnetz entsprechen soll ist, BGP-Ankündigungen über eine ExpressRoute zu behandeln. Wenn wir gerade bestehend aus eines IP-Bereichs für den Hyper-V-Host verwendet hätten wir erstellen eine Reihe von statischen Routen, damit Clients können in einer lokalen Umgebung für die Kommunikation mit der geschachtelten virtuellen Computer. Dies bedeutet, dass dies eine große Herausforderung ist nicht, da Sie konnte einen IP-Bereich für die geschachtelten virtuellen Computer bilden erstellen und dann alle Routen benötigt, um Clients an den Hyper-V-Host für diesen Bereich zu verweisen.
* Erstellen wir einen internen Switch in Hyper-V und wir werden weisen Sie der neu erstellten Schnittstelle einer IP-Adresse in einem Bereich, die, den wir für DHCP reserviert. Diese IP-Adresse wird der Standard-Gateway für unsere geschachtelten virtuellen Computer werden und werden verwendet, um die Route zwischen den internen Switch und die NIC des Hosts, die an unseren vnet angefügt wird verbunden ist.
* Wir installiert die Routing- und RAS-Rolle auf dem Host, die unsere Host in einem Router umgewandelt werden.  Dies ist erforderlich, um die Kommunikation zwischen Ressourcen außerhalb der Host und unsere geschachtelten virtuellen Computern zu ermöglichen.
* Wir werden andere Ressourcen zugreifen auf die geschachtelte VMs informieren. Dies ist erforderlich, dass wir eine benutzerdefinierte Route-Tabelle, die eine Route für die IP-Bereich, die erstellen enthält in der geschachtelten virtuellen Computer befinden. Diese statische Route wird auf die IP-Adresse für den Hyper-V verweisen.
* Klicken Sie dann setzen diese UDR auf dem Gateway-Subnetz Sie damit Clients aus lokalen wissen, wie unsere geschachtelten virtuellen Computer zugreifen.
* Setzen Sie diese UDR auch auf ein anderes Subnetz in Azure, die Verbindung mit der geschachtelten virtuellen Computer benötigt.
* Für mehrere Hyper-V-Hosts würden Sie zusätzliche "schwebenden" Subnetze erstellen und die UDR eine zusätzliche statische Route hinzugefügt.
* Wenn Sie Hyper-V-Host außer Betrieb nehmen Sie unsere "schwebenden" Subnetz löschen/wiederverwenden und entfernt diese statische Route aus unserer UDR, oder ist dies die letzte Hyper-V-Host die UDR vollständig entfernen.

## <a name="creating-the-host"></a>Erstellen des Hosts

Ich wird über alle Werte ignorieren, die bis zu überlassen, z. B. der Name, Ressourcengruppe VM usw...

1. Navigieren Sie zu portal.azure.com
2. Klicken Sie auf "Erstellen eine Ressource" in der oberen linken Ecke
3. Wählen Sie "Fenster Server 2016 VM" beliebte Spalte aus
4. Achten Sie auf der Registerkarte "Grundlagen" darauf, eine VM-Größe auszuwählen, die kann die geschachtelte Virtualisierung
5. Wechseln Sie zur Registerkarte "Netzwerke"
6. Erstellen Sie ein neues virtuelles Netzwerk mit der folgenden Konfiguration
    * Vnet angefügt wird Adressraum: 10.0.0.0/22
    * Subnetz 1
        * Name: NAT
        * Adressraum: 10.0.0.0/24
    * Subnetz 2
        * Name: Hyper-V-LAN
        * Adressraum: 10.0.1.0/24
    * Subnetz 3
        * Name: blasse Darstellung
        * Adressraum: 10.0.2.0/24
    * Subnetz 4
        * Name: Azure-VMs
        * Adressraum: 10.0.3.0/24
7. Stellen Sie sicher, dass Sie das NAT-Subnetz für den virtuellen Computer ausgewählt haben
8. Wechseln Sie zu "Rezension + erstellen" und wählen Sie "Erstellen"

## <a name="create-the-second-network-interface"></a>Erstellen Sie die zweite Netzwerkschnittstelle
1. Nach der virtuellen Computer wurde Bereitstellung wechseln Sie zu innerhalb des Azure-Portals
2. Beenden Sie den virtuellen Computer
3. Einmal beendet wechseln Sie zu "Netzwerke" unter Einstellungen
4. "Fügen Sie Netzwerkschnittstelle an"
5. "Erstellen Sie Netzwerkschnittstelle"
6. Geben sie einen Namen (nicht wichtig, was Sie nennen, aber Achten Sie darauf, dass Sie sich daran erinnern)
7. Wählen Sie "Hyper-V-LAN" für das Subnetz
8. Stellen Sie sicher, dass Sie die gleichen Ressourcengruppe auswählen in Ihrem Host befindet
9. "Erstellen"
10. Dadurch wird gelangen Sie zurück zum vorherigen Bildschirm, stellen Sie sicher, wählen die neu erstellte Netzwerkschnittstelle, und klicken Sie auf "OK"
11. Wechseln Sie zurück zum Fenster "Übersicht", und starten Sie Ihre virtuellen Computer erneut nach Abschluss die vorherige Aktion
12. Navigieren Sie auf die zweite Netzwerkkarte, die wir gerade erstellt haben, finden Sie es in der Gruppe Ressource, die Sie zuvor ausgewählt haben
13. Wechseln Sie zu "IP-Konfigurationen" und wechseln Sie "IP-Weiterleitung", "Enabled", und speichern Sie die Änderung

## <a name="setting-up-hyper-v"></a>Einrichten von Hyper-V
1. Remote in Ihrem host
2. Öffnen Sie eine PowerShell-Eingabeaufforderung
3. Führen Sie den folgenden Befehl `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. Dadurch wird den Host neu.
5. Schließen Sie wieder an den Host weiterhin mit dem Rest des Setups

## <a name="creating-our-virtual-switch"></a>Erstellen unsere virtuellen switch

1. Öffnen Sie PowerShell im Administratormodus.
2. Erstellen Sie einen internen Switch: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Weisen Sie der neu erstellten Schnittstelle einer IP-Adresse: `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>Installieren und Konfigurieren von DHCP

*Viele Personen verpassen diese Komponente, wenn er zuerst versucht, um die geschachtelte Virtualisierung zu verwenden. Im Gegensatz zu müssen in lokalen, in denen Ihre Gast-VMs DHCP über das Netzwerk, die der Host erhalten auf, geschachtelte VMs in Azure DHCP über den Host bereitgestellt werden, die sie ausgeführt. Oder Sie müssen statisch zuweisen eine IP-Adresse in jedem geschachtelten virtuellen Computer, die nicht skalierbar ist.*

1. Installieren Sie die DHCP-Rolle: `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Erstellen Sie den DHCP-Bereich: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. Konfigurieren Sie die DNS- und Standard-Gateway-Optionen für den Bereich: `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * Achten Sie darauf, dass Sie einen gültigen DNS-Server Eingabe, wenn Sie die Auflösung von Namen arbeiten möchten. In diesem Fall verwende ich [des Azure-rekursiven DNS](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances).

## <a name="installing-remote-access"></a>RAS-Installation

1. Öffnen Sie den Server-Manager, und wählen Sie "Hinzufügen von Rollen und Features".
2. Wählen Sie "Weiter" bis "Serverrollen" angezeigt.
3. Überprüfen Sie "Remote Access", und klicken Sie auf "Weiter", bis "Rollendienste" angezeigt.
4. Überprüfen Sie "Routing", wählen Sie "Features hinzufügen", und wählen Sie dann "Weiter", und klicken Sie dann "installieren". Schließen Sie den Assistenten, und warten Sie für die Installation abgeschlossen ist.

## <a name="configuring-remote-access"></a>Konfigurieren des Remotezugriffs

1. Öffnen Sie den Server-Manager, und wählen Sie "Extras", und wählen Sie dann "Routing-und RAS".
2. Auf der rechten Seite des Panels Management Routing- und RAS Sie sehen ein Symbol mit der Server-Namen daneben, rechten Maustaste klicken Sie hier, und wählen Sie "Konfigurieren und Aktivieren von Routing und RAS".
3. Wählen Sie über den Assistenten zum "Weiter", überprüfen Sie die Schaltfläche "radiale" für "Benutzerdefinierte Konfiguration", und wählen Sie dann auf "Weiter".
4. Überprüfen Sie "NAT" und "LAN-routing" und dann wählen Sie "Weiter" und "Abschließen". Wenn sie werden aufgefordert, starten Sie den Dienst, und klicken Sie dann dazu.
5. Nun navigieren Sie zum Knoten "IPv4", und erweitern Sie ihn so, dass der Knoten "NAT" zur Verfügung gestellt wird.
6. Klicken Sie mit der rechten Maustaste auf "NAT", wählen Sie "Neue Schnittstelle..." und wählen Sie "Ethernet", dies sollte die erste NIC mit der IP-Adresse des "10.0.0.4" sein
7. Nun müssen wir zum Erstellen von statischen Routen zum Erzwingen von LAN-Datenverkehr, die zweite Netzwerkkarte. Sie können dies, indem Sie auf den Knoten "Statischer Routen" unter "IPv4".
8. Einmal gibt es die folgenden Routen erstellen wir.
    * Route 1
        * Benutzeroberfläche: Ethernet
        * Ziel: 10.0.0.0
        * Netzwerk-Maske: 255.255.255.0
        * Gateway: 10.0.0.1
        * Metrik: 256
        * Hinweis: Wir fügen Sie ihn hier damit die primäre NIC auf Datenverkehr darauf, eine eigene Benutzeroberfläche reagieren können. Wenn wir dies hier hätten würde die folgende Route Datenverkehr für NIC-1, NIC 2 wechseln gedacht. Dadurch würde eine asymmetrische Route erstellt. 10.0.0.1 ist die IP-Adresse, die mit dem NAT-Subnetz Azure zuweist. Azure verwendet die erste verfügbaren IP-Adresse in einem Bereich als das Standardgateway. Daher sollte Sie 192.168.0.0/24 für das NAT-Subnetz verwendet haben, wäre das Gateway 192.168.0.1. Die spezifischere Route Routing Wins, d. h. diese Route ersetzt werden der unten Route.

    * Route 2
        * Schnittstelle: Ethernet 2
        * Ziel: 10.0.0.0
        * Netzwerk-Maske: 255.255.252.0
        * Gateway: 10.0.1.1
        * Metrik: 256
        * Hinweis: Dies ist eine Catch, die, den alle für unsere Azure vnet angefügt wird soll Datenverkehr weitergeleitet. Es wird Datenverkehr, der zweite Netzwerkkarte erzwungen. Sie müssen zusätzliche Routen für andere Bereiche hinzufügen von geschachtelten Ihre virtuellen Computer für den Zugriff auf. Wenn Sie sind daher lokales Netzwerk ist 172.16.0.0/22, und klicken Sie dann eine andere Route zu senden, die Datenverkehr, die zweite Netzwerkkarte für unsere Hypervisor möchten.

## <a name="creating-a-route-table-within-azure"></a>Erstellen eine Route-Tabelle in Azure

Finden Sie [in diesem Artikel](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal) eine gründlichere im Detail zum Erstellen und Verwalten von Routen in Azure lesen.

1. Navigieren Sie zu https://portal.azure.com.
2. Wählen Sie in der oberen linken Ecke "Erstellen eine Ressource".
3. Geben Sie in das Suchfeld "Routingtabelle", und geben Sie Treffertests.
4. Das oberste Ergebnis wird Route Tabelle wählen Sie den Namen und wählen Sie dann "Erstellen"
5. Benennen Sie die Route-Tabelle, in meinem Fall ich mit dem Namen "Routen-für-geschachtelte-VMs".
6. Stellen Sie sicher, dass Sie das gleiche Abonnement auswählen, dem in der Hyper-V-Hosts befinden.
7. Entweder erstellen Sie eine neue Ressource Gruppe oder wählen Sie eine vorhandene, und achten Sie darauf, dass das Region, in der Erstellung der Route-Tabelle in dem gleichen Bereich ist, die der Hyper-V-Host in befindet.
8. Wählen Sie erstellen"".

## <a name="configuring-the-route-table"></a>Konfigurieren der Routentabelle

1. Navigieren Sie zu der Route-Tabelle, die wir gerade erstellt haben. Hierzu können Sie den Namen der Routentabelle über die Suchleiste in der oberen Mitte des Portals gesucht.
2. Nachdem Sie ausgewählt haben, der Routingtabelle wechseln Sie zu "Strecken" von in das Blatt.
3. Wählen Sie "Hinzufügen".
4. Benennen Sie Ihre Route, ich ist ein Fehler mit "Geschachtelt-VMs".
5. Adresse Eingabe Präfix den IP-Bereich für unsere "schwebenden" Subnetz. In diesem Fall wäre es 10.0.2.0/24.
6. Für "Des nächsten Abschnitts Typ" Wählen Sie "Klarstellung" und geben Sie die IP-Adresse für den Hyper-V hostet die zweite Netzwerkkarte, die 10.0.1.4 werden würde, und wählen Sie dann auf "OK".
7. Jetzt ist von innerhalb der Blatt auswählen "Subnetze" direkt unter "Routen" ausführt.
8. Wählen Sie "Zuordnen", und klicken Sie dann wählen Sie unsere "Geschachtelt lustige" vnet angefügt wird und wählen Sie dann das Subnetz "Azure-VMs", und wählen Sie dann auf "OK".
9. Führen Sie denselben Prozess für das Subnetz, die unsere Hyper-V-Host auf sowie für alle anderen Subnetzen, die befinden auf der geschachtelten virtuellen Computer zugreifen müssen. Wenn verbunden 

# <a name="end-state-configuration-reference"></a>Referenz zu Ende State-Konfiguration
Die Umgebung in diesem Handbuch verfügt über die unten angegebenen Konfigurationen. Dieser Abschnitt gilt Inteded als Referenz verwendet werden.

1. Azure Virtual Network-Informationen.
    * Hohe vnet angefügt wird im Europäischen Wirtschaftsraum.
        * Name: Geschachtelte lustige
        * Adressraum: 10.0.0.0/22
        * Hinweis: Dies wird vier Subnetzen bestehen. Darüber hinaus sind diese Bereiche erstellten nicht festgelegt. Passen Sie Ihre Umgebung zu beheben. 

    * Erste Subnetz High Level Konfiguration.
        * Name: NAT
        * Adressraum: 10.0.0.0/24
        * Hinweis: Dies ist, in denen unsere Hyper-V primäre NIC hostet befindet. Dies wird zum Behandeln von ausgehenden NAT für die geschachtelten virtuellen Computer verwendet werden. Das Gateway mit dem Internet für Ihre geschachtelten virtuellen Computer werden.

    * Zweite Subnetz High Level Konfiguration.
        * Name: Hyper-V-LAN
        * Adressraum: 10.0.1.0/24
        * Hinweis: Unsere Hyper-V-Host einen zweiten Netzwerkadapter verfügen, die Sie behandeln das routing zwischen dem geschachtelten virtuellen Computern und Hyper-V-Host externe nicht-Internet-Ressourcen verwendet werden.

    * Dritte Subnetz High Level Konfiguration.
        * Name: blasse Darstellung
        * Adressraum: 10.0.2.0/24
        * Hinweis: Dies ist ein "schwebenden" Subnetz. Der Adressraum wird von unseren geschachtelten virtuellen Computer verwendet werden und ist vorhanden, um die Route Werbung an lokale behandeln. Keine VMs werden tatsächlich in diesem Subnetz bereitgestellt werden.

    * Vierte Subnetz High Level Konfiguration.
        * Name: Azure-VMs
        * Adressraum: 10.0.3.0/24
        * Hinweis: Subnetz, Azure-VMs enthält.

1. Unsere Hyper-V-Host hat die unten angegebenen NIC-Konfigurationen.
    * Primäre NIC 
        * IP-Adresse: 10.0.0.4
        * Subnetmask: 255.255.255.0
        * Standard-Gateway: 10.0.0.1
        * DNS: Für DHCP konfiguriert
        * IP-Weiterleitung aktiviert: Nein

    * Sekundäre NIC
        * IP-Adresse: 10.0.1.4
        * Subnetmask: 255.255.255.0
        * Standard-Gateway: leer
        * DNS: Für DHCP konfiguriert
        * IP-Weiterleitung aktiviert: Ja

    * Hyper-V-NIC für internen virtuellen Switch erstellt.
        * IP-Adresse: 10.0.2.1
        * Subnetmask: 255.255.255.0
        * Standard-Gateway: leer

3. Unsere Route-Tabelle wird eine Regel enthalten.
    * Regel 1
        * Name: Geschachtelte-VMs
        * Ziel: 10.0.2.0/24
        * Nächsten Hop: Klarstellung - 10.0.1.4

## <a name="conclusion"></a>Fazit

Sie sollten jetzt in der Lage zu einen virtuellen Computer (auch eine 32-Bit-VM!) in den Hyper-V-Host Bereitstellen von einer lokalen und in Azure zugegriffen werden.
