# Sysmon Installation

## Manuelle Installation

### Herunterladen von Sysmon:
Besuchen Sie die Sysinternals [Sysmon-Website](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) auf der Microsoft-Dokumentationsseite.
Laden Sie die neueste Version von Sysmon herunter.

### Entpacken der Sysmon-Zip-Datei:
Extrahieren Sie die heruntergeladene Zip-Datei an einen Ort Ihrer Wahl auf dem Ziel-Windows-System.

### Installation von Sysmon:

Cmd als Administrator öffnen  
Navigieren zum Verzeichnis, in dem Sysmon extrahiert wurde.
Führen Sie den Befehl für die Installation von Sysmon aus.

````bash
sysmon.exe -i
````
Dieser Befehl installiert Sysmon als Dienst.

### Sysmon-Konfiguration (optional):

Sie können Sysmon nach Ihren Anforderungen konfigurieren, indem Sie eine Konfigurationsdatei erstellen und diese beim Installationsprozess verwenden.

````bash
sysmon.exe -i -c sysmon_config.xml
````

Beispiel config-file:
````xml
<Sysmon schemaversion="4.22">
    <HashAlgorithms>*</HashAlgorithms>
    <EventFiltering>
        <NetworkConnect onmatch="exclude">
            <Image condition="is">svchost.exe</Image>
        </NetworkConnect>
        <ImageLoad onmatch="include">
            <Image condition="is">c:\windows\system32\cmd.exe</Image>
        </ImageLoad>
    </EventFiltering>
</Sysmon>
````

### Sysmon starten:

Starten Sie den Sysmon-Dienst nach der Installation mit dem folgenden Befehl:

````bash
sysmon.exe -c
````

Dieser Befehl startet Sysmon mit den aktuellen Konfigurationseinstellungen.

## Automatisiert Installation

### Sysinternals-Tools installieren:

- Ein Verzeichnis für die Tools wird erstellt (C:\Tools\Sysinternals).  
- Überprüfung, ob das Verzeichnis bereits existiert. Falls ja, wird das Skript beendet.  
- Ein weiteres Verzeichnis für Sysmon wird erstellt (C:\ProgramData\Sysmon).  
- Überprüfung, ob das Sysmon-Verzeichnis bereits existiert. Falls ja, wird das Skript beendet.  
- Download und Installation von verschiedenen Sysinternals-Tools wie Autoruns, Procmon, PsExec, Procexp, Sysmon, und Tcpview.

````bash
Write-Host "$('[{0:HH:mm}]' -f (Get-Date)) Installing SysInternals Tooling..."
$sysinternalsDir = "C:\Tools\Sysinternals"
$sysmonDir = "C:\ProgramData\Sysmon"
If(!(test-path $sysinternalsDir)) {
  New-Item -ItemType Directory -Force -Path $sysinternalsDir
} Else {
  Write-Host "Tools directory exists. Exiting."
  exit
}

If(!(test-path $sysmonDir)) {
  New-Item -ItemType Directory -Force -Path $sysmonDir
} Else {
  Write-Host "Sysmon directory exists. Exiting."
  exit
}

$autorunsPath = "C:\Tools\Sysinternals\Autoruns64.exe"
$procmonPath = "C:\Tools\Sysinternals\Procmon.exe"
$psexecPath = "C:\Tools\Sysinternals\PsExec64.exe"
$procexpPath = "C:\Tools\Sysinternals\procexp64.exe"
$sysmonPath = "C:\Tools\Sysinternals\Sysmon64.exe"
$tcpviewPath = "C:\Tools\Sysinternals\Tcpview.exe"
$sysmonConfigPath = "$sysmonDir\sysmonConfig.xml"


# Microsoft likes TLSv1.2 as well
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Write-Host "$('[{0:HH:mm}]' -f (Get-Date)) Downloading Autoruns64.exe..."
(New-Object System.Net.WebClient).DownloadFile('https://live.sysinternals.com/Autoruns64.exe', $autorunsPath)
Write-Host "$('[{0:HH:mm}]' -f (Get-Date)) Downloading Procmon.exe..."
(New-Object System.Net.WebClient).DownloadFile('https://live.sysinternals.com/Procmon.exe', $procmonPath)
Write-Host "$('[{0:HH:mm}]' -f (Get-Date)) Downloading PsExec64.exe..."
(New-Object System.Net.WebClient).DownloadFile('https://live.sysinternals.com/PsExec64.exe', $psexecPath)
Write-Host "$('[{0:HH:mm}]' -f (Get-Date)) Downloading procexp64.exe..."
(New-Object System.Net.WebClient).DownloadFile('https://live.sysinternals.com/procexp64.exe', $procexpPath)
Write-Host "$('[{0:HH:mm}]' -f (Get-Date)) Downloading Sysmon64.exe..."
(New-Object System.Net.WebClient).DownloadFile('https://live.sysinternals.com/Sysmon64.exe', $sysmonPath)
Write-Host "$('[{0:HH:mm}]' -f (Get-Date)) Downloading Tcpview.exe..."
(New-Object System.Net.WebClient).DownloadFile('https://live.sysinternals.com/Tcpview.exe', $tcpviewPath)
Copy-Item $sysmonPath $sysmonDir
````

### Sysmon-Konfiguration:
Download einer Sysmon-Konfigurationsdatei von Olaf Hartong.
Alternative Möglichkeit: Download einer Sysmon-Konfiguration von SwiftOnSecurity.
Start von Sysmon mit der heruntergeladenen Konfigurationsdatei.

````bash
# Download Olaf Hartongs Sysmon config
Write-Host "$('[{0:HH:mm}]' -f (Get-Date)) Downloading Olaf Hartong's Sysmon config..."
(New-Object System.Net.WebClient).DownloadFile('https://raw.githubusercontent.com/olafhartong/sysmon-modular/master/sysmonconfig.xml', "$sysmonConfigPath")
# Alternative: Download SwiftOnSecurity's Sysmon config
# Write-Host "Downloading SwiftOnSecurity's Sysmon config..."
# (New-Object System.Net.WebClient).DownloadFile('https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml', "$sysmonConfigPath")

# Start Sysmon
Write-Host "$('[{0:HH:mm}]' -f (Get-Date)) Starting Sysmon..."
Start-Process -FilePath "$sysmonDir\Sysmon64.exe" -ArgumentList "-accepteula -i $sysmonConfigPath"
````

### Überprüfung des Sysmon-Dienststatus:
Warten auf 5 Sekunden, um dem Sysmon-Dienst Zeit zum Starten zu geben.
Überprüfung, ob der Sysmon-Dienst erfolgreich gestartet wurde.

````bash
Write-Host "$('[{0:HH:mm}]' -f (Get-Date)) Verifying that the Sysmon service is running..."
Start-Sleep 5 # Give the service time to start
If ((Get-Service -name Sysmon64).Status -ne "Running")
{
  throw "The Sysmon service did not start successfully"
}
````

### Event Log Channel konfigurieren:
Konfiguration des Event-Log-Channels "Microsoft-Windows-Sysmon/Operational" für bessere Lesbarkeit.


````bash
# Make the event log channel readable. For some reason this doesn't work in the GPO and only works when run manually.
wevtutil sl Microsoft-Windows-Sysmon/Operational "/ca:O:BAG:SYD:(A;;0x5;;;BA)(A;;0x1;;;S-1-5-20)(A;;0x1;;;S-1-5-32-573)"
````