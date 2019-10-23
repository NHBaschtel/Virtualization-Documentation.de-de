---
title: Container vs. virtuelle Maschinen
description: In diesem Thema werden einige der wichtigsten Ähnlichkeiten und Unterschiede zwischen Containern und virtuellen Computern erläutert, und wenn Sie die einzelnen verwenden möchten. Container und virtuelle Maschinen haben jeweils ihre Verwendung – in der Tat verwenden viele Bereitstellungen von Containern virtuelle Maschinen als Hostbetriebssystem, anstatt direkt auf der Hardware zu laufen, insbesondere beim Ausführen von Containern in der Cloud.
keywords: docker, Container, VMS, virtuelle Maschinen
author: jasongerend
ms.author: jgerend
ms.date: 10/21/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 63150dfde007ec942446387064ad59f05b0aaa43
ms.sourcegitcommit: d0411b05d1ef7328a770766b84fd0743f9d9c237
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 10/23/2019
ms.locfileid: "10254399"
---
# <a name="containers-vs-virtual-machines"></a>Container vs. virtuelle Maschinen

In diesem Thema werden einige der wichtigsten Ähnlichkeiten und Unterschiede zwischen Containern und virtuellen Computern (VMS) erläutert, und wenn Sie die einzelnen verwenden möchten. Container und VMS haben jeweils ihre Verwendung – in der Tat verwenden viele Container Bereitstellungen VMS als Hostbetriebssystem, anstatt direkt auf der Hardware zu laufen, insbesondere beim Ausführen von Containern in der Cloud.

Eine Übersicht über Container finden Sie unter [Windows und Container](index.md).

## <a name="container-architecture"></a>Container Architektur

Ein Container ist ein isoliertes, leichtes Silo zum Ausführen einer Anwendung auf dem Hostbetriebssystem. Container bauen auf dem Kernel des Hostbetriebssystems auf (was als Vergrabenes Sanitär des Betriebssystems zu sehen ist) und enthalten nur apps und einige einfache Betriebssystem-APIs und-Dienste, die im Benutzermodus ausgeführt werden, wie in diesem Diagramm dargestellt.

![Architektonisches Diagramm, das zeigt, wie Container auf dem Kernel ausgeführt werden](media/container-diagram.svg)

## <a name="virtual-machine-architecture"></a>Architektur virtueller Maschinen

Im Gegensatz zu Containern führen VMS ein vollständiges Betriebssystem – einschließlich des eigenen Kernels – aus, wie in diesem Diagramm dargestellt.

![Architekturdiagramm, das zeigt, wie VMS neben dem Hostbetriebssystem ein vollständiges Betriebssystem ausführen](media/virtual-machine-diagram.svg)

## <a name="containers-vs-virtual-machines"></a>Container vs. virtuelle Maschinen

Die folgende Tabelle zeigt einige der Ähnlichkeiten und Unterschiede dieser komplementär Technologien.

|                 | Virtueller Computer  | Container  |
| --------------  | ---------------- | ---------- |
| Isolation       | Bietet vollständige Isolierung vom Hostbetriebssystem und anderen VMS. Dies ist hilfreich, wenn eine starke Sicherheitsgrenze wichtig ist, beispielsweise das Hosten von apps von konkurrierenden Unternehmen auf demselben Server oder Cluster. | Stellt in der Regel eine einfache Isolierung vom Host und anderen Containern bereit, bietet jedoch keine so starke Sicherheitsgrenze wie eine VM. (Sie können die Sicherheit erhöhen, indem Sie den [Hyper-V-Isolationsmodus](../manage-containers/hyperv-container.md) verwenden, um jeden Container in einem einfachen virtuellen Computer zu isolieren). |
| Betriebssystem | Führt ein vollständiges Betriebssystem einschließlich des Kernels aus, wodurch mehr Systemressourcen (CPU, Arbeitsspeicher und Speicher) erforderlich sind. | Führt den Benutzermodus-Teil eines Betriebssystems aus und kann so angepasst werden, dass nur die benötigten Dienste für Ihre APP mit weniger Systemressourcen enthalten sind. |
| Gast Kompatibilität | Führt fast jedes Betriebssystem innerhalb des virtuellen Computers aus | Wird unter der [gleichen Betriebssystemversion wie der Host](../deploy-containers/version-compatibility.md) ausgeführt (Hyper-V-Isolierung ermöglicht Ihnen, frühere Versionen desselben Betriebssystems in einer einfachen VM-Umgebung auszuführen)
| Bereitstellung     | Bereitstellen einzelner VMS mithilfe des Windows Admin Centers oder des Hyper-V-Managers Bereitstellen mehrerer VMS mithilfe von PowerShell oder System Center Virtual Machine Manager | Bereitstellen einzelner Container mithilfe von Docker über die Befehlszeile Bereitstellen mehrerer Container mithilfe eines Orchestrators wie Azure Kubernetes-Dienst. |
| Betriebssystemupdates und-Upgrades | Laden Sie die Betriebssystemupdates auf jeder VM herunter, und installieren Sie Sie. Für die Installation einer neuen Betriebssystemversion ist eine Aktualisierung erforderlich, oder es wird häufig nur eine völlig neue VM erstellt. Dies kann zeitaufwendig sein, besonders, wenn viele VMS vorhanden sind... | Das Aktualisieren oder Aktualisieren der Betriebssystemdateien in einem Container ist identisch: <br><ol><li>Bearbeiten Sie die Builddatei Ihres Container Bilds (so genannte Dockerfile), um auf die neueste Version des Windows-Basis Bilds zu verweisen. </li><li>Erstellen Sie Ihr Container Bild mit diesem neuen Basis Bild neu.</li><li>Schieben Sie das Container Bild in Ihre Container Registrierung.</li> <li>Erneute Bereitstellung mithilfe eines Orchestrators.<br>Der Orchestrator bietet eine leistungsfähige Automatisierung, um dies im Maßstab zu tun. Ausführliche Informationen finden Sie unter [Lernprogramm: Aktualisieren einer Anwendung im Azure Kubernetes-Dienst](https://docs.microsoft.com/azure/aks/tutorial-kubernetes-app-update).</li></ol> |
| Persistenter Speicher | Verwenden einer virtuellen Festplatte (VHD) für den lokalen Speicher für einen einzelnen virtuellen Computer oder eine SMB-Dateifreigabe für von mehreren Servern freigegebene Speichermedien | Verwenden Sie Azure-Datenträger für den lokalen Speicher für einen einzelnen Knoten oder Azure-Dateien (SMB-Freigaben) für Speicher, der von mehreren Knoten oder Servern freigegeben wird. |
| Lastenausgleich | Beim Lastenausgleich durch den virtuellen Computer werden die ausgeführten VMs auf andere Server in einem Failovercluster verschoben. | Container selbst bewegen sich nicht; Stattdessen kann ein Orchestrator Container auf Clusterknoten automatisch starten oder beenden, um Änderungen bei der Auslastung und Verfügbarkeit zu verwalten. |
| Fehlertoleranz | VMs können einen Failover zu einem anderen Server in einem Cluster durchführen, wobei das Betriebssystem des virtuellen Computers auf dem neuen Server neu gestartet wird.  | Wenn ein Clusterknoten fehlschlägt, werden alle darauf ausgeführten Container schnell vom Orchestrator auf einem anderen Clusterknoten neu erstellt. |
| Networking     | Verwendet virtuelle Netzwerkadapter. | Verwendet eine isolierte Ansicht eines virtuellen Netzwerkadapters, die etwas weniger Virtualisierung bietet – die Firewall des Hosts wird für Container freigegeben, während weniger Ressourcen verwendet werden. Weitere Informationen finden Sie unter [Windows-Container Netzwerke](../container-networking/architecture.md). |