---
title: Container im Vergleich zu virtuellen Computern
description: In diesem Thema werden einige der wichtigsten Gemeinsamkeiten und Unterschiede zwischen Containern und virtuellen Computern erläutert. Container und virtuelle Computer sind jeweils einsatzfähig – tatsächlich verwenden viele bereit Stellungen von Containern virtuelle Computer als Host Betriebssystem, anstatt direkt auf der Hardware ausgeführt zu werden, insbesondere bei der Ausführung von Containern in der Cloud.
keywords: docker, Container, VMS, virtuelle Computer
author: jasongerend
ms.author: jgerend
ms.date: 10/21/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 63150dfde007ec942446387064ad59f05b0aaa43
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910820"
---
# <a name="containers-vs-virtual-machines"></a>Container im Vergleich zu virtuellen Computern

In diesem Thema werden einige der wichtigsten Gemeinsamkeiten und Unterschiede zwischen Containern und virtuellen Computern (VMS) erläutert. Container und VMS sind jeweils einsatzfähig – tatsächlich verwenden viele bereit Stellungen von Containern VMS als Host Betriebssystem, anstatt direkt auf der Hardware ausgeführt zu werden, insbesondere bei der Ausführung von Containern in der Cloud.

Einen Überblick über Container finden Sie unter [Windows und Container](index.md).

## <a name="container-architecture"></a>Container Architektur

Ein Container ist ein isoliertes, schlankes Silo zum Ausführen einer Anwendung auf dem Host Betriebssystem. Container bauen auf dem Kernel des Host Betriebssystems auf (was Sie sich als das verschüttete betreiben des Betriebssystems vorstellen können) und enthalten nur apps und einige Lightweight-APIs und-Dienste für Betriebssysteme, die im Benutzermodus ausgeführt werden, wie in diesem Diagramm dargestellt.

![Architektur Diagramm, das zeigt, wie Container oberhalb des Kernels ausgeführt werden](media/container-diagram.svg)

## <a name="virtual-machine-architecture"></a>Architektur des virtuellen Computers

Im Gegensatz zu Containern führen virtuelle Computer ein ganzes Betriebssystem aus – einschließlich eines eigenen Kernels – wie in diesem Diagramm dargestellt.

![Architektur Diagramm, das zeigt, wie VMS ein ganzes Betriebssystem neben dem Host Betriebssystem ausführen](media/virtual-machine-diagram.svg)

## <a name="containers-vs-virtual-machines"></a>Container im Vergleich zu virtuellen Computern

In der folgenden Tabelle sind einige Gemeinsamkeiten und Unterschiede dieser ergänzenden Technologien aufgeführt.

|                 | Virtueller Computer  | Container  |
| --------------  | ---------------- | ---------- |
| Isolierung       | Bietet eine umfassende Isolierung vom Host Betriebssystem und anderen virtuellen Computern. Dies ist hilfreich, wenn eine starke Sicherheitsgrenze wichtig ist, z. b. das Hosting von apps von konkurrierenden Unternehmen auf demselben Server oder Cluster. | Stellt in der Regel eine vereinfachte Isolation vom Host und anderen Containern bereit, bietet jedoch nicht so stark eine Sicherheitsgrenze wie eine VM. (Sie können die Sicherheit erhöhen, indem Sie den [Hyper-V-Isolations Modus](../manage-containers/hyperv-container.md) verwenden, um jeden Container in einer Lightweight-VM zu isolieren). |
| Betriebssystem | Führt ein ganzes Betriebssystem einschließlich des Kernels aus und erfordert somit mehr Systemressourcen (CPU, Arbeitsspeicher und Speicher). | Führt den benutzermodusbereich eines Betriebssystems aus und kann so angepasst werden, dass nur die für Ihre APP benötigten Dienste verwendet werden, wobei weniger Systemressourcen verwendet werden. |
| Gast Kompatibilität | Führt fast alle Betriebssysteme innerhalb des virtuellen Computers aus. | Wird unter der [gleichen Betriebssystemversion ausgeführt wie der Host](../deploy-containers/version-compatibility.md) (Hyper-V-Isolation ermöglicht Ihnen das Ausführen früherer Versionen desselben Betriebssystems in einer Lightweight-VM-Umgebung).
| Bereitstellung     | Stellen Sie einzelne VMS mithilfe des Windows Admin Centers oder Hyper-V-Managers bereit. Stellen Sie mehrere VMS mithilfe von PowerShell oder System Center Virtual Machine Manager bereit. | Bereitstellen einzelner Container mithilfe von Docker über die Befehlszeile Stellen Sie mehrere Container mithilfe eines Orchestrator wie Azure Kubernetes Service bereit. |
| Betriebssystemupdates und-Upgrades | Herunterladen und Installieren von Betriebssystemupdates auf den einzelnen virtuellen Computern. Zum Installieren einer neuen Betriebssystemversion muss ein Upgrade durchgeführt werden. Dies kann sehr zeitaufwändig sein, insbesondere, wenn Sie über zahlreiche VMS verfügen... | Das Aktualisieren oder Aktualisieren der Betriebssystemdateien in einem Container ist identisch: <br><ol><li>Bearbeiten Sie die Builddatei Ihres Container Images (als dockerfile bezeichnet), um auf die neueste Version des Windows-Basis Images zu verweisen. </li><li>Erstellen Sie das Container Image mit diesem neuen Basis Image neu.</li><li>Übersetzen Sie das Container Image per Push an Ihre Container Registrierung.</li> <li>Erneutes Bereitstellen mithilfe eines Orchestrator.<br>Der Orchestrator bietet eine leistungsstarke Automatisierung für die Skalierung. Weitere Informationen finden Sie unter [Tutorial: Aktualisieren einer Anwendung in Azure Kubernetes Service](https://docs.microsoft.com/azure/aks/tutorial-kubernetes-app-update).</li></ol> |
| Dauerhafte Speicherung | Verwenden Sie eine virtuelle Festplatte (VHD) für den lokalen Speicher eines einzelnen virtuellen Computers oder eine SMB-Dateifreigabe für den Speicher, der von mehreren Servern gemeinsam genutzt wird. | Verwenden Sie Azure-Datenträger für den lokalen Speicher für einen einzelnen Knoten oder Azure Files (SMB-Freigaben) für Speicher, der von mehreren Knoten oder Servern gemeinsam genutzt wird. |
| Lastenausgleich | Beim Lastenausgleich virtueller Computer werden virtuelle Computer auf andere Server in einem Failovercluster verschoben. | Container selbst werden nicht verschoben. Stattdessen kann ein Orchestrator Container auf Cluster Knoten automatisch starten oder beenden, um Änderungen bei der Auslastung und Verfügbarkeit zu verwalten. |
| Fehlertoleranz | VMs können ein Failover zu einem anderen Server in einem Cluster ausführen, wobei das Betriebssystem des virtuellen Computers auf dem neuen Server neu gestartet wird.  | Wenn ein Cluster Knoten ausfällt, werden alle darauf laufenden Container schnell von Orchestrator auf einem anderen Cluster Knoten neu erstellt. |
| -Netzwerk     | Verwendet virtuelle Netzwerkadapter. | Verwendet eine isolierte Ansicht eines virtuellen Netzwerkadapters und bietet so eine geringfügige Virtualisierung – die Firewall des Hosts wird für Container freigegeben – bei Verwendung von weniger Ressourcen. Weitere Informationen finden Sie unter [Windows-Container Netzwerke](../container-networking/architecture.md). |