# <a name="linux-containers"></a>Linux-Container

Dieses Feature verwendet [Hyper-V-Isolierung](../manage-containers/hyperv-container.md), um für die Containerunterstützung einen Linux-Kernel mit ausreichendem Betriebssystem auszuführen. Die Änderungen an Windows und Hyper-V dafür begannen mit _Windows10 Fall Creators Update_ und _Windows Server, Version 1709_. Dies zusammenzuführen erforderte aber auch eine Zusammenarbeit mit dem Open Source-Projekt [Moby](https://www.github.com/moby/moby), auf dem die Docker-Technologie aufsetzt, ebenso wie der Linux-Kernel. 

![Videovorschau für Linux-Container](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

Um dies auszuprobieren, benötigen Sie Folgendes:

- Windows10 oder Windows Server Insider Preview Build 16267 oder höher
- Ein Build des Docker-Daemon basierend auf dem Master Branch Moby, ausgeführt mit dem Flag `--experimental`
- Kompatibles Linux-Image Ihrer Wahl

Für diese Vorschau gibt es "Erste Schritte"-Handbücher:

- [Docker Enterprise Edition Preview](https://blog.docker.com/2017/09/docker-windows-server-1709/) umfasst ein LinuxKit-System und eine Vorschau von Docker EE, das Linux-Container ausführen kann. Weitere Hintergrundinformationen finden Sie unter [Preview Linux Containers on Windows using LinuxKit](https://go.microsoft.com/fwlink/?linkid=857061).
- [Ausführen von Ubuntu-Containern mit Hyper-V-Isolierung unter Windows10 und Windows Server](https://go.microsoft.com/fwlink/?linkid=857067)


## <a name="work-in-progress"></a>In Bearbeitung

Fortschritte beim Moby-Projekt können verfolgt werden auf [GitHub](https://github.com/moby/moby/issues/33850).


### <a name="known-app-issues"></a>Bekannte App-Probleme

Diese Anwendungen erfordern die Volumezuordnung, die einigen Einschränkungen unterliegt (siehe [Binden von Bereitstellungen](#Bind-mounts)). Sie werden nicht gestartet oder ordnungsgemäß ausgeführt.

- Mysql
- Postgress
- WordPress
- Jenkins
- Mariadb
- RabbitMQ


### <a name="bind-mounts"></a>Binden von Bereitstellungen

Beim Binden der Bereitstellung von Volumes mit `docker run -v ...` werden die Dateien im Windows NTFS-Dateisystem gespeichert, sodass für POSIX-Vorgänge einige Übersetzung erforderlich ist. Einige Dateisystemvorgänge werden derzeit teilweise oder gar nicht implementiert, was bei einigen Anwendungen zu Inkompatibilitäten führen kann.

Folgende Vorgänge funktionieren momentan nicht für gebunden bereitgestellte Volumes:

- MkNod
- XAttrWalk
- XAttrCreate
- Lock
- Getlock
- Auth
- Flush
- INotify

Es gibt auch einige Vorgänge, die nicht vollständig implementiert werden:

- GetAttr – die Nlink-Zahl wird immer als 2 gemeldet
- Open – nur ReadWrite-, WriteOnly- und ReadOnly-Flags werden implementiert