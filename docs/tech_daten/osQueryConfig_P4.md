# OSQuery Config

## Config Elemente

### Installation
Zweck: Installiert osquery auf dem Host
>[!NOTE]
>Hinweis: Standardmässig wird osquery so konfiguriert, dass es eine Verbindung zum Fleet-Server auf dem Host "logger" über TLS herstellt.

````bash
Write-Host "$('[{0:HH:mm}]' -f (Get-Date)) Installiere osquery..."
$flagfile = "c:\Program Files\osquery\osquery.flags"
choco install -y --limit-output --no-progress osquery | Out-String  # Offenbar macht Out-String den Prozess wartend
````

Überprüfe, ob der osquery-Dienst noch nicht installiert ist

````bash
$service = Get-WmiObject -Class Win32_Service -Filter "Name='osqueryd'"
If (-not ($service)) {
# Setze osquery als Dienst auszuführen
Write-Host "Setze osquery als Dienst auszuführen"
New-Service -Name "osqueryd" -BinaryPathName "C:\Program Files\osquery\osqueryd\osqueryd.exe --flagfile=`"C:\Program Files\osquery\osquery.flags`""
````

### Flag-Datei von Palantir Github herunter
````bash
# GitHub erfordert TLS 1.2 seit dem 1. Februar 2018
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/palantir/osquery-configuration/master/Classic/Endpoints/Windows/osquery.flags" -OutFile $flagfile
````

## Eintrag zur Hosts-Datei für Kolide für die SSL-Validierung hinzu
Zweck: SSL-Validierung
````bash
Add-Content "c:\windows\system32\drivers\etc\hosts" "        192.168.38.105    kolide"
````

## Hinzufügen des Kolide Secret
 >[!WARNING]
 > Vermeide BOM [byte-order mark](https://www.w3.org/International/questions/qa-byte-order-mark.de)
````bash
$Utf8NoBomEncoding = New-Object System.Text.UTF8Encoding $False
[System.IO.File]::WriteAllLines("c:\Program Files\osquery\kolide_secret.txt", "enrollmentsecret", $Utf8NoBomEncoding)
````

## TLS-Serverhostname anpassen in der Flag-Datei
````bash
(Get-Content $flagfile) -replace 'tls.endpoint.server.com', 'kolide:8412' | Set-Content $flagfile
````

## Pfad zu den Secrets anpassen
````bash
(Get-Content $flagfile) -replace 'path\\to\\file\\containing\\secret.txt', 'Program Files\osquery\kolide_secret.txt' | Set-Content $flagfile
````

## Pfad zur Zertifikatsdatei anpassen
````bash
(Get-Content $flagfile) -replace 'c:\\ProgramData\\osquery\\certfile.crt', 'c:\Program Files\osquery\certfile.crt' | Set-Content $flagfile
````

## Entferne die verbose-Flagge und ersetzen
Zweck: Protokollierung minimieren und spezifische Ereignisse zu erfassen
````bash
(Get-Content $flagfile) -replace '--verbose=true', '--logger_min_status=1' | Set-Content $flagfile
````

## Füge Zertifikat hinzufügen
````bash
Copy-Item "c:\vagrant\resources\fleet\server.crt" "c:\Program Files\osquery\certfile.crt"
````

## Starten des Dienstes
````bash
Start-Service osqueryd
}
else {
Write-Host "osquery ist bereits installiert. Fahre fort."
}
````

## Überprüfung
````bash
If ((Get-Service -name osqueryd).Status -ne "Running") {
throw "osqueryd-Dienst wurde nicht gestartet"
}
````