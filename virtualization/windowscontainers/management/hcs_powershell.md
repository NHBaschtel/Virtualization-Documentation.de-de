



# Verwaltungsinteroperabilität

**Dieser Inhalt ist vorläufig und kann geändert werden.**

Zumeist gilt, dass mit PowerShell erstellte Windows-Container mit PowerShell und mit Docker erstellte Container mit Docker verwaltet werden müssen. Allerdings bietet das PowerShell-Modul „Host Computing“ die Möglichkeit **ausgeführte** Container zu ermitteln und zu beenden, und zwar unabhängig davon, wie sie erstellt wurden. Dieses Modul verhält sich wie ein Task-Manager für Container, die auf einem Containerhost ausgeführt werden.

## Anzeigen aller Container

Mit dem Befehl `Get-ComputeProcess` können Sie eine Liste der Container zurückgeben.

```powershell
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## Beenden eines Containers

Mit dem Befehl `Stop-ComputeProcess` können Sie einer Container unabhängig davon, ob er mit PowerShell oder Docker erstellt wurde, beenden.

> Zum Zeitpunkt der Verfassung dieses Artikels musste der VMMS-Dienst neu gestartet werden, damit bei Verwenden des Befehls `Get-Container` die Container als beendet angezeigt werden.

```powershell
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```






<!--HONumber=Feb16_HO3-->


