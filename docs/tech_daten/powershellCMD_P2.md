# Powershell Cmdlets

## Cmdlets WSUS-Dienst

### Installieren des WSUS-Dienstes:
````powershell
Install-WindowsFeature -Name UpdateServices -IncludeManagementTools
````

### Konfigurieren der WSUS-Verbindung:
````powershell
$wsusServer = "WSUSServerName"
$wsusPort = "8530"
$wsusConfig = Get-WsusServer -Name $wsusServer -PortNumber $wsusPort
````

### Konfigurieren von WSUS-Einstellungen:
````powershell
Set-WsusServerSynchronization -SyncFromMU
````

### Manuelles Auslösen der Synchronisierung:
````powershell
Invoke-WsusServerSync
````

### Genehmigen von Updates:
````powershell
Get-WsusUpdate | Approve-WsusUpdate -Action Install -TargetGroupName "All Computers"
````

### Überprüfen des Update-Status:
````powershell
Get-WsusUpdate | Get-WsusUpdateApproval | Where-Object {$_.Action -eq "Install"} | Format-Table -Property UpdateTitle, ComputerTargetGroupName
````

### Konfigurieren von Gruppenrichtlinien für WSUS:
````powershell
$gpo = Get-GPO -Name "WSUS Policy"
Set-GPRegistryValue -Guid $gpo.Id -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" -ValueName "WUServer" -Type String -Value "http://WSUSServerName:8530"
````

### Überprüfen der WSUS-Client-Einstellungen:
````powershell
Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate"
````

## Cmdlets WSUS-Client

### Get-WindowsUpdateLog:

   Gibt Informationen zum Windows Update-Clientprotokoll zurück.
````powershell
Get-WindowsUpdateLog
````

### Get-WUServiceManager:

   Zeigt Informationen zum Windows Update-Dienst an, einschliesslich des konfigurierten WSUS-Servers.
````powershell
Get-WUServiceManager
````

### Set-WUSettings:

   Legt die WSUS-Konfigurationsoptionen fest, einschliesslich des WSUS-Servers.
````powershell
Set-WUSettings -MicrosoftUpdateServer "http://WSUSServerName:8530" -AcceptAll
````

### Get-WUInstall:

   Startet den Installationsprozess für verfügbare Updates.
````powershell
Get-WUInstall -AcceptAll -AutoReboot
````
### Get-WURebootStatus:

   Überprüft den Neustatus des Systems nach der Installation von Updates.
````powershell
Get-WURebootStatus
````

### Invoke-WUJob:

   Startet einen Windows Update-Job, um Updates zu suchen und zu installieren.
````powershell
Invoke-WUJob -RunNow
````

### Get-WULastResults:

   Zeigt die Ergebnisse des letzten Windows Update-Scans an.
````powershell
Get-WULastResults
````

### Remove-WUServiceManager:

   Entfernt den konfigurierten WSUS-Server vom Windows Update-Dienst.
````powershell
Remove-WUServiceManager
````