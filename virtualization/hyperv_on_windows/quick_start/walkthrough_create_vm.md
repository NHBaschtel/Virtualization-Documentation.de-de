---
title: &1585827490 Bereitstellen eines virtuellen Windows-Computers in Hyper-V unter Windows 10
description: Bereitstellen eines virtuellen Windows-Computers in Hyper-V unter Windows 10
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &489986826 windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 66723f33-b12c-49d1-82cf-71ba9d6087e9
---

# Bereitstellen eines virtuellen Windows-Computers in Hyper-V unter Windows 10

Sie können einen virtuellen Computer erstellen und ihm auf viele verschiedene Arten ein Betriebssystem bereitstellen, z. B. mithilfe der Windows-Bereitstellungsdienste, durch Anfügen einer vorbereiteten virtuellen Festplatte oder manuell mithilfe des Installationsmediums. In diesem Artikel wird erläutert, wie Sie einen virtuellen Computer erstellen und auf diesem mithilfe des Installationsmediums ein entsprechendes Betriebssystem bereitstellen.

Bevor Sie mit dieser Übung beginnen, benötigen Sie eine ISO-Datei für das Betriebssystem, das Sie bereitstellen möchten. Beziehen Sie bei Bedarf eine Evaluierungsversion von Windows 8.1 oder Windows 10 aus dem <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">TechNet-Evaluierungscenter</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

## Erstellen eines virtuellen Computers mit dem Hyper-V-Manager

Sie führen die folgenden Schritte aus, um einen virtuellen Computer manuell zu erstellen und ihm ein Betriebssystem bereitzustellen.

1. Klicken Sie im Hyper-V-Manager auf <g id="2" ctype="x-strong">Aktion</g> > <g id="4" ctype="x-strong">Neu</g> > <g id="6" ctype="x-strong">Virtueller Computer</g>, um den Assistenten für neue virtuelle Computer anzuzeigen.

2. Lesen Sie den Abschnitt „Vorbemerkungen“, und klicken Sie auf <g id="2" ctype="x-strong">Weiter</g>.

3. Geben Sie dem virtuellen Computer einen Namen.
> <g id="1" ctype="x-strong">Hinweis:</g> Dies ist der Name, den Hyper-V für den virtuellen Computer verwendet, nicht der Computername für das Gastbetriebssystem, das innerhalb des virtuellen Computers bereitgestellt wird.

4. Wählen Sie einen Pfad, in dem die Dateien des virtuellen Computers gespeichert werden, z. B. <g id="2" ctype="x-strong">c:\VM</g>. Sie können auch den Standardspeicherort übernehmen. Klicken Sie, sobald Sie fertig sind, auf <g id="2" ctype="x-strong">Weiter</g>.

  <g id="1" ctype="x-linkText"></g>

5. Wählen Sie eine Generation für den virtuellen Computer aus, und klicken Sie auf <g id="2" ctype="x-strong">Weiter</g>.

  Virtuelle Computer der Generation 2 wurden mit Windows Server 2012 R2 eingeführt und bieten ein vereinfachtes virtuelles Hardwaremodell und verschiedene Zusatzfunktionen. Auf virtuellen Computern der Generation 2 kann nur ein 64-Bit-Betriebssystem installiert werden. Weitere Informationen zu virtuellen Computern der 2. Generation finden Sie unter <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Generation 2 Virtual Machine Overview</g><g id="2CapsExtId3" ctype="x-title"></g></g> (Virtuelle Computer der 2. Generation [Übersicht]).

> Wenn der neue virtuelle Computer als 2. Generation konfiguriert ist und eine Linux-Distribution ausführt, muss der sichere Start deaktiviert werden. Weitere Informationen zum sicheren Start finden Sie unter <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Secure Boot (Sicherer Start)</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

6. Wählen Sie <g id="2" ctype="x-strong">2048</g> MB als Wert für <g id="4" ctype="x-strong">Arbeitsspeicher beim Start</g>, und lassen Sie <g id="6" ctype="x-strong">Dynamischen Arbeitsspeicher aktivieren</g> ausgewählt. Klicken Sie auf die Schaltfläche <g id="2" ctype="x-strong">Weiter</g>.

  Arbeitsspeicher wird von einem Hyper-V-Host und dem auf diesem ausgeführten virtuellen Computer gemeinsam genutzt. Die Anzahl der virtuellen Computer, die auf einem Host ausgeführt werden können, hängt zum Teil vom verfügbaren Arbeitsspeicher ab. Ein virtueller Computer kann auch für die Verwendung von dynamischem Arbeitsspeicher konfiguriert werden. Falls aktiviert, gibt die dynamische Arbeitsspeicherfunktion vom ausgeführten virtuellen Computer nicht genutzten Arbeitsspeicher frei. Dadurch können mehr virtuelle Computer auf dem Host ausgeführt werden. Weitere Informationen zu dynamischem Arbeitsspeicher finden Sie unter <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Hyper-V Dynamic Memory Overview (Dynamischer Hyper-V-Arbeitsspeicher – Übersicht)</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

7. Wählen Sie im Assistenten „Netzwerk konfigurieren“ einen virtuellen Switch für den virtuellen Computer, und klicken Sie auf <g id="2" ctype="x-strong">Weiter</g>. Weitere Informationen finden Sie unter <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Erstellen eines virtuellen Switches</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

8. Benennen Sie die virtuellen Festplatte, wählen Sie einen Speicherort, oder übernehmen Sie die Standardeinstellung, und geben Sie eine Größe an. Klicken Sie im Anschluss auf <g id="2" ctype="x-strong">Weiter</g>.

  Eine virtuelle Festplatte bietet dem virtuellen Computer Speicher, der mit einer physischen Festplatte vergleichbar ist. Eine virtuelle Festplatte ist erforderlich, damit Sie ein Betriebssystem auf dem virtuellen Computer installieren können.

  <g id="1" ctype="x-linkText"></g>

9. Wählen Sie im Assistenten „Installationsoptionen“ <g id="2" ctype="x-strong">Betriebssystem von startfähiger Imagedatei installieren</g> aus, und wählen Sie dann eine ISO-Datei des Betriebssystems aus. Klicken Sie danach auf <g id="2" ctype="x-strong">Weiter</g>.

  Beim Erstellen eines virtuellen Computers können Sie verschiedene Installationsoptionen für das Betriebssystem konfigurieren. Drei Optionen sind verfügbar:

  - <g id="1" ctype="x-strong">Betriebssystem zu einem späteren Zeitpunkt installieren</g>: Bei Wahl dieser Option erfolgen keine weiteren Änderungen am virtuellen Computer.

  - <g id="1" ctype="x-strong">Betriebssystem von startfähiger Imagedatei installieren</g>: Dies ist vergleichbar mit dem Einlegen einer CD in das physische CD-ROM-Laufwerk eines physischen Computers. Um diese Option zu konfigurieren, wählen Sie ein ISO-Image aus. Dieses Image wird auf dem virtuellen CD-ROM-Laufwerk des virtuellen Computers bereitgestellt. Die Startreihenfolge des virtuellen Computers wird so geändert, dass der Start zuerst vom CD-ROM-Laufwerk erfolgt.

  - <g id="1" ctype="x-strong">Betriebssystem von einem netzwerkbasierten Installationsserver installieren</g>: Diese Option ist erst verfügbar, nachdem Sie den virtuellen Computer mit einem Netzwerkswitch verbunden haben. Bei dieser Konfiguration wird versucht, den virtuellen Computer über das Netzwerk zu starten.

10. Überprüfen Sie die Details des virtuellen Computers, und klicken Sie zum Abschließen der Erstellung auf <g id="2" ctype="x-strong">Fertig stellen</g>.

## Erstellen eines virtuellen Computers mit PowerShell

1. Öffnen Sie die PowerShell ISE als Administrator.

2. Führen Sie das folgende Skript aus:

  ```powershell
  # Set VM Name, Switch Name, and Installation Media Path.
  $VMName = 'TESTVM'
  $Switch = 'External VM Switch'
  $InstallMedia = 'C:\Users\Administrator\Desktop\en_windows_10_enterprise_x64_dvd_6851151.iso'

  # Create New Virtual Machine
  New-VM -Name $VMName -MemoryStartupBytes 2147483648 -Generation 2 -NewVHDPath "D:\Virtual Machines\$VMName\$VMName.vhdx" -NewVHDSizeBytes 53687091200 -Path "D:\Virtual Machines\$VMName" -SwitchName $Switch

  # Add DVD Drive to Virtual Machine
  Add-VMScsiController -VMName $VMName
  Add-VMDvdDrive -VMName $VMName -ControllerNumber 1 -ControllerLocation 0 -Path $InstallMedia

  # Mount Installation Media
  $DVDDrive = Get-VMDvdDrive -VMName $VMName

  # Configure Virtual Machine to Boot from DVD
  Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive
  ```

## Abschließen der Betriebssystembereitstellung

Um die Erstellung des virtuellen Computers abzuschließen, müssen Sie den virtuellen Computer starten und die Installation des Betriebssystems durchlaufen.

1. Doppelklicken Sie im Hyper-V-Manager auf den virtuellen Computer. Dadurch wird das Tool VMConnect gestartet.

2. Klicken Sie in VMConnect auf die grüne Startschaltfläche. Dies entspricht dem Drücken der Einschalttaste eines physischen Computers. Sie werden möglicherweise aufgefordert, eine beliebige Taste zu drücken, um von CD oder DVD zu starten. Befolgen Sie diese Aufforderung.
> <g id="1" ctype="x-strong">Hinweis:</g> Sie müssen möglicherweise in das VMConnect-Fenster klicken, um sicherzustellen, dass Ihre Tastatureingaben an den virtuellen Computer gesendet werden.

3. Der virtuelle Computer wird mit der Setup-Phase gestartet. Durchlaufen Sie die Installation wie auf einem physischen Computer.

  <g id="1" ctype="x-linkText"></g>

> <g id="1" ctype="x-strong">Hinweis:</g> Um Windows auf einem virtuellen Computer ausführen zu können, benötigen Sie eine separate Lizenz, es sei denn, Sie führen eine Volumenlizenzversion von Windows aus. Das Betriebssystem des virtuellen Computers ist unabhängig vom Betriebssystem des Hosts.

## Nächster Schritt: Arbeiten mit PowerShell und Hyper-V

<g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Hyper-V und Windows PowerShell</g><g id="1CapsExtId3" ctype="x-title"></g></g>





<!--HONumber=May16_HO2-->


