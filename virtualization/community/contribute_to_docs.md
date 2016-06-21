---
title: &815299562 Communityressourcen
description: Communityressourcen
keywords: windows 10, hyper-v, container, docker
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &1136166802 virtualization
ms.service: virtualization
ms.assetid: 731ed95a-ce13-4c6e-a450-49563bdc498c
---

# Beiträge zu den Dokumenten leisten

> <g id="1" ctype="x-strong">Hinweis</g>: Um Beiträge erstellen zu können, benötigen Sie ein Konto auf <g id="3CapsExtId1" ctype="x-link"><g id="3CapsExtId2" ctype="x-linkText">GitHub</g><g id="3CapsExtId3" ctype="x-title"></g></g>.

## Bearbeiten eines vorhandenen Dokuments

1. Suchen Sie das Dokument, das Sie bearbeiten möchten.

2. Klicken Sie auf die entsprechende Schaltfläche (<g id="2" ctype="x-strong">Contribute to this Topic</g>), um einen Beitrag zu leisten.  
  <g id="1" ctype="x-linkText"></g>

  Sie werden daraufhin automatisch zu der in GitHub mit dieser Datei verknüpften Markdowndatei weitergeleitet.

  Stellen Sie sicher, dass Sie bei GitHub angemeldet sind. Melden Sie sich andernfalls an, oder erstellen Sie ein GitHub-Konto.

  <g id="1" ctype="x-linkText"></g>

3. Klicken Sie auf das Symbol „Bearbeiten“, um den im Browser enthaltenen Editor aufzurufen.

  <g id="1" ctype="x-linkText"></g>

4. Achten Sie bei Änderungen auf den Kontext.

  Mögliche Aktionen:
  1. Datei bearbeiten
  2. Änderungen als Vorschau anzeigen
  3. Datei umbenennen (was Sie vermutlich nicht tun möchten)

  <g id="1" ctype="x-linkText"></g>

5. Änderungen als Pull-Anforderung vorschlagen

  <g id="1" ctype="x-linkText"></g>

6. Überprüfen Ihrer Änderungen

  <g id="1" ctype="x-strong">Bei einer Pull-Anforderung achten wir auf Folgendes</g>
  * Die Änderung ist richtig – sie spiegelt die Technologie genau wider.
  * Rechtschreibung und Grammatik sind richtig.
  * Sie befindet sich an einer logischen Position in der Dokumentation.

  <g id="1" ctype="x-linkText"></g>

7. Erstellen einer <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Pull-Anforderung</g><g id="2CapsExtId3" ctype="x-title"></g></g>

## Pull-Anforderungen

Die meisten Änderungen werden per Pull-Anforderung ausgeführt. Eine Pull-Anforderung ist eine Möglichkeit, ein Changeset gemeinsam mit mehrere Reviewern zu überprüfen, die den aktuellen Inhalt ändern und kommentieren können.


## Verzweigen des Repositorys und lokales Bearbeiten

Wenn Sie länger mit Dokumenten arbeiten müssen, klonen Sie das Repository lokal, und arbeiten Sie auf Ihrem Computer.

Die folgende Anleitung zeigt, wie Sie mein (Sarah Cooleys) Setup emulieren. Es gibt viele alternative Setups, die ebenso gut funktionieren.

> <g id="1" ctype="x-strong">Hinweis:</g> All diese Dokumenttools funktionieren ebenso gut unter Linux/OS X. Wenn Sie weitere Anleitungen möchten, fragen Sie uns bitte.

Die vorliegende Anleitung ist in drei Abschnitte unterteilt:
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Einrichten von Git</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * Installieren von Git
  * Anfängliches Setup
  * Verzweigen des Dokumentationsrepositorys
  * Klonen Ihrer Kopie auf Ihrem lokalen Computer
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Verwalten der anfänglichen Anmeldeinformationen</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * Informationen über das Ausführen eines Stashs für Anmeldeinformationen und über Hilfsprogramme für Anmeldeinformationen
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Einrichten der Dokumentumgebung</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * Installieren von VSCode
  * Exemplarische Vorgehensweise für VSCode für Git – einige nützliche Funktionen
  * Ausführen des ersten Commits

### Einrichten von Git

1. Installieren Sie Git (unter Windows) von <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">hier</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

  In dieser Installation müssen Sie nur einen Wert ändern:

  <g id="1" ctype="x-strong">Anpassen Ihrer PATH-Umgebung</g>
  Verwenden Sie Git über die Windows-Eingabeaufforderung.

  <g id="1" ctype="x-linkText"></g>

  So können Sie Git-Befehle in der PowerShell-Konsole und in jeder anderen Windows-Konsole verwenden.

2. Konfigurieren Sie Ihre Git-Identität.

  Öffnen Sie ein PowerShell-Fenster, und führen Sie Folgendes aus:

  ``` PowerShell
  git config --global user.name "User Name"
  git config --global user.email username@microsoft.com
  ```

  Git verwendet diese Werte, um Ihre Commits zu bezeichnen.

> Wenn folgender Fehler angezeigt wird, ist Git möglicherweise nicht ordnungsgemäß installiert, oder Sie müssen PowerShell neu starten.
>    ``` PowerShell
>    git: Der Begriff 'git' wurde nicht als Name eines Cmdlets, einer Funktion, einer Skriptdatei oder eines ausführbaren Programms erkannt. Prüfen Sie die Schreibweise des Namens bzw. stellen Sie sicher, dass der Pfad korrekt angegeben wurde, und versuchen Sie es erneut.
>    ```

3. Konfigurieren Sie Ihre Git-Umgebung.

   Richten Sie ein Hilfsprogramm für Anmeldeinformationen ein, damit Sie Benutzername und Kennwort (zumindest auf diesem Computer) nur einmal eingeben müssen.
   Ich habe dieses einfache <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Windows-Hilfsprogramm für Anmeldeinformationen</g><g id="2CapsExtId3" ctype="x-title"></g></g> verwendet.

   Nach der Installation führen Sie folgende Befehle aus, um das Programm zu aktivieren und das Pushverhalten einzurichten:
   ```
   git config --global credential.helper manager
   git config --global push.default simple
   ```

   Beim ersten Mal müssen Sie sich bei GitHub authentifizieren – Sie werden aufgefordert, Ihren Benutzernamen, Ihr Kennwort und Ihren Code für die zweistufige Authentifizierung einzugeben, sofern Sie diese aktiviert haben.
   Beispiel:
   ```
   C:\Users\plang\Source\Repos\Virtualization-Documentation [master]> git pull
   Please enter your GitHub credentials for https://github.com/
   username: plang@microsoft.com
   password:
   authcode (app): 562689
   ```
   Damit wird automatisch ein <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Persönliches Zugriffstoken</g><g id="2CapsExtId3" ctype="x-title"></g></g> mit den richtigen Berechtigungen für GitHub erstellt.
   Speichern Sie dieses Token an einem sicheren Ort auf dem lokalen Computer. Bei späteren Anmeldungen werden Sie nicht mehr zur Eingabe von Anmeldeinformationen aufgefordert.

4. Verzweigen Sie das Repository

5. Klonen Sie das Repository

  „git clone“ erstellt eine lokale Kopie des Git-Repositorys mit den richtigen Hooks zur Synchronisierung mit anderen Klonen des gleichen Repositorys.

  Standardmäßig erstellt der Befehl einen Ordner mit dem gleichen Namen wie das Repository im aktuellen Verzeichnis. Ich speichere all meine Git-Repositorys in meinem Benutzerverzeichnis. Weitere Informationen über „git clone“ finden Sie <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">hier</g><g id="2CapsExtId3" ctype="x-title"></g></g>.

  ``` PowerShell
  cd ~
  git clone https://github.com/Microsoft/Virtualization-Documentation.git
  ```

  Wenn der Befehl erfolgreich ausgeführt wurde, verfügen Sie jetzt über einen Ordner <g id="2" ctype="x-code">Virtualization Documentation</g>.

  ``` PowerShell
  cd Virtualization-Documentation
  ```

5. (Optional) Richten Sie Posh-Git ein.

  Posh-Git ist ein von der Community erstelltes PowerShell-Modul, das die Verwendung von Git in PowerShell etwas benutzerfreundlicher gestaltet. Das Modul fügt die Vervollständigung per Tabulatortaste zu PowerShell hinzu und ermöglicht die Anzeige von Informationen zu Verzweigung und Dateistatus an der Eingabeaufforderung. Weitere Informationen dazu finden Sie <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">hier</g><g id="2CapsExtId3" ctype="x-title"></g></g>. Sie können Posh-Git installieren, indem Sie folgenden Befehl in einer PowerShell-Administratorkonsole ausführen.

  ``` PowerShell
  Install-Module -Name posh-git
  ```

  Um Posh-Git so einzurichten, dass es bei jedem Start von PowerShell aktiviert wird, fügen Sie Ihrem PowerShell-Profil (z. B. <g id="2" ctype="x-code">%UserProfile%\My Documents\WindowsPowerShell\profile.ps1 </g>) folgenden Befehl hinzu:

  ``` PowerShell
  Import-Module posh-git

  function global:prompt {
    $realLASTEXITCODE = $LASTEXITCODE

    Write-Host($pwd.ProviderPath) -nonewline

    Write-VcsStatus

    $global:LASTEXITCODE = $realLASTEXITCODE
    return "> "
  }
  ```

### Validieren und Ausführen eines Stashs für Anmeldeinformationen

  Um zu überprüfen, ob das Repository ordnungsgemäß eingerichtet ist, rufen Sie per Pull neue Inhalte ab.

  ``` PowerShell
  git pull
  ```


### Einrichten der Umgebung für die Markdownbearbeitung

1. Laden Sie VSCode herunter.

6. Führen Sie einen Testcommit aus. Wenn ein ordnungsgemäßer Stash für Ihre Anmeldeinformationen ausgeführt wurde, sollte alles reibungslos funktionieren.









<!--HONumber=May16_HO1-->


