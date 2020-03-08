---
title: Konfigurieren von virtuellen Computern für die direkte Kommunikation mit Ressourcen in einer Azure-Virtual Network
description: Geschachtelte Virtualisierung
keywords: Windows 10, Hyper-v, Azure
author: mrajess
ms.date: 12/10/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 1ecb85a6-d938-4c30-a29b-d18bd007ba08
ms.openlocfilehash: b7944e34cab66df07df0ccc78947a774d775c9a7
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853944"
---
# <a name="configure-nested-vms-to-communicate-with-resources-in-an-azure-virtual-network"></a>Konfigurieren von virtuellen Computern für die Kommunikation mit Ressourcen in einem virtuellen Azure-Netzwerk

Die ursprüngliche Anleitung zum Bereitstellen und Konfigurieren von geschachtelten virtuellen Computern in Azure erfordert, dass Sie über einen NAT-Switch auf diese VMS zugreifen. Dies stellt verschiedene Einschränkungen dar:

1. Geschachtelte VMs können nicht auf lokale Ressourcen oder innerhalb eines Azure-Virtual Network zugreifen.
2. Lokale Ressourcen oder Ressourcen in Azure können nur über eine NAT auf die geschachtelten VMS zugreifen. Dies bedeutet, dass mehrere Gäste denselben Port nicht gemeinsam verwenden können.

In diesem Dokument wird eine Bereitstellung erläutert, bei der wir RRAS, benutzerdefinierte Routen, ein Subnetz für die ausgehende NAT verwenden, um den Zugriff auf das Gast Betriebssystem zuzulassen, und einen "unverankerten" Adressraum, um zuzulassen, dass sich das Verhalten von virtuellen Computern und die Kommunikation wie jeder andere virtuelle Computer verhält. direkt in einem vnet in Azure bereitgestellt.

Bevor Sie mit diesem Leitfaden beginnen, sollten Sie Folgendes tun:

1. Lesen Sie die [hier bereitgestellten Anleitungen](https://docs.microsoft.com/azure/virtual-machines/windows/nested-virtualization) zur schsted Virtualisierung.
2. Lesen Sie den gesamten Artikel vor der Implementierung.

## <a name="high-level-overview-of-what-were-doing-and-why"></a>Allgemeine Übersicht über das, was wir tun und warum
* Wir erstellen einen geschachtelten virtuellen Computer mit zwei NICs. 
* Eine NIC wird verwendet, um unsere virtuellen Computer mit Internet Zugriff über NAT bereitzustellen, und die andere NIC wird zum Weiterleiten von Datenverkehr von unserem internen Switch an Ressourcen außerhalb des Hypervisors verwendet. Jede NIC muss sich in einer anderen Routing Domäne befinden, d. h. ein anderes Subnetz.
* Dies bedeutet, dass wir eine Virtual Network mit mindestens drei Subnetzen benötigen. Eine für NAT, eine für das LAN-Routing und eine, die nicht verwendet wird, aber für unsere virtuellen Computer "reserviert" ist. Die Namen, die wir für die Subnetze in diesem Dokument verwenden, sind "NAT", "Hyper-V-LAN" und "ghosted".
* Die Größe dieser Subnetze liegt nach ihrem Ermessen, aber es gibt einige Überlegungen. Die Größe der "ghosted"-Subnetze bestimmt, wie viele IPS für Ihre virtuellen Computer vorhanden sind. Außerdem bestimmen die Größe der Subnetze "NAT" und "Hyper-V-LAN", wie viele IPS für Hypervisoren vorhanden sind. Daher könnten Sie technisch gesehen sehr kleine Subnetze erstellen, wenn Sie nur einen oder zwei Hypervisoren planen.
* Hintergrund: bei den virtuellen Computern wird DHCP nicht aus dem vnet empfangen, mit dem der Host verbunden ist, auch wenn Sie einen internen oder externen Switch konfigurieren. 
  * Dies bedeutet, dass der Hyper-V-Host DHCP bereitstellen muss.
* Der Hyper-v-Host kennt die derzeit zugewiesenen Leases im vnet nicht. um eine Situation zu vermeiden, in der der Host eine bereits bestehende IP-Adresse zuweist, müssen wir einen Block von IP-Adressen für die Verwendung durch den Hyper-v-Host zuordnen. Auf diese Weise können wir ein doppeltes IP-Szenario vermeiden.
  * Der von uns gewählte IP-Block entspricht einem Subnetz innerhalb desselben vnets, in dem sich Ihr Hyper-V befindet.
  * Dies soll einem vorhandenen Subnetz entsprechen, wenn BGP-Ankündigungen über eine expressroute-Verbindung verarbeitet werden sollen. Wenn wir soeben einen IP-Adressbereich für den Hyper-V-Host erstellt haben, müssten wir eine Reihe statischer Routen erstellen, um den lokalen Clients die Kommunikation mit den virtuellen Computern zu ermöglichen. Dies bedeutet, dass dies keine feste Anforderung ist, da Sie einen IP-Adressbereich für die virtuellen Computer erstellen und dann alle Routen erstellen können, die zum Weiterleiten von Clients an den Hyper-V-Host für diesen Bereich benötigt werden.
* Wir erstellen einen internen Switch in Hyper-V und weisen dann die neu erstellte Schnittstelle einer IP-Adresse in einem Bereich zu, der für DHCP reserviert wurde. Diese IP-Adresse wird als Standard Gateway für unsere virtuellen Computer verwendet und zum Weiterleiten zwischen dem internen Switch und der NIC des Hosts verwendet, der mit dem vnet verbunden ist.
* Wir installieren die Routing-und RAS-Rolle auf dem Host, wodurch der Host in einen Router verwandelt wird.  Dies ist erforderlich, um die Kommunikation zwischen Ressourcen zuzulassen, die sich außerhalb des Hosts und der virtuellen Computer befinden.
* Wir werden anderen Ressourcen mitteilen, wie Sie auf diese virtuellen Computer zugreifen können. Dies erfordert, dass eine benutzerdefinierte Routing Tabelle erstellt wird, die eine statische Route für den IP-Adressbereich enthält, in dem sich die virtuellen Computer befinden. Diese statische Route verweist auf die IP-Adresse für den Hyper-V.
* Anschließend platzieren Sie diese UDR im gatewaysubnetz, damit von der lokalen Umgebung stammende Clients wissen, wie wir unsere virtuellen Computer erreichen.
* Außerdem platzieren Sie diese UDR in jedem anderen Subnetz innerhalb von Azure, das eine Verbindung mit den geschachtelten VMS erfordert.
* Für mehrere Hyper-V-Hosts würden Sie zusätzliche "Floating" Subnetze erstellen und der UDR eine zusätzliche statische Route hinzufügen.
* Wenn Sie einen Hyper-v-Host außer Betrieb nehmen, löschen Sie das "unverankerte" Subnetz und Entfernen dieses, und entfernen Sie die statische Route aus der UDR. wenn es sich um den letzten Hyper-v-Host handelt, entfernen Sie die UDR vollständig.

## <a name="creating-the-host"></a>Erstellen des Hosts

Ich werde alle Konfigurationswerte, die auf persönliche Einstellungen basieren (z. b. VM-Name, Ressourcengruppe usw.).

1. Navigieren Sie zu Portal.Azure.com
2. Klicken Sie oben links auf "Ressource erstellen".
3. Wählen Sie in der beliebten Spalte "Windows Server 2016 VM" aus.
4. Wählen Sie auf der Registerkarte "Grundlagen" eine VM-Größe aus, die für die Netzwerkvirtualisierung geeignet ist.
5. Wechseln Sie zur Registerkarte "Netzwerk".
6. Erstellen Sie eine neue Virtual Network mit der folgenden Konfiguration:
    * Vnet-Adressraum: 10.0.0.0/22
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
7. Stellen Sie sicher, dass Sie das NAT-Subnetz für die VM ausgewählt haben
8. Wechseln Sie zu "überprüfen und erstellen", und wählen Sie "erstellen" aus.

## <a name="create-the-second-network-interface"></a>Erstellen der zweiten Netzwerkschnittstelle
1. Nachdem die Bereitstellung der VM abgeschlossen ist, navigieren Sie im Azure-Portal zu der VM.
2. Die VM wird beendet.
3. Nach dem Beenden wechseln Sie unter "Einstellungen" zu "Netzwerk".
4. "Netzwerkschnittstelle anfügen"
5. "Netzwerkschnittstelle erstellen"
6. Geben Sie einen Namen ein (dabei spielt es keine Rolle, wie Sie ihn benennen, aber denken Sie daran, ihn zu merken).
7. Wählen Sie "Hyper-V-LAN" für das Subnetz aus.
8. Wählen Sie dieselbe Ressourcengruppe aus, in der sich Ihr Host befindet.
9. Stelle
10. Dadurch gelangen Sie zurück zum vorherigen Bildschirm. Stellen Sie sicher, dass die neu erstellte Netzwerkschnittstelle ausgewählt ist, und wählen Sie "OK" aus.
11. Wechseln Sie zurück zum Bereich "Übersicht", und starten Sie den virtuellen Computer erneut, sobald die vorherige Aktion abgeschlossen ist.
12. Navigieren Sie zur zweiten NIC, die wir soeben erstellt haben. Sie finden Sie in der zuvor ausgewählten Ressourcengruppe.
13. Wechseln Sie zu "IP-Konfigurationen", und schalten Sie "IP-Weiterleitung" in "aktiviert" um, und speichern Sie die Änderung.

## <a name="setting-up-hyper-v"></a>Einrichten von Hyper-V
1. Remote Verbindung mit Ihrem Host
2. Öffnen einer PowerShell-Eingabeaufforderung mit erhöhten Rechten
3. Führen Sie den folgenden Befehl aus `Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart`
4. Dadurch wird der Host neu gestartet.
5. Stellen Sie erneut eine Verbindung mit dem Host her, um mit dem Rest des Setups fortzufahren.

## <a name="creating-our-virtual-switch"></a>Der virtuelle Switch wird erstellt.

1. Öffnen Sie PowerShell im Verwaltungsmodus.
2. Erstellen Sie einen internen Switch: `New-VMSwitch -Name "NestedSwitch" -SwitchType Internal`
3. Zuweisen der neu erstellten Schnittstelle zu einer IP-Adresse: `New-NetIPAddress –IPAddress 10.0.2.1 -PrefixLength 24 -InterfaceAlias "vEthernet (NestedSwitch)"`

## <a name="install-and-configure-dhcp"></a>Installieren und Konfigurieren von DHCP

*Viele Personen übersehen diese Komponente, wenn Sie zum ersten Mal versuchen, die geschlagene Virtualisierung zu verwenden. Anders als bei lokalen Standorten, in denen Ihre Gast-VMS DHCP von dem Netzwerk empfangen, auf dem sich der Host befindet, muss für die virtuellen Computer in Azure DHCP über den Host bereitgestellt werden, auf dem Sie ausgeführt werden. Oder Sie müssen jedem virtuellen Computer, der nicht skalierbar ist, statisch eine IP-Adresse zuweisen.*

1. Installieren Sie die DHCP-Rolle: `Install-WindowsFeature DHCP -IncludeManagementTools`
2. Erstellen Sie den DHCP-Bereich: `Add-DhcpServerV4Scope -Name "Nested VMs" -StartRange 10.0.2.2 -EndRange 10.0.2.254 -SubnetMask 255.255.255.0`
3. Konfigurieren Sie die DNS-und standardgatewayoptionen für den Bereich: `Set-DhcpServerV4OptionValue -DnsServer 168.63.129.16 -Router 10.0.2.1`
    * Stellen Sie sicher, dass Sie einen gültigen DNS-Server eingeben, wenn die Namensauflösung funktionieren soll. In diesem Fall verwende ich [das rekursive DNS von Azure](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances).

## <a name="installing-remote-access"></a>Installieren des Remote Zugriffs

1. Öffnen Sie Server-Manager, und wählen Sie "Rollen und Features hinzufügen" aus.
2. Wählen Sie "weiter", bis Sie zu "Server Rollen" gelangen.
3. Aktivieren Sie "Remote Zugriff", und klicken Sie auf "weiter", bis Sie zu "Rollen Dienste" gelangen.
4. Aktivieren Sie "Routing", wählen Sie "Features hinzufügen", und klicken Sie dann auf "weiter" und dann auf "installieren". Beenden Sie den Assistenten, und warten Sie, bis die Installation beendet ist.

## <a name="configuring-remote-access"></a>Konfigurieren des Remote Zugriffs

1. Öffnen Sie Server-Manager, wählen Sie "Tools" aus, und wählen Sie dann "Routing und Remote Zugriff" aus.
2. Auf der linken Seite des Bereichs Routing-und RAS-Verwaltung wird ein Symbol mit dem Namen Ihres Servers angezeigt. Klicken Sie mit der rechten Maustaste darauf, und wählen Sie "Konfigurieren und Aktivieren von Routing und Remote Zugriff" aus.
3. Klicken Sie im Assistenten auf "weiter", aktivieren Sie die radiale Schaltfläche "benutzerdefinierte Konfiguration", und wählen Sie "weiter" aus.
4. Überprüfen Sie "NAT" und "LAN-Routing", und wählen Sie "weiter" und dann "Fertigstellen". Wenn Sie aufgefordert werden, den Dienst zu starten, gehen Sie so vor.
5. Navigieren Sie nun zum Knoten "IPv4", und erweitern Sie ihn, sodass der NAT-Knoten verfügbar gemacht wird.
6. Klicken Sie mit der rechten Maustaste auf "NAT", und wählen Sie "neue Schnittstelle". Wählen Sie "Ethernet" aus. Dies sollte Ihre erste NIC mit der IP-Adresse "10.0.0.4" sein, und wählen Sie die öffentliche Schnittstelle mit dem Internet verbinden aus, und aktivieren Sie NAT auf dieser Schnittstelle. 
7. Nun müssen einige statische Routen erstellt werden, um den LAN-Datenverkehr über die zweite NIC zu erzwingen. Gehen Sie hierzu zum Knoten "statische Routen" unter "IPv4".
8. Anschließend erstellen wir die folgenden Routen.
    * Route 1
        * Schnittstelle: Ethernet
        * Ziel: 10.0.0.0
        * Netzwerk Maske: 255.255.255.0
        * Gateway: 10.0.0.1
        * Metrik: 256
        * Hinweis: Wir legen dies hier ab, damit die primäre NIC auf den Datenverkehr reagieren kann, der für die eigene Schnittstelle vorgesehen ist. Wenn dies nicht der Fall ist, würde die folgende Route dazu führen, dass der Datenverkehr für NIC 1 an NIC 2 weitergeleitet wird. Dadurch wird eine asymmetrische Route erstellt. 10.0.0.1 ist die IP-Adresse, die Azure dem NAT-Subnetz zuweist. In Azure wird die erste verfügbare IP-Adresse in einem Bereich als Standard Gateway verwendet. Wenn Sie also 192.168.0.0/24 für das NAT-Subnetz verwendet haben, wäre das Gateway 192.168.0.1. Bei der Weiterleitung der spezifischeren Route gewinnt das, was bedeutet, dass diese Route die nachfolgende Route aufsetzt.

    * Route 2
        * Schnittstelle: Ethernet 2
        * Ziel: 10.0.0.0
        * Netzwerk Maske: 255.255.252.0
        * Gateway: 10.0.1.1
        * Metrik: 256
        * Hinweis: Dies ist eine catch all-Route für den Datenverkehr, der für unser Azure-vnet gedacht ist. Dadurch wird der Datenverkehr über die zweite NIC erzwungen. Sie müssen zusätzliche Routen für andere Bereiche hinzufügen, auf die die netsted VMS zugreifen sollen. Wenn Sie also das lokale Netzwerk "172.16.0.0/22" verwenden, benötigen Sie eine andere Route zum Senden des Datenverkehrs an die zweite NIC des Hypervisors.

## <a name="creating-a-route-table-within-azure"></a>Erstellen einer Routentabelle in Azure

Ausführliche Informationen zum Erstellen und Verwalten von Routen in Azure finden Sie in [diesem Artikel](https://docs.microsoft.com/azure/virtual-network/tutorial-create-route-table-portal) .

1. Navigieren Sie zu https://portal.azure.com.
2. Wählen Sie in der oberen linken Ecke "Ressource erstellen" aus.
3. Geben Sie im Suchfeld "Routing Tabelle" ein, und drücken Sie die EINGABETASTE.
4. Das oberste Ergebnis ist Routing Tabelle, wählen Sie diese aus, und wählen Sie dann "erstellen" aus.
5. Nennen Sie die Routing Tabelle, in meinem Fall "Routes-for-netsted-VMS".
6. Stellen Sie sicher, dass Sie das Abonnement auswählen, in dem sich auch Ihre Hyper-V-Hosts befinden.
7. Erstellen Sie entweder eine neue Ressourcengruppe, oder wählen Sie eine vorhandene aus, und stellen Sie sicher, dass die Region, in der Sie die Routing Tabelle erstellen, dieselbe Region ist, in der sich der Hyper-V-Host befindet.
8. Wählen Sie "erstellen" aus.

## <a name="configuring-the-route-table"></a>Konfigurieren der Routing Tabelle

1. Navigieren Sie zu der Routentabelle, die wir soeben erstellt haben. Sie können dies erreichen, indem Sie in der Suchleiste oben im Portal auf der Suchleiste nach dem Namen der Routentabelle suchen.
2. Nachdem Sie die Routing Tabelle ausgewählt haben, wechseln Sie auf dem Blatt zu "Routen".
3. Wählen Sie "hinzufügen" aus.
4. Benennen Sie Ihre Route, und ich habe "netsted-VMS" erhalten.
5. Geben Sie für Adress Präfix den IP-Adressbereich für das "Floating"-Subnetz ein. In diesem Fall wäre es 10.0.2.0/24.
6. Wählen Sie für "Typ des nächsten Hops" die Option "virtuelles Gerät" aus, und geben Sie dann die IP-Adresse für die zweite NIC der Hyper-V-Hosts ein 10.0.1.4, und wählen Sie dann "OK" aus.
7. Wählen Sie nun auf dem Blatt "Subnetze" aus. dieser wird direkt unterhalb von "Routen" angezeigt.
8. Wählen Sie "zuordnen", und wählen Sie dann das vnet "nsted-Fun" aus. Wählen Sie dann das Subnetz "Azure-VMS" aus, und wählen Sie "OK" aus.
9. Führen Sie den gleichen Prozess für das Subnetz aus, in dem sich auch der Hyper-V-Host befindet, sowie für alle anderen Subnetze, die auf die geclusterte VMS zugreifen müssen. Wenn verbunden 

# <a name="end-state-configuration-reference"></a>Referenz zum Endzustands Konfigurations Referenz
Die Umgebung in dieser Anleitung umfasst die folgenden Konfigurationen. Dieser Abschnitt ist als Verweis zu verwenden.

1. Informationen zu Azure Virtual Network.
    * Vnet-Konfiguration auf hoher Ebene.
        * Name: netsted-Fun
        * Adressraum: 10.0.0.0/22
        * Hinweis: Dies wird aus vier Subnetzen bestehen. Außerdem werden diese Bereiche nicht in Stein festgelegt. Sie können Ihre Umgebung aber auch nach Wunsch ansprechen. 

    * Allgemeine Konfiguration für das erste Subnetz.
        * Name: NAT
        * Adressraum: 10.0.0.0/24
        * Hinweis: an dieser Stelle befindet sich die primäre NIC für Hyper-V-Hosts. Diese wird verwendet, um die ausgehende NAT für die virtuellen Computer zu verarbeiten. Es ist das Gateway des Internets für Ihre virtuellen Computer.

    * Allgemeine Konfiguration für das zweite Subnetz.
        * Name: Hyper-V-LAN
        * Adressraum: 10.0.1.0/24
        * Hinweis: für den Hyper-v-Host wird eine zweite NIC verwendet, die zum Verarbeiten des Routings zwischen den nicht-Internetressourcen außerhalb des Hyper-v-Hosts verwendet wird.

    * Allgemeine Konfiguration für das dritte Subnetz.
        * Name: ghosted
        * Adressraum: 10.0.2.0/24
        * Hinweis: Hierbei handelt es sich um ein "Unverankertes" Subnetz. Der Adressraum wird von unseren virtuellen Computern genutzt und ist vorhanden, um Routen Ankündigungen zurück an den lokalen Standort zu verarbeiten. In diesem Subnetz werden tatsächlich keine VMs bereitgestellt.

    * Allgemeine Konfiguration für das vierte Subnetz.
        * Name: Azure-VMS
        * Adressraum: 10.0.3.0/24
        * Hinweis: Subnetz mit Azure-VMS.

1. Der Hyper-V-Host verfügt über die folgenden NIC-Konfigurationen.
    * Primäre NIC 
        * IP-Adresse: 10.0.0.4
        * Subnetzmaske: 255.255.255.0
        * Standard Gateway: 10.0.0.1
        * DNS: für DHCP konfiguriert
        * IP-Weiterleitung aktiviert: Nein

    * Sekundäre NIC
        * IP-Adresse: 10.0.1.4
        * Subnetzmaske: 255.255.255.0
        * Standard Gateway: leer
        * DNS: für DHCP konfiguriert
        * IP-Weiterleitung aktiviert: Ja

    * Von Hyper-V erstellte NIC für internen virtuellen Switch
        * IP-Adresse: 10.0.2.1
        * Subnetzmaske: 255.255.255.0
        * Standard Gateway: leer

3. Unsere Routing Tabelle verfügt über eine einzelne Regel.
    * Regel 1
        * Name: netsted VMS
        * Ziel: 10.0.2.0/24
        * Nächster Hop: virtuelles Gerät 10.0.1.4

## <a name="conclusion"></a>Schlussfolgerung

Sie sollten jetzt in der Lage sein, einen virtuellen Computer (sogar einen virtuellen 32-Bit-Computer) auf Ihrem Hyper-V-Host bereitzustellen und ihn von einem lokalen Standort und innerhalb von Azure aus zugänglich zu lassen.
