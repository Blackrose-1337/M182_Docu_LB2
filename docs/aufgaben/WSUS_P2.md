# LB2 - Installation, Konfiguration und Testing von WSUS (20%)#
## 1.1 Lernziele#

Jede/r Lernende:

    ...kann in eigenen Worten beschreiben was der WSUS-Service ist
    ...kann in einer bestehende Umgebung den WSUS-Service für Windows-Clients einrichten

## 1.2 Aufträge#

Installieren Sie nach bestehender Anleitung (oder Ihrem Wissen) den WSUS-Service in Ihrem Lab. Testen Sie die Update-Funktionlitäten auf Ihrer Windows 11-VM aus. Alle VMs sind in den Update-Prozess eingepflegt und funktionieren nachweislich.
## 1.3 Bewertungskriterien#

Recherche, allgemein - WSUS 0-4P:

- Wofür wird der WSUS-Dienst verwendet [WSUS-Dienst](tech_daten/wsusDienst_P2.md)
- Welche Alternativen gibt es für den WSUS-Dienst [Baramundi](tech_daten/baramundi_P2.md)

Recherche, Powershell Server - WSUS 0-4P:

- Welche Powershell-Cmdlets gibt es, um den WSUS-Dienst zu konfigurieren/steuern [Powershell Cmdlets](tech_daten/powershellCMD_P2.md)

**Installation Client - WSUS 0-4P: **

- Dokumenation der Konfiguration um den Update-Prozess mittels GPOs zu steuern [Dokumentation GPO Konfiguration](tech_daten/dokumentationGPO_P2.md)
- Welche Powershell-Cmdlets gibt es, um den WSUS-Client zu konfigurieren/steuern [Powershell Cmdlets](tech_daten/powershellCMD_P2.md)

**Testing und Reporting - WSUS 0-4P: **

- Testen Sie die Update-Funktionalität 
- Welche Reports können aus dem WSUS über die GUI erstellt werden

## 1.4 Tipps und Tricks#

Abhängig von den Produkten und Klassifi­zierungen, für die man Updates bezieht, kann sich WSUS zu einem Speicher­fresser aus­wachsen. Ein manuelles Auf­räumen ist dann zu auf­wändig, und der Assi­stent für die Server­bereinigung erweist sich als zahnlos. Ein Script von der TechNet-Gallery räumt gründlich auf.

[WSUS aufräumen](https://www.windowspro.de/wolfgang-sommergut/wsus-speicherplatz-zurueckgewinnen-alte-ersetzte-updates-loeschen?utm_source=feedburner&utm_medium=email&utm_campaign=Feed%3A+windowspro+%28WindowsPro%29)

Hier finden Sie Best Practices im Umgang mit WSUS:

[Best Practices bei WSUS](https://learn.microsoft.com/en-us/troubleshoot/mem/configmgr/update-management/windows-server-update-services-best-practices)

Hier finden Sie eine gute Anleitung für die Konfiguration von WSUS:

[Konfigurationsanleitung von MS zu WSUS](https://learn.microsoft.com/de-de/windows/deployment/update/waas-manage-updates-wsus)