# <a name="known-issues-for-insider-builds"></a>Bekannte Probleme bei Insider-Builds

## <a name="build-16237"></a>Build 16237

- Die Hyper-V-Isolation funktioniert nicht ordnungsgemäß. Diese Problem Umgehung ist erforderlich, um die Hyper-V-Isolation in Build 16237 zu verwenden. Führen Sie diese Befehle in PowerShell aus:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server wird jetzt als Benutzer ausgeführt, sodass Befehle, die Administratorrechte erfordern, fehlschlagen. Eine Befehlszeile wie beispielsweise „RUN setx /M PATH” verursachen einen Fehler beim Build. In diesem Szenario können Sie folgende Alternative verwenden:

```dockerfile
RUN setx PATH <path>
```
