# Anwendungskompatibilität im Windows-Containern

Dies ist eine Vorschau. Ziel ist es, dass Anwendungen, die unter Windows ausgeführt werden, auch in einem Container ausgeführt werden können. In diesem Artikel wollen wir uns mit unserem derzeitigen Kompatibilitätsstatus von Anwendungen beschäftigen.

Der einzige Zweck dieses Dokuments ist, Sie über unsere Erfahrungen zu informieren.

Fehlt etwas in dieser Liste? Lassen Sie uns über [die Foren](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers) wissen, was in Ihrer Umgebung funktioniert und was nicht.

## Windows Server-Container

Wir haben versucht, die folgenden Programme in einem Windows Server-Container auszuführen. Diese Ergebnisse garantieren nicht , dass die Anwendung ordnungsgemäß funktioniert.

| **Name**| **Version**| **Windows Server Core-Basis-Image**| **Nano Server-Basis-Image**| **Anmerkungen**|
|:-----|:-----|:-----|:-----|:-----|
| .NET| 3.5| Ja| Unbekannt| |
| .NET| 4.6| Ja| Unbekannt| |
| .NET CLR| 5 Beta 6| Ja| Ja| x64 und x86|
| Active Python| 3.4.1| Ja| Ja| |
| Apache Cassandra| | Ja| Unbekannt|
| Apache CouchDB| 1.6.1| Nein| Nein| |
| Apache Hadoop| | Ja| Nein| |
| Apache HTTPD| 2.4| Ja| Ja| VC++-Laufzeit wird nicht installiert, wenn der „dedup“-Filter geladen wird.Entladen Sie „dedup“ mithilfe von `fltmc unload dedup`.|
| Apache Tomcat| 8.0.24 x64| Ja| Unbekannt| |
| ASP.NET| 4.6| Ja| Unbekannt| |
| ASP.NET| 5 Beta 6| Ja| Ja| x64 und x86|
| Django| | Ja| Ja| |
| Go| 1.4.2| Ja| Ja| |
| Internetinformationsdienste| 10.0| Ja| Ja| VC++-Laufzeit wird nicht installiert, wenn der „dedup“-Filter geladen wird.Entladen Sie „dedup“ mithilfe von `fltmc unload dedup`.|
| Java| 1.8.0_51| Ja| Ja| Verwenden Sie die Serverversion.Die Clientversion wird nicht ordnungsgemäß installiert.|
| MongoDB| 3.0.4| Ja| Unbekannt| |
| MySQL| 5.6.26| Ja| Ja| |
| NGinx| 1.9.3| Ja| Ja| |
| Node.js| 0.12.6| Teilweise| Teilweise| NPM kann Pakete nicht herunterladen.|
| Perl| | Ja| Unbekannt| |
| PHP| 5.6.11| Ja| Ja| VC++-Laufzeit wird nicht installiert, wenn der „dedup“-Filter geladen wird.Entladen Sie „dedup“ mithilfe von `fltmc unload dedup`.|
| PostgreSQL| 9.4.4| Ja| Unbekannt| VC++-Laufzeit wird nicht installiert, wenn der „dedup“-Filter geladen wird.Entladen Sie „dedup“ mithilfe von `fltmc unload dedup`.|
| Python| 3.4.3| Ja| Ja| |
| R| 3.2.1| Nein| Nein| |
| RabbitMQ| 3.5.x| Ja| Unbekannt| |
| Redis| 2.8.2101| Ja| Ja| |
| Ruby| 2.2.2| Ja| Ja| x64 und x86|
| Ruby on Rails| 4.2.3| Ja| Ja| |
| SQLite| 3.8.11.1| Ja| Nein| |
| SQL Server Express| 2014 LocalDB| Nein| Nein| |
| Sysinternals-Tools| *| Ja| Ja| Es wurden nur diejenigen getestet, die keine grafische Benutzeroberfläche benötigen.PsExec funktioniert mit dem aktuellen Design nicht.|

## Hyper-V-Container

Wir haben versucht, die folgenden Programme in einem Hyper-V-Container auszuführen. Diese Ergebnisse garantieren nicht , dass die Anwendung ordnungsgemäß funktioniert.

| **Name**| **Version**| **Nano Server-Basis-Image**| **Anmerkungen**|
|:-----|:-----|:-----|:-----|
| Apache Hadoop| | Nein| |
| Apache HTTPD| 2.4| Ja| VC++-Laufzeit wird nicht installiert, wenn der „dedup“-Filter geladen wird.Entladen Sie „dedup“ mithilfe von `fltmc unload dedup`.|
| ASP.NET| 5 Beta 6| Ja| x64 und x86|
| Django| | Ja| Wenn das Image mit einer DockerFile erstellt wird und Python-Binärdateien als Teil davon kopiert werden, funktioniert Python nicht.Starten Sie den Container, und kopieren Sie die Python-Binärdateien.|
| Go| 1.4.2| Ja| |
| Internetinformationsdienste| 10.0| Ja| Mithilfe von „dism“ kann IIS nicht direkt installiert.Führen Sie eine unbeaufsichtigte Installation von IIS mithilfe von „dism“-Befehlen aus.|
| Java| 1.8.0_51| Ja| Verwenden Sie die Serverversion.Die Clientversion wird nicht ordnungsgemäß installiert.|
| MySQL| 5.6.26| Ja| |
| NGinx| 1.9.3| Ja| |
| Node.js| 0.12.6| Teilweise| NPM kann Pakete nicht herunterladen.|
| Python| 3.4.3| Ja| Wenn das Image mit einer DockerFile erstellt wird und Python-Binärdateien als Teil davon kopiert werden, funktioniert Python nicht.Starten Sie den Container, und kopieren Sie die Python-Binärdateien.|
| Redis| 2.8.2101| Ja| |
| Ruby| 2.2.2| Ja| x64 und x86|
| Ruby on Rails| 4.2.3| Ja| |
| Sysinternals-Tools| | Ja| Es wurden nur diejenigen getestet, die keine grafische Benutzeroberfläche benötigen.PsExec funktioniert mit dem aktuellen Design nicht.|

## Teilen Sie uns Ihre Erfahrungen mit.

Fehlt etwas in dieser Liste? Lassen Sie uns über [die Foren](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers) wissen, was in Ihrer Umgebung funktioniert und was nicht.




