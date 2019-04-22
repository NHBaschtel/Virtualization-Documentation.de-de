# <a name="known-issues-for-insider-builds"></a>Bekannte Probleme bei der Insider-builds

## <a name="build-16237"></a>Build16237

- Hyper-V-Isolierung ist nicht ordnungsgemäß funktioniert. Diese problemumgehung ist zur Verwendung von Hyper-V-Isolierung in 16237 erforderlich. Führen Sie diese Befehle in PowerShell aus:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server wird jetzt als Benutzer ausgeführt, sodass Befehle, die Administratorrechte erfordern, fehlschlagen. Eine Befehlszeile wie beispielsweise „RUN setx /M PATH” verursachen einen Fehler beim Build. In diesem Szenario können Sie folgende Alternative verwenden:

```dockerfile
RUN setx PATH <path>
```
