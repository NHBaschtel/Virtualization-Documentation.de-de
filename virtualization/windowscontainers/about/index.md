---
title: About Windows Containers
description: Learn about Windows containers.
keywords: docker, containers
author: taylorb-microsoft
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 8e273856-3620-4e58-9d1a-d1e06550448
ms.openlocfilehash: 2be7a06c7b7b154e392c30981cdf954d2d1b796e
ms.sourcegitcommit: 8e193d8c274a549aef497f16dcdb00d7855e9fa7
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 08/02/2017
---
# Windows-Container

## Was sind Container?

Container sind eine Möglichkeit, eine Anwendung isoliert einzupacken. Die Anwendung, die sich in einem Container befindet, weiß nichts von anderen Anwendungen oder Prozessen, die außerhalb des Containers existieren. Alles, was für die erfolgreiche Ausführung der Anwendung erforderlich ist, befindet sich ebenfalls in diesem Container.  Und wenn der Container verschoben wird, hat das keine Auswirkungen auf die Anwendung, da alles enthalten ist, damit sie ausgeführt werden kann.

Stellen Sie sich eine Küche vor. Wir verpacken alle Geräte, Möbelstücke, sämtliches Geschirr, das Spülmittel und die Geschirrtücher. Das ist unser Container.

<center style="margin: 25px">![](media/box1.png)</center>

Diesen Container können wir in jede beliebige Wohnung stellen, die Küche ist immer die gleiche. Wir brauchen lediglich einen Strom- und Wasseranschluss und können sofort etwas kochen (da alle Geräte, die wir brauchen, vorhanden sind).

<center style="margin: 25px">![](media/apartment.png)</center>

Mit Containern verhält es sich in gewisser Weise wie mit dieser Küche. Die Räume können unterschiedlich oder auch gleich sein. Wichtig ist, dass der Container alles umfasst, was benötigt wird.

Sehen Sie sich eine kurze Übersicht an: [Windows-based containers: Modern app development with enterprise-grade control](https://youtu.be/Ryx3o0rD5lY) (Windows-basierte Container: Moderne App-Entwicklung mit Steuerung auf Unternehmensniveau).

## Grundlegendes zu Containern

Container sind eine isolierte, von Ressourcen gesteuerte und portierbare Laufzeitumgebung, die auf einem Hostcomputer oder virtuellen Computer ausgeführt wird. Anwendungen oder Prozesse, die in Containern ausgeführt werden, sind mit allen erforderlichen Abhängigkeiten und Konfigurationsdateien verpackt, so als ob außerhalb des Containers keine anderen Prozesse ausgeführt werden würden.

Der Host des Containers stellt einen Satz von Ressourcen für den Container bereit, und der Container verwendet nur diese Ressourcen. Für den Container existieren keine anderen Ressourcen als die angegebenen. Er kommt somit mit keinen Ressourcen in Berührung, die für einen anderen Container bereitgestellt wurden.

Die folgenden grundlegenden Konzepte können nützlich sein, wenn Sie beginnen, Windows-Container zu erstellen und damit zu arbeiten.

**Container Host:** Physical or Virtual computer system configured with the Windows Container feature. Auf dem Containerhost werden ein oder mehrere Windows-Container ausgeführt.

**Containerimage:** Wenn z.B. durch eine Softwareinstallation das Dateisystem oder die Registrierung eines Containers geändert wird, werden diese Änderungen in einem Sandkasten erfasst. In vielen Fällen ist es wünschenswert, diesen Zustand zu erfassen, damit neue Container erstellt werden können, die diese Änderungen übernehmen. That’s what an image is – once the container has stopped you can either discard that sandbox or you can convert it into a new container image. For example, let’s imagine that you have deployed a container from the Windows Server Core OS image. You then install MySQL into this container. Creating a new image from this container would act as a deployable version of the container. This image would only contain the changes made (MySQL), however would work as a layer on top of the Container OS Image.

**Sandbox:** Once a container has been started, all write actions such as file system modifications, registry modifications or software installations are captured in this ‘sandbox’ layer.

**Container OS Image:** Containers are deployed from images. The container OS image is the first layer in potentially many image layers that make up a container. Das Image stellt die Betriebssystemumgebung bereit. Das Betriebssystemimage eines Containers ist unveränderlich. Das heißt, es bleibt immer gleich.

**Containerrepository:** Bei jeder Erstellung eines Containerimages werden das Containerimage und dessen Abhängigkeiten in einem lokalen Repository gespeichert. Diese Images können auf dem Containerhost mehrfach wiederverwendet werden. Die Containerimages können auch in einer öffentlichen oder privaten Registrierung, wie z. B. DockerHub, gespeichert werden, damit sie in vielen verschiedenen Containerhosts verwendet werden können.

<center>![](media/containerfund.png)</center>

Wenn Sie sich mit virtuellen Computern auskennen, werden Ihnen die Ähnlichkeiten mit Containern auffallen. A container runs an operating system, has a file system and can be accessed over a network just as if it was a physical or virtual computer system. Was Technologie und Konzepte anbelangt, unterscheiden sich Container jedoch sehr von virtuellen Computern.

Mark Russinovich, Experte für Microsoft Azure, erklärt diese Unterschiede in einem [großartigen Blogbeitrag](https://azure.microsoft.com/en-us/blog/containers-docker-windows-and-trends/).

## Windows-Containertypen

Windows Containers include two different container types, or runtimes.

**Windows Server Containers** – provide application isolation through process and namespace isolation technology. A Windows Server Container shares a kernel with the container host and all containers running on the host. Diese Container bieten keine schädlichen Sicherheitsgrenzen und sollten nicht zum Isolieren von nicht vertrauenswürdigen Codes verwendet werden. Aufgrund des gemeinsam genutzten Kernelspeichers benötigen diese Container die gleiche Kernelversion und -konfiguration.

**Hyper-V-Isolierung**: Erweitert die von Windows Server-Containern bereitgestellte Isolierung, indem jeder Container in einem hochgradig optimierten virtuellen Computer ausgeführt wird. In this configuration, the kernel of the container host is not shared with other containers on the same host. Diese Container wurden für schädliches mandantenfähiges Hosting mit der gleichen Sicherheitsgarantie wie der eines virtuellen Computers entwickelt. Da diese Container den Kernel nicht mit dem Host oder mit anderen Containern auf dem Host teilen, können sie Kernel mit verschiedenen Versionen und Konfigurationen (in unterstützten Versionen) ausführen – z.B. wenn alle Windows-Container unter Windows10 Hyper-V-Isolierung verwenden, um die Version und Konfiguration des Windows Server-Kernels zu nutzen.

Die Ausführung eines Containers unter Windows mit oder ohne Hyper-V-Isolierung ist eine Entscheidung während der Laufzeit. Sie können den Container auch zunächst mit Hyper-V-Isolierung erstellen und ihn später zur Laufzeit stattdessen als Windows Server-Container ausführen.

## Was ist Docker?

Informationen zu Containern erwähnen zwangsläufig auch Docker. Docker ist der Behälter, in dem die Containerimages verpackt und ausgeliefert werden. Dieser automatisierte Prozess erstellt Images (im Prinzip Vorlagen), die überall – lokal, in der Cloud oder auf einem persönlichen Computer – als Container ausgeführt werden können.

<center>![](media/docker.png)</center>

Ein Windows Server-Container kann wie jeder andere Container mit [Docker](https://www.docker.com) verwaltet werden.

## Container für Entwickler ##

From a developer’s desktop to a testing machine to a set of production machines, a Docker image can be created that will deploy identically across any environment in seconds. This story has created a massive and growing ecosystem of applications packaged in Docker containers, with DockerHub, the public containerized-application registry that Docker maintains, currently publishing more than 180,000 applications in the public community repository.

When you containerize an app, only the app and the components needed to run the app are combined into an "image". Containers are then created from this image as you need them. You can also use an image as a baseline to create another image, making image creation even faster. Multiple containers can share the same image, which means containers start very quickly and use fewer resources. For example, you can use containers to spin up light-weight and portable app components – or ‘micro-services’ – for distributed apps and quickly scale each service separately.

Because the container has everything it needs to run your application, they are very portable and can run on any machine that is running Windows Server 2016. You can create and test containers locally, then deploy that same container image to your company's private cloud, public cloud or service provider. The natural agility of Containers supports modern app development patterns in large scale, virtualized and cloud environments.

With containers, developers can build an app in any language. These apps are completely portable and can run anywhere - laptop, desktop, server, private cloud, public cloud or service provider - without any code changes.  

Containers helps developers build and ship higher-quality applications, faster.

## Containers for IT Professionals ##

IT Professionals can use containers to provide standardized environments for their development, QA, and production teams. They no longer have to worry about complex installation and configuration steps. By using containers, systems administrators abstract away differences in OS installations and underlying infrastructure.

Containers help admins create an infrastructure that is simpler to update and maintain.

## Video Overview

<iframe src="https://channel9.msdn.com/Blogs/containers/Containers-101-with-Microsoft-and-Docker/player" width="800" height="450" allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>

## Windows Server-Container testen

Möchten Sie die großartigen Möglichkeiten von Containern nutzen? Im Folgenden finden Sie Informationen zum Bereitstellen Ihres ersten Containers: <br/>
Benutzer von Windows Server: [Schnellstartanleitung für Windows Server](../quick-start/quick-start-windows-server.md) <br/>
Benutzer von Windows 10: [Schnellstartanleitung für Windows 10](../quick-start/quick-start-windows-10.md)

