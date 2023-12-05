# Sysmon XML Config

## Config Elemente

### Sysmon-Schema-Version:

Zweck: Gibt die Schema-Version von Sysmon an.

````xml
    <Sysmon schemaversion="4.90"/>
````
### Hash-Algorithmen:

Zweck: Definiert die Hash-Algorithmen, die Sysmon verwenden soll.

````xml
    <HashAlgorithms>*</HashAlgorithms>
````

### Überprüfung der Widerrufung:

Zweck: Bestimmt, ob die Überprüfung der Zertifikatswiderrufung aktiviert ist.

````xml
    <CheckRevocation>False</CheckRevocation>
````

### DNS-Lookup:

Zweck: Deaktiviert das DNS-Lookup-Verhalten von Sysmon.

````xml
    <DnsLookup>False</DnsLookup>
````

### Archivverzeichnis:

Zweck: Setzt den Namen des Verzeichnisses im Stammverzeichnis von C:, in dem gespeicherte Dateien gespeichert werden sollen.
````xml
    <ArchiveDirectory>Sysmon</ArchiveDirectory>
````
### EventFiltering - Regelgruppe für ProcessCreate:

Zweck: Filtert Ereignisse für die Prozesserstellung basierend auf verschiedenen Bedingungen.
s
````xml
    <EventFiltering>
      <RuleGroup groupRelation="or">
      </RuleGroup>
    </EventFiltering>
````

### ProcessCreate onmatch

Zweck: Definiert das Verhalten, wenn die Bedingungen in einer Regel erfüllt sind.
````xml
<ProcessCreate onmatch="include">
  <!-- Weitere Regelbedingungen und Aktionen -->
</ProcessCreate>
````

### ParentImage condition

Zweck: Legt Bedingungen für das übergeordnete (aufrufende) Image des Prozesses fest.
````xml

<ParentImage condition="is" name="...">
  <!-- Weitere Bedingungen für das übergeordnete Image -->
</ParentImage>
````

### OriginalFileName condition

Zweck: Definiert Bedingungen für den Originaldateinamen des Prozesses.
````xml

<OriginalFileName condition="is" name="...">
  <!-- Weitere Bedingungen für den Originaldateinamen -->
</OriginalFileName>
````

### CommandLine condition

Zweck: Legt Bedingungen für die Befehlszeile (Command Line) des Prozesses fest.
````xml
<CommandLine condition="contains" name="...">
  <!-- Weitere Bedingungen für die Befehlszeile -->
</CommandLine>
````

### Config Struktur

```xml
<Sysmon schemaversion="X.XX">
    <!-- Globale Einstellungen und Konfiguration -->
    <HashAlgorithms>*</HashAlgorithms>
    <CheckRevocation>False</CheckRevocation>
    <DnsLookup>False</DnsLookup>
    <!-- Weitere globale Einstellungen -->

    <!-- Eventfilterung: Hier werden Regeln für die Überwachung definiert -->
    <EventFiltering>
        <!-- Regelgruppen -->
        <RuleGroup groupRelation="or">
            <!-- Einzelne Regeln -->
            <Rule name="RuleName1">
                <!-- Bedingungen für diese Regel -->
            </Rule>
            <Rule name="RuleName2">
                <!-- Bedingungen für diese Regel -->
            </Rule>
            <!-- Weitere Regeln in dieser Gruppe -->
        </RuleGroup>

        <!-- Weitere Regelgruppen -->
    </EventFiltering>

    <!-- Optionale Einstellungen und Erweiterungen -->
    <!-- Weitere Konfigurationselemente -->

</Sysmon>
```

#### Quellen
[Microsoft Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)  
[Github Olaf Hartong Sysmon XML example](https://gist.github.com/olafhartong)