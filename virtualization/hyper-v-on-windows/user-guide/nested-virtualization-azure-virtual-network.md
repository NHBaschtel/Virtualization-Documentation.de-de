---
title: Konfigurieren von geschachtelten virtuellen Computern direkt mit Ressourcen in einer Azure virtuelles Netzwerk kommunizieren
description: Geschachtelte Virtualisierung
keywords: Windows 10, hyper-V, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: c39a2e5639d013a0aba15150117b7ab18ea32f97
ms.sourcegitcommit: 1aef193cf56dd0870139b5b8f901a8d9808ebdcd
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 01/11/2019
ms.locfileid: "9001656"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Konfigurieren von geschachtelten virtuellen Computern mit Ressourcen in einer Azure virtuelles Netzwerk kommunizieren

Die ursprüngliche Anleitung zum Bereitstellen und Konfigurieren von geschachtelten virtuellen Computern in Azure ist erforderlich, dass Sie diese virtuellen Computer über ein NAT-Switch zugreifen. Dies stellt mehrere Einschränkungen:

1. Geschachtelte VMs können nicht auf lokale Ressourcen zugreifen oder innerhalb eines virtuellen Azure-Netzwerks.
2. Lokalen Ressourcen oder Ressourcen in Azure zugreifen können nur der geschachtelten virtuellen Computer über eine NAT was bedeutet, dass mehrere Gäste auf demselben Port freigeben können.

Dieses Dokument enthält eine Bereitstellung, bei dem wir nutzen von RRAS, einige Benutzer definiert Routen und eine "schwebenden" Adressraum geschachtelte VMs Verhalten und wie alle anderen virtuellen Computer bereitgestellt, die direkt an eine vnet angefügt wird in Azure kommunizieren können.

Bevor Sie beginnen finden dieses Handbuch enthält:

1. Lesen Sie die [Anleitungen finden Sie hier](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/nested-virtualization) auf die geschachtelte Virtualisierung, erstellen Sie die Schachtelung können virtuelle Computer und installieren Sie die Hyper-V-Rolle in diese VMs. Fahren Sie nicht über die Hyper-V-Rolle einrichten.
2. Lesen Sie vor der Implementierung der Artikel.

Dieses Handbuch Annahmen folgende über die Ziel-Umgebung:

1. Wir arbeiten in einer [Sterntopologie](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)mit unseren Hub mit einer ExpressRoute verbunden.
1. Unser Netzwerk Netz zugewiesen Adressbereich des 10.0.0.0/23, die in zwei /24 Subnetze nahm.
    * 10.0.0.0/24 – Subnetz, in der Hyper-V-Host gespeichert ist.
    * 10.0.1.0/24 – Dies ist ein "schwebenden" Subnetz wird. Dieser Adressraum wird von unseren geschachtelten virtuellen Computer verwendet werden und ist vorhanden, um die Route Werbung auf lokale zu behandeln.
    * Das Netz vnet angefügt wird heißt "Netz" intelligente Weise.

1. Unsere Hub Netzwerke IP-Bereich ist nicht relevant, jedoch wissen, dass der Name "Hub" ist.
1. Unsere Hyper-V ist die Adresse des 10.0.0.4/24 zugewiesen.
1. Bei 10.0.0.10/24, haben wir einen DNS-Server, dies ist nicht erforderlich, aber eine Annahme, unsere Exemplarische Vorgehensweise.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Hohe Übersicht was wir tun und warum

* Hintergrund: Erhalten DHCP geschachtelten virtuellen Computer wird nicht von der vnet angefügt wird, die mit ihrem Host verbunden ist, auch wenn Sie einem internen oder externen Switch konfigurieren. 
  * Dies bedeutet, dass der Hyper-V-Host DHCP angeben muss.
* Wir werden einen Block von IP-Adressen für die Verwendung von Hyper-V-Host nur zuordnen.  Hyper-V-Host ist nicht im Hinblick auf die derzeit zugewiesene Leases auf die vnet angefügt wird, damit um Situationen zu vermeiden, in dem der Host eine IP-Adresse bereits vorhanden ist, weist, einen Block von IP-Adressen für die Verwendung von Hyper-V-Host nur zugeordnet werden muss. Dadurch können wir eine doppelte IP-Szenario zu vermeiden.
  * Subnetz innerhalb der vnet angefügt wird, die auf dem Hyper-V-Host befindet, wird der Block der IP-Adressen, die wir wählen entsprechen.
  * Der Grund, dass dies zu einem vorhandenen Subnetz entsprechen soll ist, BGP-Ankündigungen über die ExpressRoute zu behandeln. Wenn wir gerade bestehend aus eines IP-Bereichs für den Hyper-V-Host verwenden wir müsste erstellen Sie eine Reihe von statischen Routen, damit Clients können in einer lokalen Umgebung für die Kommunikation mit der geschachtelten virtuellen Computer. Dies bedeutet, dass dies eine große Herausforderung ist nicht, da Sie eine IP-Bereich für die geschachtelten virtuellen Computer bilden und Sie dann die Routen, die benötigt erstellen, um Clients, auf dem Hyper-V-Host für diesen Bereich zu verweisen.
* Erstellen wir einen internen Switch in Hyper-V und wir werden weisen Sie der neu erstellten Schnittstelle einer IP-Adresse in einem Bereich, die, den wir für DHCP reserviert. Diese IP-Adresse wird der Standard-Gateway für unsere geschachtelten virtuellen Computer werden und werden verwendet, um die Route zwischen den internen Switch und die NIC des Hosts, die an unseren vnet angefügt wird verbunden ist.
* Wir installiert die Routing- und RAS-Rolle auf dem Host, die unsere Host in einem Router umgewandelt werden.  Dies ist erforderlich, um die Kommunikation zwischen Ressourcen, die außerhalb der Host und unsere geschachtelten virtuellen Computern zu ermöglichen.
* Wir werden andere Ressourcen zugreifen auf die geschachtelte VMs informieren. Dies ist erforderlich, dass wir eine benutzerdefinierte Route-Tabelle, die eine Route für die IP-Bereich, die erstellen enthält in der geschachtelten virtuellen Computer befinden. Diese statische Route wird auf die IP-Adresse für den Hyper-V verweisen.
* Sie werden dann diese UDR auf dem Gateway-Subnetz platzieren, sodass Clients, die von lokalen wissen, wie unsere geschachtelten virtuellen Computern zu erreichen.
* Setzen Sie diese UDR auch auf ein anderes Subnetz in Azure, die Verbindung mit der geschachtelten virtuellen Computer benötigt.
* Für mehrere Hyper-V-Hosts würden Sie zusätzliche "schwebenden" Subnetze erstellen und die UDR eine zusätzliche statische Route hinzugefügt.
* Wenn Sie Hyper-V-Host außer Betrieb nehmen Sie unsere "schwebenden" Subnetz löschen/wiederverwenden und entfernt diese statische Route aus unserem UDR, oder ist dies die letzte Hyper-V-Host die UDR vollständig entfernen.

## <a name="creating-our-virtual-switch"></a>Erstellen unsere virtuellen Switch

1. Öffnen Sie PowerShell im Administratormodus.
2. Erstellen Sie einen internen Switch: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Weisen Sie der neu erstellten Schnittstelle einer IP-Adresse: `New-NetIPAddress –IPAddress 10.0.1.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>Installieren und Konfigurieren von DHCP

*Viele einzubeziehen dieser Komponente, wenn er zuerst versucht, um die geschachtelte Virtualisierung arbeiten zu erhalten. Im Gegensatz zu müssen in lokalen, in denen Ihre Gast-VMs DHCP über das Netzwerk, die der Host erhalten auf, geschachtelte virtuellen Computern in Azure DHCP über den Host bereitgestellt werden, die sie ausgeführt. Oder Sie müssen statisch zuweisen eine IP-Adresse in jedem geschachtelten virtuellen Computer, der nicht skalierbar ist.*

1. Installieren Sie die DHCP-Rolle: `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Erstellen Sie die DHCP-Bereich: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.1.2 -EndRange 10.0.1.254 -SubnetMask 255.255.255.0`
3. Konfigurieren Sie die DNS- und Standard-Gateway-Optionen für den Bereich: `Set-DhcpServerV4OptionValue -DnsServer 10.0.0.10 -Router 10.0.1.1`
    * Achten Sie darauf, dass Sie einen gültigen DNS-Server eingeben. In diesem Fall passieren ich einen Server im Netzwerk 10.0.0.0/24 verfügen, die von Windows-DNS fungiert.

## <a name="installing-remote-access"></a>RAS-Installation

* Öffnen Sie den Server-Manager, und wählen Sie "Hinzufügen von Rollen und Features".
* Wählen Sie "Weiter" bis "Serverrollen" angezeigt.
* Überprüfen Sie "Remote Access", und klicken Sie auf "Weiter", bis "Rollendienste" angezeigt.
* Überprüfen Sie "Routing", wählen Sie "Features hinzufügen", und wählen Sie dann "Weiter", und klicken Sie dann "installieren". Schließen Sie den Assistenten, und warten Sie für die Installation abgeschlossen ist.

## <a name="configuring-remote-access"></a>Konfigurieren des Remotezugriffs

* Öffnen Sie den Server-Manager, und wählen Sie "Extras", und wählen Sie dann "Routing-und RAS".
* Auf der rechten Seite des Panels Management RRAS -Sie sehen ein Symbol mit der Server-Namen daneben, rechten Maustaste klicken Sie hier, und wählen Sie "Konfigurieren und aktivieren Routing-und RAS".
* Wählen Sie über den Assistenten zum aus "Weiter", überprüfen Sie die radiale Schaltfläche "Benutzerdefinierte Konfiguration", und wählen Sie dann auf "Weiter".
* Überprüfen Sie "NAT" und "LAN-routing" und dann wählen Sie "Weiter", und dann "Beenden". Wenn sie werden aufgefordert, starten Sie den Dienst, und klicken Sie dann dazu.
* Nun navigieren Sie zum Knoten "IPv4", und erweitern Sie ihn, damit der Knoten "NAT" zur Verfügung gestellt wird.
* Klicken Sie mit der rechten Maustaste auf "NAT", wählen Sie "... neue Benutzeroberfläche", und Sie sollten drei Optionen angezeigt werden. 
* Zwei dieser Optionen sind für die virtueller Hyper-V-Switch, sollten Sie eine Option für "Ethernet" sehen, wählen Sie diese Option. Kurz gesagt, wählen Sie die Schnittstelle, die direkt mit Ihrer Azure vnet angefügt wird verbunden ist.

## <a name="creating-a-route-table-within-azure"></a>Erstellen eine Route-Tabelle in Azure

Finden Sie [in diesem Artikel](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal) eine gründlichere im Detail zum Erstellen und Verwalten von Routen in Azure zu lesen.

* Navigieren Sie zu https://portal.azure.com.
* Wählen Sie in der oberen linken Ecke "Erstellen eine Ressource".
* Geben Sie "Routingtabelle" in das Suchfeld ein, und geben Sie Treffertests.
* Das Ergebnis der oberste Route-Tabelle werden, wählen Sie diese und wählen Sie dann "Erstellen"
* Benennen Sie die Route-Tabelle, in meinem Fall ich mit dem Namen "Routen-für-geschachtelte-VMs".
* Stellen Sie sicher, dass Sie das gleiche Abonnement auswählen, dem in der Hyper-V-Hosts befinden.
* Erstellen eine neue Ressourcengruppe, oder wählen Sie eine vorhandene, und achten Sie darauf, dass der Region, in der Erstellung der Route-Tabelle, in dem gleichen Bereich, dem ist in der Hyper-V-Host befindet.
* Wählen Sie erstellen"".

## <a name="configuring-the-route-table"></a>Konfigurieren der Routentabelle

* Navigieren Sie zu der Route-Tabelle, die wir gerade erstellt haben. Dies ist möglich, indem eine Suche nach den Namen der Routentabelle über die Suchleiste, in der oberen Mitte des Portals.
* Nachdem Sie ausgewählt haben, in der Tabelle Route wechseln Sie zu "Routen" von in das Blatt.
* Wählen Sie "Hinzufügen".
* Benennen Sie die Route, ich ist ein Fehler mit "Geschachtelt-VMs".
* Für die Adresse Eingabe Präfix den IP-Bereich für unsere "schwebenden" Subnetz. In diesem Fall wäre es 10.0.1.0/24.
* Wählen Sie "Klarstellung" für "Nächsten Hop-Typ", und geben Sie die IP-Adresse für Hyper-V-Host, der 10.0.0.4 werden würden, und klicken Sie auf "OK".
* Nun wird von innerhalb der Blatt auswählen "Subnetze" dies direkt unter "Routen" sein.
* Wählen Sie "Zuordnen", und wählen Sie unsere "Hub" virtuelles Netzwerk und wählen Sie dann die "GatewaySubnet", und klicken Sie auf "OK".
* Führen Sie denselben Prozess für das Subnetz, die unsere Hyper-V-Host auf sowie für alle anderen Subnetzen, die auf der geschachtelten virtuellen Computer zugreifen müssen.

## <a name="conclusion"></a>Fazit

Sie sollten jetzt in der Lage zu einen virtuellen Computer (auch eine 32-Bit-VM!) bereitstellen, in den Hyper-V-Host aus lokalen und in Azure zugegriffen werden.
