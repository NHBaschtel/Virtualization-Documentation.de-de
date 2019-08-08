# <a name="known-issues-for-insider-builds"></a>Bekannte Probleme bei Insider-Builds

## <a name="build-16237"></a>Build16237

- Die Hyper-V-Isolierung funktioniert nicht ordnungsgemäß. Diese Problemumgehung ist erforderlich, um die Hyper-V-Isolierung in Build 16237 zu verwenden. Führen Sie diese Befehle in PowerShell aus:

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server wird nun als Benutzer ausgeführt, daher schlagen Befehle, die Administratorrechte erfordern, fehl. Eine Befehlszeile wie beispielsweise „RUN setx /M PATH” verursachen einen Fehler beim Build. In diesem Szenario können Sie folgende Alternative verwenden:

```dockerfile
RUN setx PATH <path>
```
