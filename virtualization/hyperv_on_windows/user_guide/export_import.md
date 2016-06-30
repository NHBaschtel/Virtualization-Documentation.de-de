---
title: Exportieren und Importieren virtueller Computer
description: Exportieren und Importieren virtueller Computer
keywords: windows 10, hyper-v
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-10-hyperv
ms.service: windows-10-hyperv
ms.assetid: 7fd996f5-1ea9-4b16-9776-85fb39a3aa34
translationtype: Human Translation
ms.sourcegitcommit: 8006ec4d71159113efce497d70ba591da2b84de9
ms.openlocfilehash: e11c8d1a8fabc6e8282c9396a8efcbc448b9a6d4

---

# Exportieren und Importieren virtueller Computer

Mithilfe der Export- und Importfunktionen von Hyper-V können Sie virtuelle Computer rasch duplizieren.  Exportierte virtuelle Computer können für die Sicherung oder zum Verschieben eines virtuellen Computers zwischen Hyper-V-Hosts verwendet werden.  

Mittels Import können Sie virtuelle Computer wiederherstellen.  Sie müssen einen virtuellen Computers nicht exportieren, um ihn importieren zu können. Während des Importvorgangs wird versucht, den virtuellen Computer aus den verfügbaren Daten wiederherzustellen.  Geben Sie mithilfe des Assistenten zum **Importieren virtueller Computer** den Speicherort der Dateien an. Dadurch wird der virtuelle Computer bei Hyper-V registriert und zur Verfügung gestellt.
 
Dieses Dokument begleitet Sie durch das Ex- und Importieren eines virtuellen Computers und einige der Optionen, die Sie beim Ausführen dieser Aufgaben wählen können.

## Exportieren eines virtuellen Computers

### Verwenden des Hyper-V-Managers

Wenn Sie einen Export eines virtuellen Computers erstellen, werden alle zugehörigen Dateien im Exportpaket gebündelt. Dazu gehören Konfigurationsdateien, Festplattendateien und auch alle vorhandenen Prüfpunktdateien. So erstellen Sie einen Export eines virtuellen Computers

1. Klicken Sie im Hyper-V-Manager mit der rechten Maustaste auf den gewünschten virtuellen Computer, und wählen Sie **Exportieren** aus.

2. Geben Sie den Speicherort für die exportierten Dateien an, und klicken Sie auf die Schaltfläche **Exportieren**.

**Hinweis:** Dieser Vorgang kann auf einem gestarteten und beendeten virtuellen Computer ausgeführt werden.

Nach Abschluss des Exportvorgangs finden Sie alle exportierten Dateien am Exportspeicherort.

### Mithilfe der PowerShell

Zum Exportieren eines virtuellen Computers mithilfe von PowerShell verwenden Sie den Befehl **Export-VM**. 

```powershell
Export-VM -Name <vm name> -Path <path>
```

Informationen zum Exportieren virtueller Computer mithilfe von Windows PowerShell finden Sie unter [Export-VM](https://technet.microsoft.com/library/hh848491.aspx).

## Importieren eines virtuellen Computers 

Beim Importieren wird der virtuelle Computer beim Hyper-V-Host registriert. Der Export eines virtuellen Computers kann zurück auf den Host, von dem dieser stammt, oder auf einen neuen Host importiert werden. 

Hyper-V bietet drei Importtypen:

- **Direktes Registrieren**: Exportdateien werden in dem Pfad abgelegt, in dem der virtuelle Computer ausgeführt werden soll. Nach dem Importieren hat der virtuelle Computer dieselbe ID wie zum Zeitpunkt des Exports. Wenn der virtuelle Computer bereits bei Hyper-V registriert ist, muss er deshalb zunächst gelöscht werden, damit der Import funktioniert. Nach Abschluss des Imports werden die Exportdateien zu den Dateien mit dem Ausführungszustand, die nicht entfernt werden können.

- **Virtuellen Computer wiederherstellen**: Sie haben die Möglichkeit, die VM-Dateien an einem bestimmten Speicherort zu speichern oder einen der Standardspeicherorte von Hyper-V zu verwenden. Bei diesem Importtyp wird eine Kopie der exportierten Datei erstellt und an den ausgewählten Speicherort verschoben. Nach dem Importieren hat der virtuelle Computer dieselbe ID wie zum Zeitpunkt des Exports. Wenn der virtuelle Computer bereits bei Hyper-V registriert ist, muss er deshalb zunächst gelöscht werden, ehe der Import abgeschlossen werden kann. Wenn der Import abgeschlossen ist, bleiben die exportierten Dateien unverändert und können entfernt und/oder erneut importiert werden.

- **Virtuellen Computer kopieren**: Dieser Importtyp ähnelt dem Typ „Wiederherstellen“ dahingehend, dass Sie einen Speicherort für die VM-Dateien auswählen können. Der Unterschied besteht darin, dass der virtuelle Computer nach dem Importieren eine neue eindeutige ID hat. Dies ermöglicht, dass der virtuelle Computer mehrmals auf denselben Host importiert werden kann.


### Verwenden des Hyper-V-Managers

So importieren Sie einen virtuellen Computer auf einen Hyper-V-Host

1. Klicken Sie im Hyper-V-Manager im Aktionsmenü auf **Virtuellen Computer importieren**.

2. Klicken Sie auf der Seite „Vorbereitungen“ auf **Weiter**.

3. Wählen Sie den Ordner mit den exportierten Dateien aus, und klicken Sie auf **Weiter**.

4. Wählen Sie den zu importierenden virtuellen Computer aus (wahrscheinlich gibt es nur eine Option).

5. Wählen Sie einen der drei Importtypen aus, und klicken Sie auf „Weiter“. 

6. Klicken Sie auf dem Bildschirm „Zusammenfassung“ auf **Fertig stellen**.

Der Assistent „Virtuellen Computer importieren“ führt Sie beim Importieren des virtuellen Computers auf den neuen Host auch durch die Schritte zur Behandlung eventueller Inkompatibilitäten. Der Assistent hilft Ihnen daher auch bei der Konfiguration der zugehörigen physischen Hardware wie z. B. Arbeitsspeicher, virtuelle Switches und virtuelle Prozessoren.

Beim Import eines virtuellen Computers führt der Assistent die folgenden Schritte aus:  
1. Er erstellt eine Kopie der Konfigurationsdatei des virtuellen Computers. Dies ist eine Vorsichtsmaßnahme für den Fall, dass der Host beispielsweise aufgrund eines Stromausfalls außerplanmäßig neu gestartet wird.  

2. Er überprüft die Hardware. Die Informationen in der Konfigurationsdatei des virtuellen Computers werden mit der Hardware des neuen Hosts verglichen.

3. Er erstellt eine Fehlerliste. Diese Liste zeigt auf, wo Konfigurationsbedarf besteht, und bestimmt somit, welche Assistentenseiten als nächstes angezeigt werden.

4. Zeigt die relevanten Seiten, einer Kategorie zu einem Zeitpunkt. Der Assistent wird erläutert, jede Inkompatibilität, mit denen Sie den virtuellen Computer neu konfigurieren, damit er mit dem neuen Host kompatibel ist.

5. Er entfernt die Kopie der Konfigurationsdatei. Danach kann der virtuelle Computer gestartet werden.


### Mithilfe der PowerShell

Zum Importieren eines virtuellen Computers mithilfe von PowerShell verwenden Sie den Befehl **Import-VM**.  Die folgenden Befehle dienen zum Demonstrieren des Importierens mithilfe aller drei Importtypen und von PowerShell.

Für einen Import mit direkter Registrierung eines virtuellen Computers sieht der Befehl etwa wie folgt aus. Wie bereits erwähnt, werden bei diesem Importvorgang die Dateien an dem Speicherort verwendet, an denen sie sich zum Zeitpunkt des Imports befinden, und die ID virtueller Computer wird beibehalten.

```powershell
Import-VM -Path 'C:\<vm export path>\2B91FEB3-F1E0-4FFF-B8BE-29CED892A95A.vmcx' 
```

Zum Importieren des virtuellen Computers unter Angabe Ihres eigenen Pfads für die Dateien des virtuellen Computers sieht der Befehl wie folgt aus.

```powershell
Import-VM -Path ‘C:\<vm export path>\2B91FEB3-F1E0-4FFF-B8BE-29CED892A95A.vmcx' -Copy -VhdDestinationPath 'D:\Virtual Machines\WIN10DOC' -VirtualMachinePath 'D:\Virtual Machines\WIN10DOC'
```

Für einen Kopierimportvorgang und das Verschieben der Dateien für den virtuellen Computer an den Hyper-V-Standardspeicherort wählen Sie einen Befehl wie diesen.

``` PowerShell
Import-VM -Path 'C:\<vm export path>\2B91FEB3-F1E0-4FFF-B8BE-29CED892A95A.vmcx' -Copy -GenerateNewId
```

Weitere Informationen finden Sie unter [Import-VM](https://technet.microsoft.com/library/hh848495.aspx).



<!--HONumber=Jun16_HO4-->


