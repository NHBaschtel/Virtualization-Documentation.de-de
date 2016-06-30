### PowerShell für Docker

Im Gespräch mit unseren Benutzern – in Foren, über Twitter, in GitHub oder auch persönlich – wurde uns eine Frage häufiger gestellt als jede andere: Warum werden Docker-Container in PowerShell nicht angezeigt? 

Nach einer ausführlichen Diskussion über die Vor- und Nachteile sowie die verschiedenen Optionen sind wir zu dem Schluss gekommen, dass das PowerShell-Containermodul ein Update vertragen kann ... Wir werden also das PowerShell-Containermodul, das mit den Vorschaubuilds von Windows Server 2016 bereitgestellt wurde, außer Betrieb nehmen und haben damit begonnen, es durch ein neues PowerShell-Modul für Docker zu ersetzen.  Die Entwicklung dieses neuen Moduls ist bereits in vollem Gang, dabei verfolgen wir jedoch einen anderen Ansatz als bisher: Wir machen unsere Arbeit öffentlich.  Wir werden bei diesem Modul verstärkt auf die Zusammenarbeit mit der Community setzen, damit das Docker-Modul zu einer benutzerfreundlicheren PowerShell-Oberfläche für Container beiträgt.  Dieses neue Modul setzt direkt auf der REST-Schnittstelle des Docker-Moduls auf und ermöglicht es Benutzern, die Docker-Befehlszeilenschnittstelle, PowerShell oder beide Tools einzusetzen.

Die Entwicklung eines leistungsstarken PowerShell-Moduls ist kein leichtes Unterfangen: Nicht nur der Code muss stimmen, sondern auch die richtige Balance zwischen Objekten, Parametersätzen und Cmdlet-Namen.  Deshalb wenden wir uns an Sie – unsere Endbenutzer und die riesige PowerShell- und Docker-Community: Helfen Sie uns, dieses Modul perfekt zu machen!  Welche Parametersätze sind wichtig für Sie?  Sollen wir ein Äquivalent zu „docker run“ einbauen, oder sollte eine Pipe von new-container zu start-container vorhanden sein – was ziehen Sie vor?  Um weitere Informationen zu diesem Modul zu erhalten und sich an der Entwicklung zu beteiligen, melden Sie sich bei unserer GitHub-Seite an (https://github.com/Microsoft/Docker-PowerShell/), und machen Sie mit.

Sobald wir ein solides Alphamodul entwickelt haben, werden wir dieses im PowerShell-Katalog bereitstellen und diese Seite mit Informationen zur Verwendung aktualisieren.


<!--HONumber=Jun16_HO4-->


