# [Container in der Windows-Dokumentation](index.md) 

# Übersicht
## [Informationen zu Windows-Containern](about/index.md)
## [Container vs. VMS](about/containers-vs-vm.md)
## [Systemanforderungen](deploy-containers/system-requirements.md)
## [Häufig gestellte Fragen](about/faq.md)

# Erste Schritte
## [Einrichten Ihrer Umgebung](quick-start/set-up-environment.md)
## [Ausführen des ersten Containers](quick-start/run-your-first-container.md)
## [Containerisieren einer Beispiel-App](quick-start/building-sample-app.md)

# Lernprogramme
## Erstellen eines Windows-Containers
### [Schreiben eines Dockerfile](manage-docker/manage-windows-dockerfile.md)
### [Optimieren einer Dockerfile](manage-docker/optimize-windows-dockerfile.md)
## Ausführen auf Azure Kubernetes-Dienst
### [Erstellen eines Windows-Container Clusters auf AKS](/azure/aks/windows-container-cli)
### [Aktuelle Einschränkungen](/azure/aks/windows-node-limitations)
## Auf Service Fabric ausführen
### [Bereitstellen Ihres ersten Containers](/azure/service-fabric/service-fabric-quickstart-containers)
### [Bereitstellen einer .NET-Anwendung in einem Windows-Container](/azure/service-fabric/service-fabric-host-app-in-a-container)
## Ausführen des Azure-App-Diensts
### [Azure App-Dienst-Schnellstart](/azure/app-service/app-service-web-get-started-windows-container)
### [Migrieren einer ASP.net-App mit Windows-Containern und Azure App-Dienst](/azure/app-service/app-service-web-tutorial-windows-containers-custom-fonts)
## Linux-Container unter Windows
### [Übersicht](deploy-containers/linux-containers.md)
### [Ausführen des ersten LCOW-Containers](quick-start/quick-start-windows-10-linux.md)
## Verwenden von Containern mit dem Windows-Insider-Programm
### [Übersicht](deploy-containers/insider-overview.md)

# Konzepte
## Windows-Container-Grundlagen
### [Container-Basisbilder](manage-containers/container-base-images.md)
### [Isolationsmodi](manage-containers/hyperv-container.md)
### [Versionskompatibilität](deploy-containers/version-compatibility.md)
### [Ressourcen Steuerelemente](manage-containers/resource-controls.md)
## Docker
### [Docker-Modul unter Windows](manage-docker/configure-docker-daemon.md)
### [Remote Verwaltung eines Windows-andockbaren Hosts](management/manage_remotehost.md)
## Container-Orchestrierung
### [Übersicht](about/overview-container-orchestrators.md)
### Kubernetes unter Windows
#### [Kubernetes unter Windows](kubernetes/getting-started-kubernetes-windows.md)
#### [Erstellen eines Kubernetes-Masters](kubernetes/creating-a-linux-master.md)
#### [Auswählen einer Netzwerklösung](kubernetes/network-topologies.md)
#### [Teilnehmen an Windows-Mitarbeitern](kubernetes/joining-windows-workers.md)
#### [Teilnehmen an Linux-Mitarbeitern](kubernetes/joining-linux-workers.md)
#### [Bereitstellen von Kubernetes-Ressourcen](kubernetes/deploying-resources.md)
#### [Problembehandlung](kubernetes/common-problems.md)
#### [Windows-Dienste auf Kubernetes](kubernetes/kube-windows-services.md)
#### [Kompilieren von Kubernetes-Binärdateien](kubernetes/compiling-kubernetes-binaries.md)
### Dienststruktur
#### [Dienststruktur und Container](/azure/service-fabric/service-fabric-containers-overview)
#### [Ressourcen-Governance](/azure/service-fabric/service-fabric-resource-governance)
### Andocker-Schwarm
#### [Schwarm Modus](manage-containers/swarm-mode.md)
## Workloads
### Gruppenverwaltete Dienstkonten
#### [Erstellen Sie ein gMSA.](manage-containers/manage-serviceaccounts.md)
#### [Konfigurieren der APP für die Verwendung eines gMSA](manage-containers/gmsa-configure-app.md)
#### [Ausführen eines Containers mit einem gMSA](manage-containers/gmsa-run-container.md)
#### [Orchestrieren von Containern mit einem gMSA](manage-containers/gmsa-orchestrate-containers.md)
#### [Problembehandlung bei gMSAs](manage-containers/gmsa-troubleshooting.md)
### [Druckerdienste](deploy-containers/print-spooler.md)
## Networking
### [Übersicht](container-networking/architecture.md)
### [Netzwerktopologien und-Treiber](container-networking/network-drivers-topologies.md)
### [Netzwerkisolierung und-Sicherheit](container-networking/network-isolation-security.md)
### [Konfigurieren von erweiterten Netzwerkoptionen](container-networking/advanced.md)
## Speicher
### [Übersicht](manage-containers/container-storage.md)
### [Persistenter Speicher](manage-containers/persistent-storage.md)
## Geräte
### [Hardware Geräte](deploy-containers/hardware-devices-in-containers.md)
### [GPU-Beschleunigung](deploy-containers/gpu-acceleration.md)

# Referenz
## [Basis-Lebenszyklus von Bildservice](deploy-containers/base-image-lifecycle.md)
## [Virenschutz Optimierung](https://docs.microsoft.com/windows-hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)
## [Tools für die Container Plattform](deploy-containers/containerd.md)
## [EULA für Container-Betriebssystem Bilder](Images_EULA.md)

# Ressourcen
## [Container Beispiele](samples.md)
## [Problembehandlung](troubleshooting.md)
## [Container Forum](https://social.msdn.microsoft.com/Forums/home?forum=windowscontainers)
## [Community-Videos und-Blogs](communitylinks.md)