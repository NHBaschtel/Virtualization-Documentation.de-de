# Bekannte Probleme der Insider-Builds

## Build16237

- Hyper-V-Container funktionieren nicht ordnungsgemäß. Diese Problemumgehung ist zur Verwendung von Hyper-V-Containern in 16237 erforderlich. Führen Sie diese Befehle in PowerShell aus:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server wird jetzt als Benutzer ausgeführt, damit die Befehle, die Administratorrechten erfordern, fehlschlagen. Eine Befehlszeile wie beispielsweise „RUN setx /M PATH” verursachen einen Fehler beim Build. In diesem Szenario können Sie folgende Alternative verwenden:

```dockerfile
RUN setx PATH <path>
```
